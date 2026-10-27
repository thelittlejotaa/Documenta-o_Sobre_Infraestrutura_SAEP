# GUIA SAEP - INFRAESTRUTURA, ERROS E DIAGNOSTICO

PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express | manual de terminal
Nova edicao: dicionario comentado dos comandos, opcoes e saidas esperadas no final do guia.

Preparacao para a avaliacao pratica da Luthieria Jubal

Pontuacao da lista

35 pontos

Tecnologias deste guia

PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express

Metodo de diagnostico

descobrir a camada antes de alterar o codigo

BASE DOCUMENTAL: o caderno da avaliacao e a lista de verificacao fornecidos sao a fonte. O Anexo 1 completo nao veio nas paginas recebidas; por isso este guia nao inventa a versao exata do SGBD/linguagem nem o tempo exato de expiracao da sessao. Esses dois pontos devem ser copiados do Anexo 1/caderno completo.

REGRA DA PROVA: o caderno fornecido informa que nao e permitida internet durante a prova. Prepare bibliotecas, arquivos e conhecimentos antes; durante a prova use ambiente local, localhost, terminal e recursos ja disponiveis.


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

Sumario

Leitura correta da prova e da pontuacao

Infraestrutura inicial pelo terminal

PostgreSQL: operacao, criacao, verificacao e permissoes

O erro classico do Python na inicializacao do banco

Python/Flask: montar, iniciar e diagnosticar

Node/Express: montar, iniciar, permissoes e diagnosticar

Java/Spring Boot: compilar, iniciar e diagnosticar

JWT e sessoes: autenticacao, expiracao e permissao

Erros independentes da linguagem

SQL, PK, FK, UNIQUE, NOT NULL e JOIN

Implementacao alinhada aos criterios: Login e Principal

Implementacao alinhada aos criterios: Cliente

Instrumento e Agendamento

Teste completo pelo terminal com curl

Empacotamento e conferencia da entrega

Checklist final de 35/35

Folha de emergencia

Windows PowerShell: equivalentes essenciais

SQLSTATE PostgreSQL: codigos de erro para diagnostico

Referencias tecnicas oficiais consultadas

1. Leitura correta da prova e da pontuacao

A lista de verificacao fornecida possui sete atividades e 19 evidencias observaveis. Somando os pesos, o total e 35 pontos. O objetivo deste guia e transformar cada evidencia em um teste pratico.

Atividade

Foco

Peso

1. Documentacao

requisitos, infraestrutura e DER

4

2. Banco

FKs e 3+ registros por tabela

3

3. Login

autenticacao, expiracao e falha

6

4. Principal

nome, sair e navegacao

4

5. Cliente

listar, excluir, busca, inserir, proteger dados, editar

13

6. Instrumento

inserir associado a cliente

3

7. Agendamento

listar por data e alterar data

5

Prioridade de tempo: infraestrutura -> banco -> login -> Cliente -> instrumento/agendamento -> documentacao/empacotamento. Isso reduz o risco de descobrir no fim que a API sequer conecta ao PostgreSQL.

1.1 Matriz de gabarito

Item

Peso

Evidencia

Prova rapida

1.1

2

4+ requisitos funcionais claros e verificaveis

documento

1.2

1

SGBD + linguagem exatamente como no Anexo 1

documento

1.3

1

DER coerente com PK/FK/tabelas

DER

2.1

2

tabelas com chaves estrangeiras corretas

psql + SQL

2.2

1

3 registros em TODA tabela criada

COUNT(*)

3.1

1

login valida credencial do banco

login

3.2

3

sessao/token com expiracao do caderno

exp do token/sessao

3.3

2

falha de login tratada

401/feedback

4.1

2

nome do usuario autenticado no principal

sessao/token

4.2

1

sair do sistema

logout

4.3

1

acesso a Cliente, Instrumento, Tecnico, Agendamento

menu/rotas

5.1

2

listar + excluir

GET + DELETE

5.2

3

buscar por pelo menos um termo

query param

5.3

2

inserir com persistencia

POST + SELECT

5.4

3

senha e CPF protegidos

hash/cifra

5.5

3

editar cliente

PUT/PATCH + SELECT

6.1

3

instrumento ligado a cliente

FK

7.1

3

agendamentos ordenados por data e com relacionados

ORDER BY + JOIN

7.2

2

alterar data do agendamento

PATCH/PUT


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

2. Infraestrutura inicial pelo terminal

2.1 Descobrir o ambiente antes de modificar

Diagnostico basico Linux

pwd
whoami
cat /etc/os-release
uname -a
lscpu | sed -n '1,12p'
free -h
df -h

2.2 Python

python3 --version
python3 -m pip --version
command -v python3
python3 -m venv --help >/dev/null && echo "venv OK"

2.3 PostgreSQL

psql --version
pg_isready
sudo systemctl status postgresql --no-pager
sudo -u postgres psql -c "SELECT version();"

2.4 Java, JDK, Maven e Gradle

Primeiro descubra o que realmente existe no computador. Nao presuma que JDK, Maven/Gradle ou wrapper estejam instalados; em prova sem internet, um comando de instalacao pode falhar por falta de pacotes.

java -version
javac -version
echo "$JAVA_HOME"
command -v java
command -v javac
mvn -v
gradle -v
find . -maxdepth 3 -type f \( -name pom.xml -o -name mvnw -o -name build.gradle -o -name build.gradle.kts -o -name gradlew \) -print

Se houver mvnw/gradlew no projeto, prefira o wrapper desde que ele consiga executar com os recursos ja disponiveis. Se o wrapper tentar baixar uma distribuicao e a prova nao tiver internet, use apenas Maven/Gradle instalado e dependencias locais, quando isso for autorizado.

2.5 Node/NPM

node -v
npm -v
command -v node
command -v npm
npm config get cache
npm config get prefix

2.6 Processos e portas

ss -ltnp
sudo lsof -iTCP -sTCP:LISTEN -P -n
sudo lsof -i :5432
sudo lsof -i :5000
sudo lsof -i :3000
sudo lsof -i :8080

O objetivo desta etapa e responder cinco perguntas: qual sistema operacional estou usando? qual Python/Node tenho? PostgreSQL esta vivo? quais portas estao ocupadas? de quem sao os arquivos?


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

3. PostgreSQL: operacao, criacao, verificacao e permissoes

3.1 Servico

sudo systemctl status postgresql --no-pager
sudo systemctl start postgresql
sudo systemctl restart postgresql
pg_isready

Nao use restart por reflexo. Para um processo simplesmente parado, start e suficiente. Restart e mais invasivo e pode derrubar uma conexao que ja esteja funcionando.

3.2 Criar e localizar o banco

sudo -u postgres psql -c "SELECT datname FROM pg_database ORDER BY datname;"
sudo -u postgres psql -c "CREATE DATABASE jubal_db;"

O primeiro comando lista as bases existentes. O PostgreSQL nao aceita CREATE DATABASE IF NOT EXISTS na sintaxe comum: se jubal_db ja aparecer, nao repita a criacao nem apague a base sem confirmar que e apenas treino e que os dados podem ser perdidos. Se a base ja existe, siga para a inspecao/importacao conforme o estado atual.

3.3 Executar o SQL da prova

ON_ERROR_STOP=1 faz o psql parar com codigo de falha no primeiro erro SQL, em vez de continuar e deixar parte da base criada. Se o usuario postgres nao consegue ler um arquivo em uma pasta home privada, copie o .sql para um caminho legivel/autorizado ou execute com o usuario de banco correto.

cd /caminho/do/projeto
sudo -u postgres psql -v ON_ERROR_STOP=1 -d jubal_db -f jubal_db.sql

3.4 Inspecionar o banco correto

sudo -u postgres psql -d jubal_db

-- dentro do psql
\conninfo
SELECT current_database(), current_schema(), current_user;
\dn
\dt
\d cliente
\d instrumento
\d tecnico
\d agendamento
\d usuario
\q

3.5 Provar as 3 linhas por tabela

SELECT 'usuario' AS tabela, COUNT(*) AS total FROM usuario
UNION ALL
SELECT 'cliente', COUNT(*) FROM cliente
UNION ALL
SELECT 'tecnico', COUNT(*) FROM tecnico
UNION ALL
SELECT 'instrumento', COUNT(*) FROM instrumento
UNION ALL
SELECT 'agendamento', COUNT(*) FROM agendamento;

Se houver outras tabelas no seu DER, acrescente todas. O criterio da lista vale para cada tabela criada.

3.6 Erros de conexao e como separar as causas

Mensagem

O que realmente aconteceu

O que fazer

connection refused / could not connect

nao existe um servidor aceitando naquela porta

pg_isready, systemctl, porta

database does not exist

servidor existe, mas a base indicada nao

criar/usar a base correta

password authentication failed

usuario existe, senha falhou

corrigir credencial

Peer authentication failed

autenticacao local esta usando peer

usar forma de acesso adequada ou ajustar politica

no pg_hba.conf entry

PostgreSQL rejeitou pela regra de acesso

ver pg_hba.conf e recarregar

role does not exist

usuario PostgreSQL inexistente

\du; criar/usar role correta

permission denied for schema public

usuario nao pode acessar o schema

GRANT no schema

permission denied for sequence

INSERT tentou usar sequence sem privilegio

GRANT USAGE/SELECT nas sequences

3.7 Permissoes de um usuario da aplicacao

GRANT CONNECT ON DATABASE jubal_db TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;

Este bloco e para diagnosticar/perceber permissoes. Na prova, use o usuario indicado pelo ambiente. Em producao, evite usar postgres como usuario da aplicacao.

3.8 Se o script foi executado duas vezes

Caso tenha inserido IDs manualmente em coluna SERIAL/IDENTITY, a sequence pode ficar atrasada. O proximo INSERT pode gerar duplicate key mesmo que o ID nao tenha sido digitado no novo registro. Confirme a sequence associada antes de corrigi-la; nao altere IDs nem execute setval as cegas.

relation already exists -> tabela ja existe.

duplicate key -> INSERT repetido em valor UNIQUE/PK.

Se for treino, volte ao estado inicial com DROP DATABASE apenas quando nao houver trabalho que precise ser preservado.

Se o problema for so um registro duplicado, corrija o INSERT em vez de destruir a base inteira.

# Somente para reset do AMBIENTE DE TREINO
sudo -u postgres psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='jubal_db' AND pid <> pg_backend_pid();"
sudo -u postgres psql -c "DROP DATABASE IF EXISTS jubal_db;"
sudo -u postgres psql -c "CREATE DATABASE jubal_db;"
sudo -u postgres psql -v ON_ERROR_STOP=1 -d jubal_db -f jubal_db.sql


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

4. O erro classico do Python na inicializacao do banco

Muita gente perde tempo aqui porque o traceback aponta para app.py, init_db() ou psycopg2. O ponto exato do Python indica onde o erro foi acionado; a mensagem final pode ser totalmente do PostgreSQL.

4.1 Exemplo: banco inexistente

Traceback (most recent call last):
  File "app.py", line 80, in <module>
    init_db()
  File "app.py", line 20, in init_db
    conn = psycopg2.connect(...)
psycopg2.OperationalError: FATAL: database "jubal_db" does not exist

A leitura correta e: o Python chamou init_db(); init_db tentou psycopg2; PostgreSQL respondeu "database does not exist". A correcao e de infraestrutura/banco, nao de sintaxe Flask.

4.2 Classificacao rapida

Classe do erro Python

Pista

Camada

psycopg2.OperationalError

connect, authentication, refused

PostgreSQL/credencial/porta

psycopg2.errors.UndefinedTable

relation X does not exist

tabela/schema/banco

psycopg2.errors.UndefinedColumn

column X does not exist

modelo/SQL

psycopg2.errors.ForeignKeyViolation

violates foreign key constraint

relacionamento

psycopg2.errors.UniqueViolation

duplicate key

UNIQUE/PK

psycopg2.errors.NotNullViolation

null value ...

campo obrigatorio

psycopg2.errors.SyntaxError

syntax error at or near

SQL

4.3 Isolar a conexao sem Flask

python3 - <<'__PY__'
import psycopg2
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    dbname="jubal_db",
    user="postgres",
    password="SENHA"
)
print(conn.get_dsn_parameters())
cur = conn.cursor()
cur.execute("SELECT current_database(), current_schema(), current_user")
print(cur.fetchone())
cur.close()
conn.close()
__PY__

Se a aplicacao falhar em init_db(), adicione temporariamente prints antes/depois de connect, execute e commit; registe type(e).__name__ e a mensagem completa no terminal e use raise para preservar o traceback. Nao conclua que e erro de Flask so porque o traceback aponta para app.py: a excecao final pode ser do psycopg2/PostgreSQL.

5. Python/Flask: montar, iniciar e diagnosticar

5.1 Criar ambiente virtual

mkdir -p ~/saep_jubal/treino
cd ~/saep_jubal
python3 -m venv .venv
source .venv/bin/activate
python -m pip --version
python -m pip list

5.2 Conferir bibliotecas

python -c "import flask; print('Flask OK')"
python -c "import psycopg2; print('psycopg2 OK')"
python -c "import jwt; print('PyJWT OK')" 2>/dev/null || true
python -c "import cryptography; print('cryptography OK')" 2>/dev/null || true

5.3 Instalacao offline, se houver pacotes locais autorizados

python -m pip install --no-index --find-links ./packages Flask psycopg2-binary PyJWT cryptography

5.4 Validar sintaxe sem iniciar o servidor

python -m py_compile treino/app.py
python -m compileall treino

5.5 Iniciar

cd ~/saep_jubal
source .venv/bin/activate
python treino/app.py

5.6 Estrutura enxuta

saep_jubal/
  .venv/
  jubal_db.sql
  documentacao.pdf
  DER.pdf
  treino/
    app.py
    templates/
    static/

5.7 Erros Python/Flask

Diagnosticos adicionais: confira sys.executable e pip dentro do venv (python -c "import sys; print(sys.executable)"); use python -m pip show Flask psycopg2-binary PyJWT para confirmar o ambiente ativo. Request.get_json pode falhar se o Content-Type/body estiver incorreto; retorno None costuma indicar body ausente ou parsing nao executado. Se aparecer 500, veja o traceback e a ultima excecao, nao apenas a linha app.py que chamou init_db(). Para erros PostgreSQL, o SQLSTATE e a mensagem do psycopg2 identificam a camada real [R11, R12].

Outros erros possiveis no SAEP: ImportError (import circular ou nome errado); AttributeError (objeto None/retorno errado); TypeError: not all arguments converted ou format mismatch (parametros SQL no formato incorreto); InterfaceError: connection already closed (conn fechada antes do cursor); InFailedSqlTransaction/current transaction is aborted (faltou rollback apos a query anterior falhar); UndefinedFunction (funcao SQL/ILIKE/dialect incorreto); InsufficientPrivilege (GRANT/owner). Em desenvolvimento, registre a excecao completa no terminal, mas nao devolva senha, segredo, CPF ou traceback ao cliente.

Mensagem

Camada

Causa

Acao

ModuleNotFoundError: flask

dependencia

venv errada ou Flask ausente

ativar .venv; pip list; usar pacote local

ModuleNotFoundError: psycopg2

driver DB

driver PostgreSQL ausente

instalar psycopg2-binary de fonte local

ModuleNotFoundError: jwt

JWT

PyJWT ausente

disponibilizar PyJWT localmente

SyntaxError

codigo

sintaxe Python invalida

py_compile

IndentationError

codigo

indentacao invalida

padronizar 4 espacos

NameError

codigo

nome nao definido

conferir import/nome

KeyError

entrada

chave nao veio no JSON/sessao

validar request.json/dados

TypeError

tipo

operacao com tipo errado

conferir string/int/dict/None

AttributeError em NoneType

dados

resultado esperado veio None

tratar busca por ID inexistente

ValueError

conversao

valor nao converte

validar antes de int/data

Address already in use

porta

5000 ocupada

lsof -i :5000

500 Internal Server Error

multicamadas

excecao nao tratada

ler traceback completo

BuildError

Flask

url_for aponta para endpoint inexistente

conferir nome da rota

CORS policy

browser

origem bloqueada

configurar somente se houver frontend separado


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

6. Node/Express: montar, iniciar, permissoes e diagnosticar

6.1 Criar projeto

mkdir -p ~/saep_jubal_node/treino
cd ~/saep_jubal_node
npm init -y
node -v
npm -v

6.2 Conferir dependencias sem internet

npm ls express pg jsonwebtoken --depth=0
npm cache verify

6.3 Pacotes locais/cache

# Se a escola/preparacao fornecer arquivos .tgz:
npm install ./packages/express-*.tgz
npm install ./packages/pg-*.tgz
npm install ./packages/jsonwebtoken-*.tgz

# Se o cache local ja tiver tudo e for permitido:
npm install --offline

6.4 EACCES - permissao no Node

ls -la
find . -maxdepth 2 -not -user "$USER" -ls
ls -ld ~/.npm
sudo chown -R "$USER":"$USER" ~/saep_jubal_node
sudo chown -R "$USER":"$USER" ~/.npm
npm install

EACCES geralmente significa que algum arquivo/pasta foi criado por outro usuario, frequentemente root. Usar sudo npm install como primeira resposta pode piorar o problema porque cria mais arquivos com dono root.

6.5 Validar JavaScript sem iniciar o servidor

node --check treino/app.js

6.6 Iniciar

node treino/app.js
# ou
npm start

6.7 Erros Node/Express

Outros erros possiveis: ERR_MODULE_NOT_FOUND (import/ESM ou pacote ausente); ERR_REQUIRE_ESM (require() em pacote ESM); Cannot use import statement outside a module (type/module ou extensao incoerente); SyntaxError: Unexpected token (JS/JSON invalido); TypeError: Cannot read properties of undefined (req.body/req.params/campo ausente); listen EACCES (porta privilegiada ou permissao); ETIMEDOUT (servico demora); ECONNRESET (conexao interrompida); 28P01/42P01 recebidos do pg (senha/tabela PostgreSQL); query retorna Promise e a rota nao usa await/catch (erro assinc. nao tratado em versoes/configuracoes onde nao e propagado automaticamente). Express requer tratamento adequado de rejeicoes async; ver [R14].

Mensagem

Camada

Causa

Acao

EACCES

Linux/permissao

arquivo/pasta sem acesso

chown/permissions; evitar sudo npm install

EADDRINUSE

porta

3000 ocupada

lsof -i :3000

ECONNREFUSED

servico/DB

nada aceitando na porta

PostgreSQL/API/porta

MODULE_NOT_FOUND

dependencia/caminho

pacote ou arquivo ausente

npm ls; conferir caminho; pacote local

ENOENT package.json

projeto

npm fora da pasta

cd correto

req.body vazio

Express

parser JSON ausente

app.use(express.json()) antes das rotas

Cannot GET /clientes

rota

metodo/caminho incorretos

conferir get/router/prefixo

next is not a function

middleware

assinatura errada

(req,res,next) + next()

JsonWebTokenError

JWT

token invalido

Bearer/secret/formato

TokenExpiredError

JWT

token expirado

login novamente; conferir exp

secretOrPrivateKey must have a value

JWT

secret vazia

configurar secret

password authentication failed

PostgreSQL

credencial rejeitada

usuario/senha/role

relation does not exist

PostgreSQL

tabela ausente

rodar SQL no banco correto

duplicate key

PostgreSQL

UNIQUE/PK duplicada

verificar registro

foreign key violation

PostgreSQL

ID pai inexistente

criar pai primeiro

CORS policy

browser

origem bloqueada

configurar CORS somente quando necessario


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

7. Java/Spring Boot: compilar, iniciar e diagnosticar

Esta secao cobre Java como trilha alternativa de implementacao para a mesma prova da Luthieria Jubal. Os comandos de exemplo assumem um terminal Linux. Use a versao de JDK, o build tool, os nomes dos pacotes e as rotas do projeto fornecido no posto de trabalho; nao altere a stack exigida no Anexo 1.

IMPORTANTE SOBRE A PROVA: a lista exige sessao com expiracao, mas as paginas recebidas nao revelam o tempo exato nem exigem explicitamente JWT. Use sessao HTTP ou JWT conforme o caderno/Anexo 1 completo. JWT e um mecanismo possivel, nao uma exigencia presumida.

7.1 Diagnostico inicial do ambiente Java

Antes de editar o codigo, confirme Java/JDK e a ferramenta de build. java executa; javac compila. Se javac nao existir, pode haver somente JRE ou o JDK pode nao estar no PATH.

java -version
javac -version
echo "$JAVA_HOME"
command -v java
command -v javac
mvn -v
gradle -v
find . -maxdepth 4 -type f \( -name pom.xml -o -name mvnw -o -name build.gradle -o -name build.gradle.kts -o -name gradlew \) -print

Se houver Maven Wrapper, teste ./mvnw -v; se houver Gradle Wrapper, teste ./gradlew --version. Erro de permissao no wrapper: ls -l mvnw gradlew e chmod +x mvnw (ou gradlew) apenas se o arquivo pertencer ao projeto e for esperado que seja executavel.

Equivalentes rapidos no PowerShell: Get-Command java,javac,mvn,gradle -ErrorAction SilentlyContinue; Get-ChildItem -Recurse -Depth 3 -Include pom.xml,mvnw,build.gradle,build.gradle.kts,gradlew. Nao suponha nomes de servico ou caminhos iguais em todos os computadores.

7.2 Encontrar o projeto e escolher o comando certo

Maven: confirme pom.xml. Gradle: confirme build.gradle/build.gradle.kts. Nao crie outro projeto nem substitua arquivos de configuracao antes de localizar a estrutura recebida.

# Maven Wrapper, se ja instalado e com dependencias disponiveis
./mvnw -o -DskipTests package

# Maven instalado localmente
mvn -o -DskipTests package

# Gradle Wrapper
./gradlew --offline build

# Localizar o JAR depois do build Maven
ls -1 target/*.jar 2>/dev/null

# Executar JAR compilado (substitua o nome real)
java -jar target/NOME-REAL.jar

A opcao offline evita que Maven tente buscar dependencias no repositorio remoto; ela nao faz dependencias ausentes aparecerem. Se faltar um plugin, artefato ou a propria distribuicao do wrapper, use somente recursos previamente instalados/disponiveis ou o procedimento autorizado pelo avaliador. Nao dependa de internet: o caderno fornecido proibe seu uso durante a avaliacao.

7.3 Erros de JDK, compilacao e Maven/Gradle

Mensagem/sintoma

Causa provavel

Diagnostico e correcao pelo terminal

java: command not found / java nao e reconhecido

JDK/JRE ausente ou PATH incorreto

command -v java; java -version; conferir PATH e ambiente instalado.

javac: command not found

JDK ausente ou PATH aponta para runtime sem compilador

javac -version; localizar JDK preinstalado; nao tentar baixar sem internet.

JAVA_HOME is not defined correctly

JAVA_HOME aponta para pasta errada ou inclui /bin indevidamente

echo "$JAVA_HOME"; readlink -f "$(command -v java)"; ajustar para raiz real do JDK.

mvn: command not found / gradle: command not found

Ferramenta nao instalada no sistema

verificar mvnw/gradlew no projeto; tentar wrapper se cache/distribuicao local existir.

invalid source release / invalid target release

O projeto pede bytecode mais novo que o JDK disponivel

mvn -v ou ./gradlew --version; comparar JDK e maven.compiler.release/sourceCompatibility com o projeto. Nao mudar a versao exigida sem motivo.

UnsupportedClassVersionError

Classe compilada com JDK mais novo que o runtime atual

comparar java -version com a versao usada para compilar; executar com runtime compativel ou recompilar usando o JDK do ambiente.

package ... does not exist / cannot find symbol

Dependencia ausente, import errado, nome de classe/metodo errado ou compilacao incompleta

mvn -o -DskipTests compile; ./gradlew --offline compileJava; ler a primeira falha do compilador.

Non-resolvable parent POM / Could not resolve dependencies

Maven precisa de dependencia/plugin que nao esta em cache, ou coordenada/redepositorio incorretos

ler groupId/artifactId/version; verificar pom.xml, ~/.m2/repository e se o wrapper/build offline e viavel.

Could not find or load main class

Classe principal ou classpath/caminho errado

usar comando de execucao do projeto; confirmar package, classe @SpringBootApplication e JAR correto.

NoClassDefFoundError / ClassNotFoundException

Classe/dependencia indisponivel em runtime ou classpath incompleto

mvn -o package; conferir dependencias runtime e executar o artefato gerado, nao uma classe solta.

NoSuchMethodError / NoSuchFieldError

Versoes incompatíveis de bibliotecas no classpath

examinar dependencia conflitante; Maven: ./mvnw -o dependency:tree se plugin disponivel; nao misturar JARs de versoes diferentes.

7.4 Validacao de sintaxe, build e logs

Use a ordem mais curta que confirme a camada que falha. Nao recompile repetidamente sem ler a primeira mensagem de erro; mensagens posteriores podem ser apenas cascata da primeira.

# Maven
./mvnw -o -DskipTests compile
./mvnw -o -DskipTests package

# Gradle
./gradlew --offline compileJava
./gradlew --offline build

# Executar com diagnostico Spring Boot (JAR compilado)
java -jar target/NOME-REAL.jar --debug

Em Spring Boot, APPLICATION FAILED TO START e um resumo, nao necessariamente a causa raiz. Leia Description/Action, o primeiro bloco Caused by relevante e o fim do log. --debug aumenta informacao da avaliacao de auto-configuracao; nao significa que toda classe passe a registrar DEBUG.

# Buscar linhas importantes em um log salvo (se a aplicacao redireciona para arquivo)
grep -nE 'ERROR|Caused by:|APPLICATION FAILED TO START|Description:|Action:' app.log | tail -n 100

# Capturar log do processo em terminal para consulta
./mvnw -o spring-boot:run 2>&1 | tee saep-java.log

7.5 Configuracao JDBC e PostgreSQL no Spring Boot

O Spring Boot configura DataSource por propriedades spring.datasource.* e identifica o driver normalmente pela URL, desde que o driver esteja no classpath. O projeto precisa conter a dependencia PostgreSQL compativel no pom.xml/build.gradle. Veja a documentacao oficial de SQL do Spring Boot [R1].

# Exemplo ilustrativo de application.properties; adapte credenciais e nome ao ambiente
spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/jubal_db
spring.datasource.username=postgres
spring.datasource.password=COLOQUE_A_SENHA_LOCAL
# driver-class-name pode ser omitido quando a URL permite a deteccao
# Use a estrategia de schema indicada pelo projeto; nao use create-drop se precisa preservar os dados.

Teste a conexao com o MESMO host, porta, database e usuario utilizados pela aplicacao. Uma conexao local por socket/peer como usuario do sistema postgres nao prova que a conexao TCP com usuario/senha usada pelo Java funcionara.

psql -h 127.0.0.1 -p 5432 -U postgres -d jubal_db -W -c "SELECT current_database(), current_user, current_schema();"
pg_isready -h 127.0.0.1 -p 5432
ss -ltnp | grep ':5432'
sudo journalctl -u postgresql --since '-10 min' --no-pager

7.6 Erros de inicializacao Spring Boot e beans

Mensagem/sintoma

Causa provavel

Como diagnosticar/corrigir

APPLICATION FAILED TO START

Resumo do Spring Boot; ainda falta descobrir a excecao raiz

ler Description/Action e Caused by; se necessario executar JAR com --debug [R2, R3].

Port 8080 was already in use / Web server failed to start

Outro processo ja usa a porta

ss -ltnp | grep :8080; sudo lsof -i :8080; identificar o processo antes de encerrar, ou usar porta alternativa se permitido.

Failed to configure a DataSource: url attribute is not specified

URL JDBC ausente, perfil errado ou configuracao nao carregada

conferir src/main/resources/application.properties, perfil SPRING_PROFILES_ACTIVE e variaveis de ambiente.

Failed to determine a suitable driver class

URL/driver/dependencia PostgreSQL ausente ou configuracao incompleta

conferir JDBC URL jdbc:postgresql://host:5432/banco e dependencia org.postgresql:postgresql.

Cannot load driver class org.postgresql.Driver / No suitable driver

Driver PostgreSQL nao esta no runtime classpath ou versao/build nao inclui dependencia

conferir pom.xml/build.gradle e reconstruir offline apenas se a dependencia estiver local.

PSQLException: Connection refused

PostgreSQL parado, host/porta errados ou escuta inacessivel

pg_isready, ss/lsof, testar psql via TCP com as mesmas credenciais.

FATAL: database does not exist / password authentication failed

Nome de base ou credenciais incorretos

psql -h 127.0.0.1 -U USUARIO -d jubal_db -W; comparar exatamente com DataSource.

BeanCreationException

Um bean falhou ao ser criado; a mensagem externa pode ser genérica

procurar Caused by mais interno; identificar configuracao, EntityManagerFactory, SecurityFilterChain ou repository citado.

UnsatisfiedDependencyException

Controller/service/repository depende de bean que nao iniciou

seguir a cadeia do stack trace ate a primeira causa concreta.

NoSuchBeanDefinitionException / No qualifying bean

Bean ausente, pacote fora do component scan ou configuracao nao registrada

checar @Component/@Service/@Repository/@Bean, pacote da classe principal e dependencias.

Not a managed type: ...

Classe de entidade nao reconhecida pelo JPA

conferir @Entity, imports jakarta.persistence no Spring Boot 3+ e pacote escaneado.

Not a valid repository method / QueryCreationException

Nome de metodo derivado nao corresponde a propriedade ou query invalida

comparar findBy... com nomes Java reais da entidade; conferir @Query e parametros.

Schema-validation: missing table/column / relation does not exist

SQL, DER e mapeamentos JPA nao coincidem ou app conecta em outra base

psql: SELECT current_database(); \dt; \d nome_tabela; comparar @Table/@Column e SQL.

Permission denied for table/sequence/schema

O usuario da aplicacao nao possui GRANT adequado

conferir current_user, owner, schema e permissao de sequence; conceder somente privilegios necessarios.

Too many connections / HikariPool connection timeout

PostgreSQL no limite, pool esgotado ou conexoes sem liberar

conferir logs, pg_stat_activity e fechamento de recursos; nao aumentar limites sem entender a causa.

7.7 Erros JPA/Hibernate e de persistencia

Mensagem/sintoma

O que indica

Acao pratica

DataIntegrityViolationException

Spring traduziu uma violacao de integridade; a causa interna geralmente tem SQLSTATE PostgreSQL [R5].

procurar a causa interna/PSQLException: NOT NULL, UNIQUE, FK, CHECK ou tipo.

ConstraintViolationException

Pode ser constraint do banco ou validacao Bean Validation; olhar package/classe e mensagem raiz.

se org.hibernate.exception, conferir restricao SQL; se jakarta.validation, conferir @NotBlank/@NotNull/@Size e dados.

could not execute statement

Hibernate falhou ao enviar SQL; mensagem externa e insuficiente.

ler PSQLException/SQLSTATE; testar query no psql.

TransactionRequiredException / Executing an update/delete query

Operacao que altera dados foi chamada sem transacao adequada ou por query configurada incorretamente.

verificar @Transactional na camada de servico e @Modifying quando for Spring Data JPA.

LazyInitializationException

Relacionamento lazy foi acessado fora do contexto/sessao Hibernate.

carregar dados dentro de transacao, usar consulta/fetch apropriado ou DTO; nao trocar tudo para EAGER sem pensar.

Not an entity / Not a managed type

Classe nao mapeada pelo JPA.

@Entity, @Id, pacote/imports corretos e compilacao limpa.

StackOverflowError ao serializar JSON

Relacao bidirecional Cliente <-> Instrumento/Agendamento recorre indefinidamente.

retornar DTO/view model ou configurar serializacao para evitar ciclo; nao expor grafo JPA inteiro automaticamente.

EmptyResultDataAccessException / No result found

O codigo exige registro unico, mas a busca nao encontrou.

validar ID e responder 404/erro tratado; usar Optional quando apropriado.

IncorrectResultSizeDataAccessException / NonUniqueResultException

Consulta esperava 1 linha, mas retornou 0 ou varias.

conferir critério UNIQUE, filtro e tipo de retorno.

OptimisticLockingFailureException

Registro foi alterado concorrentemente ou versao esperada nao bate.

reler registro e repetir/recusar atualizacao conscientemente; menos comum no SAEP, mas possivel.

7.8 Erros HTTP/JSON em Controller Spring MVC

Mensagem/status

Causa frequente

Correcao/teste

HttpMessageNotReadableException / 400

JSON malformado, tipo incompatível, corpo vazio ou data fora do formato.

conferir body JSON e Content-Type; enviar data ISO como 2026-10-10 ou data-hora 2026-10-10T14:30:00 conforme o tipo Java.

HttpMediaTypeNotSupportedException / 415

Content-Type nao aceito pelo endpoint.

curl/Thunder Client: -H "Content-Type: application/json".

HttpRequestMethodNotSupportedException / 405

Rota existe, mas nao aceita GET/POST/PUT/PATCH/DELETE usado.

conferir anotacao @GetMapping/@PostMapping/@PutMapping/@PatchMapping/@DeleteMapping.

NoResourceFoundException / 404

Caminho nao mapeado/recurso inexistente; o nome depende da versao Spring.

conferir path, prefixo @RequestMapping, metodo e ID; testar curl -i.

Failed to convert String to Long/LocalDate

Parametro de rota/query nao corresponde ao tipo esperado.

usar ID numerico e formato de data esperado; validar entrada antes de chamar service.

MethodArgumentNotValidException

DTO falhou em @Valid e constraints.

verificar campo, @NotBlank/@NotNull/@Size e devolver mensagem de validacao.

HttpMessageConversionException / loop no JSON

Objeto nao serializavel ou relacionamento circular.

retornar DTO; conferir getters, tipo e relacoes bidirecionais.

500 sem detalhe no cliente

Excecao nao tratada; o cliente pode receber resposta genérica.

ler o terminal/log do Spring; localizar stack trace e excecao raiz. Nao devolver stack trace a utilizador final.

7.9 Spring Security, sessao, logout e JWT

O Spring Security usa filtros antes do Controller. Uma rota pode retornar 401/403 sem que o Controller execute; isso nao prova erro no banco. A documentacao oficial descreve essa cadeia de filtros e o processamento de Bearer tokens [R4, R5].

Sintoma

Causa a testar primeiro

Solucao/confirmacao

401 antes de entrar no Controller

Usuario/sessao ausente, cookie JSESSIONID nao enviado, Bearer ausente/invalido ou autenticacao falhou.

testar primeiro login; ver header/cookie; observar log Security. Se sessao, manter cookies entre requisicoes.

403 em POST/PUT/PATCH/DELETE, mas GET funciona

Regra de autorizacao/role, CSRF ou CORS quando chamado no navegador.

ver log Spring Security; em app web com cookie/sessao nao desligar CSRF indiscriminadamente; para API stateless, configurar conforme o modelo real.

AccessDeniedException

Autenticado sem autoridade exigida.

conferir autoridade real e regra hasRole/hasAuthority. ROLE_ADMIN e ADMIN nao sao nomes equivalentes em todas as APIs.

BadCredentialsException

Login ou senha nao confere, usuario pode nao existir ou comparacao de senha esta errada.

consultar usuario e validar hash; a prova exige falha tratada, nao erro 500.

Encoded password does not look like BCrypt

Senha em texto puro/algoritmo diferente armazenado no banco, mas verificador espera BCrypt.

nao comparar hash por igualdade; usar PasswordEncoder correspondente e armazenar senhas com hash seguro.

No qualifying bean of type PasswordEncoder

Configuracao do encoder/bean ausente.

registrar PasswordEncoder compatível com dependencias e o metodo de autenticação.

JwtDecoder not found / No qualifying bean JwtDecoder

Aplicacao espera JWT mas a configuracao/dependencias/decoder nao foi providenciada.

confirmar se JWT e realmente a estrategia exigida; se for, alinhar dependencias e configuracao de decoder/secret/chaves.

invalid_token, invalid signature, expired token

Token malformado, assinatura/chave incorreta, validade expirada ou claims recusadas.

emitir token novo; comparar algoritmo/chave/issuer/audience e relogio; nao aceitar token apenas decodificado sem validar assinatura.

Logout parece funcionar, mas rota protegida continua aberta

Token JWT continua armazenado no cliente ou sessao nao foi invalidada.

sessao: invalidar sessao e limpar cookie; JWT: remover token do cliente e reconhecer que isso nao revoga automaticamente uma copia ja emitida sem mecanismo server-side.

Para uma sessao HTTP, cookie e timeout precisam coincidir com a regra fornecida no caderno. Para JWT, o claim exp precisa usar o valor exigido. As paginas recebidas nao mostram o valor exato do tempo; nao memorize um numero hipotetico.

7.10 Testar a aplicacao Java apenas pelo terminal

Ajuste a porta e as rotas aos mapeamentos reais. O curl nao impõe CORS: se curl funciona e o navegador falha, investigue CORS/origem/cookies antes de culpar o PostgreSQL.

# Confirmar que existe processo na porta padrao Spring Boot
ss -ltnp | grep ':8080'
sudo lsof -i :8080

# Ver se o servidor responde (404 tambem confirma que houve resposta HTTP)
curl -i -v http://127.0.0.1:8080/

# Exemplo de login JSON; adapte caminho e campos ao projeto
curl -i -X POST http://127.0.0.1:8080/login -H 'Content-Type: application/json' -d '{"email":"usuario@teste.local","senha":"123456"}'

# Se o projeto usa JWT e emite bearer token
curl -i http://127.0.0.1:8080/clientes -H 'Authorization: Bearer COLE_TOKEN_AQUI'

# Se o projeto usa sessao por cookie, guardar e enviar cookies no fluxo real
curl -i -c cookies.txt -X POST http://127.0.0.1:8080/login -H 'Content-Type: application/x-www-form-urlencoded' --data 'username=USUARIO&password=SENHA'
curl -i -b cookies.txt http://127.0.0.1:8080/clientes

Os dois exemplos de login (JWT/JSON e sessao/form) sao alternativas. Nao os copie juntos sem conferir o tipo de aplicacao fornecida.

7.11 Matriz de emergencia Java/Spring

Erro visto

Primeira verificacao

Camada provavel

APPLICATION FAILED TO START

ler Description/Action/Caused by

inicio/configuracao

invalid source/target release

java -version, javac -version, mvn -v

JDK/build

Port 8080 already in use

ss -ltnp / lsof -i :8080

porta/processo

Failed to determine suitable driver

JDBC URL e dependencia PostgreSQL

configuracao/classpath

PSQLException / SQLSTATE

ler detalhe do PostgreSQL

banco/SQL/credencial

BeanCreationException

seguir primeira causa interna

configuracao/bean

UnsatisfiedDependencyException

qual dependencia falhou?

injecao de dependencia

Not a managed type

@Entity/@Id/pacote escaneado

JPA/mapeamento

DataIntegrityViolationException

causa interna + SQLSTATE

PK/FK/UNIQUE/NOT NULL/CHECK

LazyInitializationException

acesso lazy fora da transacao

JPA/ciclo de vida

401

sessao/token/filtro Security

autenticacao

403

role/CSRF/permissao

autorizacao/seguranca

415 ou HttpMessageNotReadableException

Content-Type, JSON, datas e tipos

HTTP/body

404 ou 405

caminho, prefixo, metodo e ID

Controller/rota

8. JWT e sessoes: autenticacao, expiracao e permissao

Pense sempre em tres perguntas: (1) existe token? (2) o token e valido? (3) o usuario tem permissao?

Sintoma

Diagnostico

Correcao

401 - Missing Authorization

header nao enviado

Authorization: Bearer TOKEN

401 - jwt malformed

token cortado/formato invalido

login novamente e copiar token inteiro

401 - invalid signature

secret de assinatura diferente da verificacao

mesma secret/algoritmo

401 - jwt expired

expiracao vencida

login novamente; revisar exp e relogio

401 - invalid algorithm

algoritmo inesperado

alinhar algoritmo

403 - token valido

autenticou mas nao pode acessar

corrigir role/regra da rota

login 200, rota 401

middleware nao esta lendo o token

verificar Authorization/middleware

uma rota funciona e outra 403

politicas diferentes

comparar middleware/autorizacao

secretOrPrivateKey...

secret vazia

definir secret

token expira imediatamente

clock/exp mal calculado

conferir date e calculo da expiracao

8.1 JWT: header correto (somente se esta for a estrategia do projeto)

Authorization: Bearer eyJhbGciOi...

8.2 Teste local

curl -i -X POST http://127.0.0.1:5000/login   -H 'Content-Type: application/json'   -d '{"email":"usuario@teste.local","senha":"123456"}'

curl -i http://127.0.0.1:5000/clientes   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'

8.3 Expiracao da sessao/token

A lista de verificacao exige uma sessao com tempo de expiracao conforme o caderno. Isso nao prova, sozinho, que JWT seja obrigatorio: sessao HTTP no servidor e JWT sao alternativas. O valor numerico do timeout nao aparece nas paginas recebidas; copie-o do caderno/Anexo 1 completo. Se usar JWT, valide assinatura e exp; se usar sessao, teste cookie, persistencia e invalidacao no logout.


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

9. Erros independentes da linguagem

Atencao: a rubrica da avaliacao fala em sessao com expiracao. Isso pode ser uma sessao tradicional (cookie no navegador) ou um token conforme a stack escolhida. Nao diagnostique toda falha de sessao como JWT; em uma aplicacao baseada em cookie, verifique cookie, dominio/host, expirar e invalidacao no logout.

Erro

Significado

Primeiro passo

400 Bad Request

body/campo/formato invalido

revisar JSON, parametros e validacao

401 Unauthorized

falha de autenticacao

login, token, Bearer, exp, secret

403 Forbidden

sem autorizacao

role/permissao

404 Not Found

rota/ID inexistente

URL, metodo e ID

409 Conflict

conflito

UNIQUE ou regra de negocio

415 Unsupported Media Type

Content-Type inadequado

application/json

422 Unprocessable Entity

dados semanticamente invalidos

validacao

500 Internal Server Error

excecao interna

ler terminal/log

503 Service Unavailable

servico indisponivel

API/PostgreSQL parado

connection refused

porta sem servidor

ss/lsof + iniciar processo

timeout

conexao nao terminou

host/porta/processo

permission denied

permissao OS/DB

ls -l, chown, GRANT, pg_hba

wrong database

app conectado em outra base

SELECT current_database()

9.1 Mapa mental para nao se perder

401 -> AUTENTICACAO / JWT
403 -> AUTORIZACAO / ROLE
404 -> URL / ROTA / ID
415 -> CONTENT-TYPE / JSON
500 -> TERMINAL + ULTIMA EXCECAO
EACCES -> PERMISSAO
EADDRINUSE -> PORTA OCUPADA
ECONNREFUSED -> SERVICO OU BANCO NAO ACEITA CONEXAO
relation does not exist -> BANCO/SCHEMA/TABELA
foreign key violation -> ID PAI NAO EXISTE
duplicate key -> UNIQUE/PK REPETIDA
not-null violation -> CAMPO OBRIGATORIO VAZIO


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

10. SQL, PK, FK, UNIQUE, NOT NULL e JOIN

10.1 Ordem de criacao

usuario
cliente
tecnico
instrumento -> cliente
agendamento -> cliente + instrumento + tecnico

10.2 Exemplo de modelo minimo para estudo

CREATE TABLE usuario (
    id_usuario SERIAL PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    senha_hash TEXT NOT NULL
);

CREATE TABLE cliente (
    id_cliente SERIAL PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    cpf_criptografado TEXT NOT NULL,
    senha_hash TEXT,
    telefone VARCHAR(30)
);

CREATE TABLE tecnico (
    id_tecnico SERIAL PRIMARY KEY,
    nome VARCHAR(120) NOT NULL
);

CREATE TABLE instrumento (
    id_instrumento SERIAL PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    tipo VARCHAR(80),
    cliente_id INTEGER NOT NULL REFERENCES cliente(id_cliente)
);

CREATE TABLE agendamento (
    id_agendamento SERIAL PRIMARY KEY,
    data_agendamento TIMESTAMP NOT NULL,
    cliente_id INTEGER NOT NULL REFERENCES cliente(id_cliente),
    instrumento_id INTEGER NOT NULL REFERENCES instrumento(id_instrumento),
    tecnico_id INTEGER NOT NULL REFERENCES tecnico(id_tecnico)
);

Este bloco e um MODELO DE ESTUDO, nao uma afirmacao de nomes exatos do Anexo 1. Na prova, adapte para o DER exigido. Tambem evite criar tabelas extras sem necessidade: cada tabela criada aumenta o trabalho de populacao e verificacao.

10.3 JOIN da listagem de agendamentos

SELECT
    a.id_agendamento,
    a.data_agendamento,
    c.nome AS cliente,
    i.nome AS instrumento,
    t.nome AS tecnico
FROM agendamento a
INNER JOIN cliente c ON c.id_cliente = a.cliente_id
INNER JOIN instrumento i ON i.id_instrumento = a.instrumento_id
INNER JOIN tecnico t ON t.id_tecnico = a.tecnico_id
ORDER BY a.data_agendamento ASC;

10.4 Busca por termo

SELECT *
FROM cliente
WHERE nome ILIKE '%ana%'
ORDER BY nome;

Na aplicacao, use parametro da biblioteca de banco em vez de concatenar texto diretamente. A query acima e didatica para entender a ideia.

10.5 Diagnostico de integridade

Erro PostgreSQL

Significado

Pergunta certa

UndefinedTable

tabela nao existe

estou no banco/schema certo?

UndefinedColumn

coluna nao existe

codigo e DER usam mesmo nome?

ForeignKeyViolation

pai inexistente

cliente/tecnico/instrumento existe?

UniqueViolation

valor unico repetido

email/CPF/PK ja existe?

NotNullViolation

obrigatorio vazio

JSON/INSERT trouxe valor?

invalid input syntax

tipo invalido

string/int/data estao corretos?

relation already exists

script repetido

tabela ja foi criada?


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

11. Implementacao alinhada aos criterios: Login e Principal

11.1 Fluxo esperado

LOGIN
  -> recebe login + senha
  -> consulta banco
  -> compara senha com hash
  -> cria sessao/token com expiracao
  -> acesso ao Principal
  -> Principal mostra nome autenticado
  -> Principal oferece Cliente, Instrumento, Tecnico, Agendamento
  -> Sair limpa/invalida sessao/token

11.2 Testes que precisam passar

Teste

Resultado esperado

Falha perigosa

login correto

entrada no principal

500

senha errada

falha tratada / 401

500

sem token em rota protegida

401

dados liberados

token expirado

401

rota liberada

logout

acesso protegido deixa de funcionar

token continua valido

principal

nome + 4 acessos

menu incompleto

11.3 Senha: hash, nao texto puro

# Python / Werkzeug
from werkzeug.security import generate_password_hash, check_password_hash
h = generate_password_hash("123456")
ok = check_password_hash(h, "123456")

// Node / crypto nativo
const crypto = require('crypto');
function hashPassword(password, salt = crypto.randomBytes(16).toString('hex')) {
  const hash = crypto.scryptSync(password, salt, 64).toString('hex');
  return `${salt}:${hash}`;
}

11.4 Cuidado com o criterio 5.4

A lista explicita proteger pelo menos a senha e o CPF do cliente. Tecnicamente, senha deve ser armazenada como hash com salt; CPF, quando a regra exige criptografia, deve usar cifra reversivel com chave. Nao confunda hash com criptografia.


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

12. Implementacao alinhada aos criterios: Cliente

Criterio

Endpoint de estudo

Teste no terminal

5.1 listar

GET /clientes

curl -i GET + conferir 200

5.1 excluir

DELETE /clientes/:id

DELETE + SELECT/GET

5.2 busca

GET /clientes?busca=ana

termo reduz/altera listagem

5.3 inserir

POST /clientes

POST + SELECT

5.4 proteger dados

POST/PATCH /clientes

SELECT e conferir que nao estao em texto puro

5.5 editar

PATCH /clientes/:id

PATCH + GET/SELECT

12.1 Exemplo de payload

{
  "nome": "Ana Lima",
  "cpf": "00000000000",
  "senha": "123456",
  "telefone": "24999999999"
}

O JSON acima e didatico. Os nomes exatos dependem do seu DER/Anexo 1. O importante para a pontuacao e que inserir, editar, excluir, buscar e persistir realmente funcionem.

12.2 Erros de cliente e resposta correta

Situacao

Nao faça

Melhor comportamento

cliente inexistente

500

404 ou 400 tratado

CPF/email duplicado

500

409 ou mensagem de duplicidade

campo faltando

excecao

400/422 com validacao

cliente com dependencias

DELETE forçado

tratar FK/conflito


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

13. Instrumento e Agendamento

13.1 Instrumento ligado a cliente

POST /instrumentos
{
  "nome": "Violao classico",
  "tipo": "Cordas",
  "cliente_id": 1
}

O cliente_id precisa existir. Se nao existir, o banco deve impedir o registro por FK ou o backend deve validar e devolver erro controlado.

13.2 Agendamento ordenado

SELECT
    a.id_agendamento,
    a.data_agendamento,
    c.nome AS cliente,
    i.nome AS instrumento,
    t.nome AS tecnico
FROM agendamento a
JOIN cliente c ON c.id_cliente = a.cliente_id
JOIN instrumento i ON i.id_instrumento = a.instrumento_id
JOIN tecnico t ON t.id_tecnico = a.tecnico_id
ORDER BY a.data_agendamento ASC;

13.3 Atualizacao da data

PATCH /agendamentos/1
{
  "data_agendamento": "2026-10-10T14:30:00"
}

Depois do PATCH/PUT, confirme no PostgreSQL com SELECT. Alterar apenas a tela nao atende ao requisito de persistencia.

13.4 Datas: formato

Quando o backend usa data/hora tipada, um formato ISO como 2026-10-10T14:30:00 costuma ser o caminho mais previsivel. Adapte ao formato definido pelo seu modelo e framework.


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

14. Teste completo pelo terminal com curl

Esta secao permite verificar o backend sem navegador. Ajuste porta, rota e nomes de campos ao seu projeto.

14.1 Health check local

curl -i http://127.0.0.1:5000/
curl -i http://127.0.0.1:3000/

14.2 Login

curl -i -X POST http://127.0.0.1:5000/login   -H 'Content-Type: application/json'   -d '{"email":"usuario@teste.local","senha":"123456"}'

14.3 Rota protegida

curl -i http://127.0.0.1:5000/clientes   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'

14.4 Cliente

curl -i http://127.0.0.1:5000/clientes   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'

curl -i 'http://127.0.0.1:5000/clientes?busca=ana'   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'

curl -i -X POST http://127.0.0.1:5000/clientes   -H 'Content-Type: application/json'   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'   -d '{"nome":"Ana Lima","cpf":"00000000000","senha":"123456"}'

curl -i -X PATCH http://127.0.0.1:5000/clientes/1   -H 'Content-Type: application/json'   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'   -d '{"nome":"Ana Lima Atualizada"}'

curl -i -X DELETE http://127.0.0.1:5000/clientes/1   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'

14.5 Instrumento e agendamento

curl -i -X POST http://127.0.0.1:5000/instrumentos   -H 'Content-Type: application/json'   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'   -d '{"nome":"Violao classico","cliente_id":2}'

curl -i http://127.0.0.1:5000/agendamentos   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'

curl -i -X PATCH http://127.0.0.1:5000/agendamentos/1   -H 'Content-Type: application/json'   -H 'Authorization: Bearer COLE_SEU_TOKEN_AQUI'   -d '{"data_agendamento":"2026-10-10T14:30:00"}'

14.6 Testes negativos

# senha errada
curl -i -X POST http://127.0.0.1:5000/login   -H 'Content-Type: application/json'   -d '{"email":"usuario@teste.local","senha":"ERRADA"}'

# sem token
curl -i http://127.0.0.1:5000/clientes


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

15. Empacotamento e conferencia da entrega

O caderno exige uma pasta compactada com nome completo, contendo documentacao.pdf, o DER em imagem/PDF, jubal_db.sql e o diretorio treino com os codigos das funcionalidades.

15.1 Estrutura final

Nome Completo.zip
├── documentacao.pdf
├── DER.pdf
├── jubal_db.sql
└── treino/
    ├── login.*
    ├── principal.*
    ├── cliente.*
    ├── instrumento.*
    └── agendamento.*

15.2 Converter documentacao para PDF via terminal

mkdir -p entrega
libreoffice --headless --convert-to pdf --outdir entrega documentacao.docx
ls -lh entrega/

15.3 Compactar e conferir

cd ~/Desktop
zip -r "Nome Completo.zip" documentacao.pdf DER.pdf jubal_db.sql treino/
unzip -l "Nome Completo.zip"

15.4 Conferencia do SQL antes de fechar

grep -n "CREATE TABLE" jubal_db.sql
grep -n "INSERT INTO" jubal_db.sql
file -bi jubal_db.sql

Se o ZIP estiver certo mas a aplicacao depender de arquivos fora dele, o trabalho fica frágil. Use caminhos relativos e confira o conteudo do ZIP antes de considerar a entrega pronta.


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

16. Checklist final de 35/35

Item

OK

Evidencia para conferir

1.1

[ ]

4 ou mais requisitos funcionais documentados

1.2

[ ]

SGBD e linguagem conforme Anexo 1

1.3

[ ]

DER bate com banco e relacionamentos

2.1

[ ]

FKs respeitadas

2.2

[ ]

3+ registros em TODA tabela criada

3.1

[ ]

login com credencial do banco

3.2

[ ]

sessao/token expira no tempo do caderno

3.3

[ ]

falha de autenticacao tratada

4.1

[ ]

nome do usuario aparece no principal

4.2

[ ]

logout funciona

4.3

[ ]

Principal acessa Cliente, Instrumento, Tecnico, Agendamento

5.1

[ ]

listar e excluir clientes

5.2

[ ]

buscar cliente por termo

5.3

[ ]

inserir cliente persiste no banco

5.4

[ ]

senha e CPF protegidos

5.5

[ ]

editar cliente persiste

6.1

[ ]

instrumento associado a cliente

7.1

[ ]

agendamentos ordenados por data + cliente + instrumento + tecnico

7.2

[ ]

data do agendamento atualizavel

16.1 Ordem final de testes

1. PostgreSQL: pg_isready + SELECT current_database().

2. SQL: \dt + contagem de registros + FKs.

3. Backend: syntax check + inicializacao sem traceback.

4. Login: correto, errado, token ausente, token expirado.

5. Principal: nome, logout e quatro acessos.

6. Cliente: listar, buscar, inserir, editar, excluir.

7. Instrumento: inserir com cliente existente.

8. Agendamento: listar ordenado + atualizar data.

9. Gerar PDF/DER/ZIP e executar unzip -l.

17. Folha de emergencia

APLICACAO NAO INICIA
-> Java: java -version; javac -version; ./mvnw -o -DskipTests compile OU ./gradlew --offline compileJava
-> Python: python -m py_compile app.py
-> Node: node --check app.js
-> depois: dependencia / configuracao / porta / log da primeira causa

401
-> login / sessao-cookie OU Authorization Bearer / validade / assinatura / exp

403
-> permissao / role

404
-> URL / metodo / ID / prefixo do router

415
-> Content-Type: application/json

500
-> NAO ADIVINHE
-> leia o terminal
-> procure a ultima excecao

PYTHON + POSTGRES
-> pg_isready
-> psql -d jubal_db
-> SELECT current_database(), current_user;
-> \dt
-> \d tabela

NODE + EACCES
-> ls -la
-> find ... -not -user "$USER"
-> chown do projeto e ~/.npm
-> npm install sem sudo

JAVA / SPRING
APPLICATION FAILED TO START -> ler Description/Action/Caused by
Port 8080 already in use -> ss -ltnp | grep :8080
DataSource/driver -> JDBC URL + dependencia PostgreSQL
BeanCreationException -> primeira causa interna
Not a managed type -> @Entity / @Id / pacote escaneado

POSTGRES
relation does not exist -> banco/schema/tabela
column does not exist   -> nome da coluna
foreign key violation   -> pai nao existe
duplicate key           -> UNIQUE/PK
authentication failed   -> usuario/senha
permission denied       -> GRANT / owner / pg_hba

FINAL
-> COUNT(*) >= 3 em cada tabela
-> login OK + login errado
-> Cliente 5.1 a 5.5 OK
-> Instrumento OK
-> Agendamento 7.1 e 7.2 OK
-> documentacao.pdf + DER + jubal_db.sql + treino
-> zip + unzip -l

A melhor forma de ganhar tempo e diagnosticar de baixo para cima: sistema operacional -> processo/porta -> linguagem -> framework -> autenticao -> SQL -> PostgreSQL -> regra de negocio.

FONTE E LIMITES: esta apostila foi montada a partir do caderno da avaliacao e da lista de verificacao fornecidos. O Anexo 1 completo nao aparece nas paginas recebidas; por isso nao foi inventado o valor da expiracao da sessao nem a versao exata de infraestrutura. Confira esses pontos no material completo antes da prova.

18. Windows PowerShell: equivalentes essenciais

A maior parte dos comandos de servico deste guia usa Linux/Bash. Se o posto estiver em Windows, nao execute sudo, systemctl, ss, lsof ou source no PowerShell. Use os equivalentes abaixo; confirme o nome exato do servico e o caminho instalado. Referencias oficiais: Microsoft Learn e documentacao Python [R17-R20].

18.1 Identificar o terminal e as ferramentas

Get-Location
whoami
$PSVersionTable.PSVersion
Get-Command python,py,node,npm,java,javac,mvn,gradle,psql -ErrorAction SilentlyContinue
py --version
python --version
node --version
npm --version
java -version
javac -version
psql --version

Se Get-Command nao localizar um programa, isso pode significar que nao esta instalado ou que nao esta no PATH. Nao instale pela internet durante a prova; procure os arquivos/pacotes locais autorizados ou informe ao avaliador se o ambiente fornecido estiver incompleto.

18.2 Verificar/iniciar PostgreSQL e verificar portas

Get-Service *postgres*
# Copie o Name exato retornado e substitua o marcador abaixo
Get-Service -Name "NOME_EXATO_DO_SERVICO"
Start-Service -Name "NOME_EXATO_DO_SERVICO"
Test-NetConnection 127.0.0.1 -Port 5432
Get-NetTCPConnection -State Listen -LocalPort 5432,5000,3000,8080 -ErrorAction SilentlyContinue | Select-Object LocalAddress,LocalPort,OwningProcess
Get-Process -Id NUMERO_PID

Start-Service normalmente exige PowerShell elevado e permissoes administrativas. Use somente se tiver autorizacao; se nao, nao altere servicos/permissoes por conta propria. Nomes como postgresql-x64-17 variam por versao/instalacao: descubra com Get-Service *postgres* em vez de adivinhar [R17-R19].

18.3 Testar/importar SQL por TCP no Windows

psql -h 127.0.0.1 -p 5432 -U postgres -d jubal_db -W -c "SELECT current_database(), current_user;"
psql -v ON_ERROR_STOP=1 -h 127.0.0.1 -p 5432 -U postgres -d jubal_db -f .\jubal_db.sql

Se psql nao for reconhecido, o cliente PostgreSQL pode nao estar no PATH; localize a instalacao ou use o terminal fornecido no posto. Um teste com credencial/porta explicita e mais representativo da conexao TCP usada pela aplicacao do que uma sessao administrativa por socket/peer no Linux.

18.4 Ambiente virtual Python, Node e Java no PowerShell

# Python: se a ativacao do venv for bloqueada, use o python.exe direto abaixo
py -m venv .venv
.\.venv\Scripts\Activate.ps1
.\.venv\Scripts\python.exe -m pip --version
.\.venv\Scripts\python.exe -m py_compile .\app.py
.\.venv\Scripts\python.exe .\app.py

# Node
node --check .\app.js
npm ls --depth=0
npm start

# Maven/Gradle Wrapper no Windows
.\mvnw.cmd -o -DskipTests package
.\gradlew.bat --offline build
java -jar .\target\NOME-REAL.jar

Se a politica do PowerShell bloquear Activate.ps1, nao e necessario desativar a politica global: invoque .\.venv\Scripts\python.exe diretamente. Substitua app.py/app.js pelo nome real do projeto.

18.5 HTTP e problemas de permissao no Windows

Use curl.exe explicitamente para evitar confusao com alias do PowerShell. Exemplo: curl.exe -i http://127.0.0.1:8080/ . Para projeto com erro de acesso a arquivo/pasta, execute em uma pasta gravavel do perfil do usuario, confira Get-Acl CAMINHO e Get-Process -Id PID; nao rode terminal como Administrador nem mude ACL sem entender o motivo.

19. SQLSTATE PostgreSQL: codigos de erro para diagnostico

Quando o log mostra SQLSTATE/codigo de cinco caracteres, use-o junto da mensagem. A classificacao oficial do PostgreSQL e uma forma rapida de distinguir conexao, permissao, sintaxe e integridade [R6].

SQLSTATE

Significado

Proximo passo

08001 / 08006

Falha ao estabelecer/manter conexao

host, porta, servico, rede/credenciais e log PostgreSQL.

28P01 / 28000

Senha/autenticacao invalida

usuario, senha, metodo de autenticacao; testar psql via TCP.

3D000

Database inexistente

corrigir nome ou criar a base antes de importar o script.

3F000

Schema inexistente

conferir schema e qualificacao public.tabela.

42501

Privilegio insuficiente

GRANT/owner/database/schema/table/sequence.

42601

Erro de sintaxe SQL

ler a posicao indicada; testar trecho no psql.

42P01 / 42703

Tabela/coluna inexistente

banco/schema/nome real/ORM/SQL.

42P07 / 42701

Tabela/coluna duplicada

script executado novamente ou DDL repetido.

22P02 / 22007 / 22008

Representacao de dado/data invalida

tipo, formato ISO, decimal/inteiro e conversao.

23502 / 23503 / 23505 / 23514

NOT NULL / FK / UNIQUE / CHECK

validar entrada e relacoes antes do INSERT/UPDATE.

25P02

Transacao atual esta abortada

ROLLBACK; corrigir o primeiro erro que ocorreu antes.

40001 / 40P01

Falha de serializacao / deadlock

repetir transacao com cuidado; encontrar concorrencia/origem do deadlock.

53100 / 53300

Disco cheio / conexoes demais

ver df -h; consultar pg_stat_activity; nao matar conexoes aleatoriamente.

57014 / 57P03

Consulta cancelada / servidor indisponivel para conectar

tempo limite, carga, reinicio ou log do servidor.

20. Referencias tecnicas oficiais consultadas

As referencias abaixo fundamentam os diagnosticos tecnicos e devem ser estudadas antes da prova. Como o caderno fornecido proibe internet durante a avaliacao, nao dependa de abrir estes links no posto de trabalho. A fonte dos criterios de pontuacao continua sendo a lista de verificacao SAEP e o caderno fornecidos pelo usuario.

R1. Spring Boot - SQL databases, DataSource, JDBC e JPA: https://docs.spring.io/spring-boot/reference/data/sql.html

R2. Spring Boot - falhas de inicializacao / Failure Analyzers: https://docs.spring.io/spring-boot/reference/features/spring-application.html

R3. Spring Boot - logs, niveis de log e --debug: https://docs.spring.io/spring-boot/reference/features/logging.html

R4. Spring Security - arquitetura de filtros e autenticacao: https://docs.spring.io/spring-security/reference/servlet/architecture.html

R5. Spring Security - Resource Server JWT e Bearer tokens: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html

R6. PostgreSQL - codigos de erro SQLSTATE: https://www.postgresql.org/docs/current/errcodes-appendix.html

R7. PostgreSQL - autenticacao de clientes e pg_hba.conf: https://www.postgresql.org/docs/current/client-authentication.html

R8. Spring Framework - hierarquia/traducao de excecoes de acesso a dados: https://docs.spring.io/spring-framework/reference/data-access/dao.html

R9. Java - compilador javac e opcoes de compilacao: https://docs.oracle.com/en/java/javase/21/docs/specs/man/javac.html

R10. Maven - configuracao de plugins e dependencias de build: https://maven.apache.org/guides/mini/guide-configuring-plugins.html

R11. Flask - tratamento de excecoes e erros 500: https://flask.palletsprojects.com/en/stable/errorhandling/

R12. Flask - quickstart e conversao de respostas/JSON: https://flask.palletsprojects.com/en/stable/quickstart/

R13. Node.js - codigos de erro do sistema (EACCES, EADDRINUSE, ECONNREFUSED): https://nodejs.org/api/errors.html

R14. Express - tratamento de erros e middleware: https://expressjs.com/en/guide/error-handling/

R15. node-jsonwebtoken - tipos e mensagens de erro JWT: https://github.com/auth0/node-jsonwebtoken/blob/master/README.md

R16. PyJWT - excecoes de token/assinatura/expiracao: https://pyjwt.readthedocs.io/en/stable/api.html

R17. Microsoft Learn - Get-Service: https://learn.microsoft.com/pt-br/powershell/module/microsoft.powershell.management/get-service?view=powershell-7.6

R18. Microsoft Learn - Start-Service: https://learn.microsoft.com/pt-br/powershell/module/microsoft.powershell.management/start-service?view=powershell-7.5

R19. Microsoft Learn - Get-NetTCPConnection e portas/processos TCP: https://learn.microsoft.com/pt-br/powershell/module/nettcpip/get-nettcpconnection?view=windowsserver2025-ps

R20. Python - criacao e ativacao de ambientes virtuais em Windows/PowerShell: https://docs.python.org/3/library/venv.html


SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 

APENDICE A - DICIONARIO COMENTADO DOS COMANDOS

Este apendice explica os comandos que aparecem no guia. Leia a coluna “O que faz” para saber a finalidade do comando; use “Como interpretar” para entender a saida ou decidir o proximo passo. Os comandos sao exemplos: ajuste caminhos, nomes de banco, portas e credenciais ao computador da prova.

Importante: use os comandos da secao correspondente ao sistema operacional. Comandos Linux/macOS (por exemplo, sudo systemctl e source) nao sao comandos nativos do PowerShell. No Windows, use as alternativas PowerShell apresentadas adiante.

A.1 - Como ler a sintaxe antes de executar

Comando / sintaxe

O que faz

Como interpretar / cuidado

COMANDO --version / -v

Normalmente exibe a versao instalada do programa. Ex.: python --version, node -v, npm -v, psql --version.

Se o comando nao for reconhecido, o programa pode nao estar instalado ou pode estar fora do PATH.

comando -h / --help

Pede a ajuda de um programa, quando ele oferece essa opcao.

Mostra opcoes e sintaxe; nem todo programa suporta exatamente -h ou --help.

python -m MODULO

Executa um modulo Python usando aquele interpretador. Ex.: python -m pip garante que o pip pertence a esse Python.

Ajuda a evitar instalar pacote em um Python diferente daquele que executa a aplicacao.

python -c "CODIGO"

Executa uma pequena instrucao Python diretamente na linha de comando.

Bom para testar imports, variaveis e conexao sem iniciar o servidor completo.

sudo -u USUARIO comando

Executa um comando como outro usuario do sistema. Ex.: sudo -u postgres psql.

Use somente quando necessario e com usuario correto; nao e o mesmo que usuario/senha do banco dentro da aplicacao.

comando A && comando B

Executa B somente se A terminar com sucesso.

Bom para sequencias em que o segundo passo depende do primeiro.

comando A || comando B

Executa B somente se A falhar.

Pode servir como alternativa, mas nao o use para esconder um erro importante.

comando A | comando B

Envia a saida de A para a entrada de B (pipe).

Ex.: ss ... | grep 8080 filtra a saida para mostrar linhas relevantes.

2>/dev/null

Descarta mensagens de erro (stderr) no Linux/Unix.

Pode deixar a tela limpa, mas tambem esconder a causa; remova durante o diagnostico.

2>&1

Redireciona stderr para o mesmo destino de stdout.

Comum para guardar saida e erros juntos em um log.

> arquivo / tee arquivo

Redireciona a saida para um arquivo; tee exibe na tela e grava uma copia.

Use tee quando quiser ver a inicializacao e guardar o log para procurar Caused by/ERROR depois.

~ / . / .. / ./programa

~ representa a pasta pessoal; . a pasta atual; .. a pasta pai; ./programa executa um arquivo da pasta atual.

Caminho relativo depende da pasta em que voce esta. Use pwd para conferir.

NOME-REAL.jar / <CAMINHO> / NOME_EXATO

Sao marcadores para substituir, nao valores literais.

Troque pelo nome real encontrado com ls/find ou Get-ChildItem; nao inclua os sinais < > ao copiar caminhos de exemplo.

MAIUSCULAS_DE_EXEMPLO

Indica valor que precisa ser personalizado, como SENHA, TOKEN, PID ou nome do servico.

Substitua pelo valor correto. Nao compartilhe senha/token real em logs ou documentos.

HTTP: -H, -X, -d, -i, -v

Opcoes de curl: -H envia header; -X define metodo; -d envia corpo; -i mostra headers da resposta; -v mostra detalhes da conexao.

Use -H Content-Type para JSON e -i para ver status como 200, 401 ou 500.

psql: -h, -p, -U, -d, -W, -c, -f

Opcoes: host, porta, usuario, banco, pedir senha, executar SQL de uma linha, executar SQL de arquivo.

Se conectar ao banco errado, confira -h, -p, -U e -d antes de modificar dados.

Maven/Gradle: -o / --offline

Pede ao gerenciador de build para nao acessar a internet.

Funciona apenas se plugins e dependencias necessarios estiverem disponiveis localmente.

Maven: -DskipTests

Define a propriedade que faz o build pular a execucao dos testes naquela chamada.

Ajuda a compilar/empacotar rapido para diagnostico; nao prova que os testes passam.

A.2 - Sistema operacional e terminal Linux

Comando

O que faz

Como interpretar / cuidado

pwd

Mostra a pasta atual do terminal.

Se o arquivo nao e encontrado, confira se voce esta na pasta correta.

whoami

Mostra o nome do usuario do sistema que esta executando o terminal.

Ajuda a diagnosticar propriedade de arquivos e permissoes.

cat /etc/os-release

Mostra a distribuicao Linux e sua versao.

Use para escolher comandos compativeis com o sistema instalado.

uname -a

Mostra informacoes do kernel e arquitetura do sistema.

Util para identificar ambiente; nao instala nem altera nada.

lscpu

Mostra informacoes da CPU. No guia, sed limita a exibicao as primeiras linhas.

E um inventario do equipamento, nao um teste da aplicacao.

sed -n 1,12p

Imprime somente as linhas 1 a 12 do texto recebido.

A barra vertical antes dele faz receber a saida do comando anterior.

free -h

Mostra memoria RAM/swap em unidades legiveis.

Se faltar memoria, processos podem falhar; -h significa human-readable.

df -h

Mostra espaco livre/usado nos sistemas de arquivos montados.

Disco cheio pode impedir instalar dependencias ou gravar logs.

mkdir -p PASTA

Cria pasta e pastas-pai que ainda nao existirem; nao falha so porque elas ja existem.

Use para montar a estrutura treino sem criar cada nivel manualmente.

cd PASTA

Muda a pasta atual do terminal.

Caminhos relativos passam a ser avaliados a partir da nova pasta.

ls -la

Lista arquivos, inclusive ocultos, com detalhes como permissao, dono e tamanho.

Use para procurar .env, package.json, pom.xml, arquivos SQL e conferir proprietario.

find . -maxdepth N -type f ...

Procura arquivos a partir da pasta atual, limitando a profundidade e os tipos/nome solicitados.

Use para localizar pom.xml, mvnw, gradlew, app.js ou outros arquivos. . significa “aqui”.

command -v NOME

Mostra o caminho do executavel que o shell encontra para aquele nome.

Se nao retorna caminho, o executavel pode estar ausente ou fora do PATH.

grep -nE PADRAO ARQUIVO

Procura texto usando expressao regular estendida; -n inclui numero da linha.

Util para achar ERROR, Caused by e CREATE TABLE dentro de logs/SQL.

tail -n 100 ARQUIVO

Exibe as ultimas 100 linhas de um arquivo.

Logs geralmente mostram a causa real perto do fim; adapte a quantidade se preciso.

file -bi ARQUIVO

Tenta identificar tipo de arquivo e codificacao MIME/charset.

Ajuda a detectar arquivo SQL com codificacao inesperada.

zip -r ARQUIVO.zip ITENS

Cria um ZIP recursivamente, incluindo arquivos dentro de pastas.

Confira os nomes e pastas incluidos antes de entregar.

unzip -l ARQUIVO.zip

Lista o conteudo do ZIP sem extrair os arquivos.

Confirme documentacao.pdf, DER, SQL e treino dentro do arquivo final.

libreoffice --headless --convert-to pdf ...

Converte documento DOCX para PDF sem abrir interface grafica.

Se LibreOffice nao estiver instalado no posto, use uma alternativa disponivel; isso nao faz parte da logica da API.

A.3 - Python, pip, venv e Flask

Comando / trecho

O que faz

Como interpretar / cuidado

python --version / python3 --version / py --version

Exibe a versao do interpretador Python encontrado.

Se python e py mostrarem versoes diferentes, confirme qual deles esta usando para criar o ambiente e executar app.py.

python -m pip --version

Mostra a versao do pip e o caminho de Python ao qual pertence.

E preferivel a chamar pip sozinho porque reduz a chance de usar outro ambiente.

python -m pip list

Lista os pacotes instalados no ambiente Python ativo.

Procure Flask, psycopg2-binary, PyJWT e outras dependencias usadas pelo codigo.

python -m pip show PACOTE

Mostra detalhes de um pacote instalado, como versao e localizacao.

Se nao encontra pacote, o pacote esta ausente daquele ambiente.

python -m venv .venv

Cria um ambiente virtual Python na pasta .venv.

Isola dependencias do projeto das dependencias globais. Se .venv ja existir, confira antes de recriar.

source .venv/bin/activate

Ativa o ambiente virtual no shell Linux/macOS atual.

O prompt pode mostrar (.venv); a ativacao muda qual python/pip sera encontrado.

.\.venv\Scripts\Activate.ps1

Ativa o ambiente virtual no PowerShell do Windows.

Se a politica de execucao bloquear o script, use o python.exe do venv diretamente como mostrado no guia.

python -m py_compile ARQUIVO.py

Verifica a sintaxe de um arquivo Python e gera bytecode, sem rodar a aplicacao normalmente.

Sem saida costuma indicar sintaxe valida; nao garante conexao ao banco nem funcionamento das rotas.

python -m compileall PASTA

Compila/valida sintaxe dos arquivos Python sob a pasta indicada.

Use para localizar erros de sintaxe antes de iniciar o Flask.

python -c "import flask; print(...)"

Tenta importar Flask e imprime uma mensagem se funcionar.

ModuleNotFoundError indica dependencia ausente ou interpretador/venv errado.

python -c "import psycopg2; print(...)"

Testa se o driver psycopg2 pode ser importado.

Se falhar, confira a instalacao do driver no ambiente ativo.

python -c "import jwt; print(...)"

Testa importacao do modulo jwt normalmente fornecido por PyJWT.

O nome do pacote na instalacao e PyJWT; o nome do modulo importado e jwt.

python -m pip install --no-index --find-links ./packages ...

Instala pacotes sem consultar indice de internet, procurando arquivos locais em ./packages.

A pasta precisa conter os pacotes e dependencias compativeis com o Python/OS. Sem isso a instalacao falha.

python treino/app.py

Executa o arquivo app.py usando o interpretador ativo.

Leia o terminal; “Running on...” indica que o servidor iniciou, mas ainda teste as rotas.

python -m pip --version / sys.executable

sys.executable mostra o caminho exato do Python em execucao.

Use print(sys.executable) para confirmar se e o Python dentro de .venv.

python - <<'__PY__' ... __PY__

Executa um script Python enviado diretamente pelo terminal via heredoc (Linux/macOS).

Bom para testar uma conexao isolada sem criar arquivo; nao e sintaxe nativa do PowerShell.

try/except + print(type(e).__name__, e)

Nao e comando de terminal: e instrumentacao temporaria do codigo para mostrar classe e mensagem da excecao.

Remova credenciais pessoais dos logs e use raise quando quiser preservar o traceback durante o diagnostico.

A.4 - PostgreSQL e psql

Comando / SQL

O que faz

Como interpretar / cuidado

psql --version

Mostra a versao do cliente psql instalado.

Nao prova que o servidor PostgreSQL esta rodando; verifica so o cliente.

pg_isready

Verifica se o servidor PostgreSQL esta aceitando conexoes no host/porta padrao ou configurados.

“accepting connections” e sinal positivo; “no response” pede checagem do servico/porta.

systemctl status postgresql --no-pager

Exibe o estado do servico no Linux sem abrir paginador.

active (running) indica ativo; inactive/failed indica que precisa investigar. Nao e comando PowerShell.

systemctl start postgresql

Inicia o servico PostgreSQL no Linux.

Use se estiver parado e tiver permissao; nao apaga nem cria bases.

systemctl restart postgresql

Para e inicia de novo o servico PostgreSQL.

Pode interromper conexoes; nao use sem necessidade se start/resolver config bastar.

sudo -u postgres psql -c "SELECT version();"

Abre psql como usuario do sistema postgres e executa uma consulta que mostra a versao do servidor.

Se falhar, verifique servico e modo de autenticacao local.

SELECT datname FROM pg_database ORDER BY datname;

Lista os nomes dos bancos de dados e os ordena alfabeticamente.

Confirme se jubal_db aparece antes de criar/importar.

CREATE DATABASE jubal_db;

Cria um banco chamado jubal_db.

Execute no contexto autorizado; se ja existir, a instrucao falha. Nao apague dados para resolver sem conferir.

psql -d jubal_db

Abre uma sessao interativa conectada ao banco jubal_db.

Dentro do psql, os comandos \d e \dt sao metacomandos do cliente, nao SQL padrao.

\conninfo

Mostra informacoes da conexao atual do psql.

Use para identificar host, usuario, porta e banco da sessao.

SELECT current_database(), current_schema(), current_user;

Mostra banco, schema e usuario efetivamente usados naquela sessao.

E uma das melhores verificacoes para detectar conexao na base errada.

\dn

Lista os schemas do banco.

Se a tabela existe em schema nao padrao, investigue qualificacao e search_path.

\dt

Lista as tabelas visiveis no schema/search_path atual.

Se nao aparecer cliente, confira banco, schema e se o SQL foi executado.

\d cliente

Mostra estrutura da tabela cliente: colunas, tipos e restricoes.

Compare nomes/tipos com as consultas e modelos do programa.

\q

Sai do cliente psql.

Nao apaga nem altera dados por si so.

psql -v ON_ERROR_STOP=1 -d jubal_db -f jubal_db.sql

Executa o arquivo SQL no banco escolhido e manda o psql parar no primeiro erro SQL.

O codigo de saida denuncia falha. Leia a primeira mensagem de erro: ela costuma ser a causa original.

psql -h HOST -p PORTA -U USUARIO -d BANCO -W

Escolhe explicitamente host, porta, usuario e banco; -W pede a senha no terminal.

Bom para testar conexao TCP como a aplicacao faria. Nao coloque senha na linha de comando.

GRANT CONNECT ON DATABASE ...

Concede ao usuario direito de conectar ao banco nomeado.

Nao concede automaticamente SELECT/INSERT nas tabelas.

GRANT USAGE ON SCHEMA public ...

Permite usar/resolver objetos do schema public.

Sem acesso ao schema, o usuario pode nao conseguir encontrar objetos mesmo se a tabela existe.

GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES ...

Concede leitura, inclusao, atualizacao e exclusao nas tabelas existentes do schema.

Use o usuario real da aplicacao; conceder permissao nao corrige nome de tabela ou conexao errada.

GRANT USAGE, SELECT ON ALL SEQUENCES ...

Concede uso/leitura de sequences, inclusive as associadas a IDs SERIAL em cenarios comuns.

Sem isso, INSERT pode falhar ao gerar o proximo ID.

COUNT(*)

Conta quantas linhas existem no conjunto consultado.

No SAEP, use para confirmar pelo menos tres registros em cada tabela criada.

UNION ALL

Combina resultados de varias consultas preservando linhas repetidas.

Util no checklist que devolve o total de linhas de varias tabelas em uma unica saida.

pg_terminate_backend(pid)

Encerra uma conexao de outro processo ao PostgreSQL, caso tenha permissao.

Usado no reset de ambiente para liberar conexoes antes de apagar uma base. Pode derrubar trabalho alheio.

DROP DATABASE IF EXISTS jubal_db;

Apaga a base jubal_db se ela existir.

ACAO DESTRUTIVA. Use apenas em ambiente de treino descartavel, apos confirmar que nao ha dados a preservar.

psycopg2.OperationalError

Nao e comando: e uma excecao comum do driver Python para falhas de conexao/operacao.

Leia a mensagem final: pode ser banco inexistente, credencial, host, porta ou servidor indisponivel.

SQLSTATE 28P01 / 3D000 / 42P01

Codigos do PostgreSQL: autenticacao falhou / banco inexistente / tabela-relacao inexistente.

O codigo ajuda a classificar a falha independentemente da linguagem cliente.

A.5 - Node.js, npm e Express

Comando / mensagem

O que faz / significa

Como interpretar / cuidado

node -v / node --version

Mostra a versao do runtime Node.js.

Se nao reconhecido, verifique instalacao/PATH com command -v node ou Get-Command node.

npm -v / npm --version

Mostra a versao do gerenciador de pacotes npm.

Node pode existir sem npm corretamente configurado; teste ambos.

npm init -y

Cria package.json com configuracao inicial padrao sem perguntas interativas.

Se ja houver package.json, leia antes de substituir; esse arquivo define metadados e scripts.

npm ls express pg jsonwebtoken --depth=0

Lista as dependencias de primeiro nivel indicadas e suas versoes.

Mostra missing/empty se pacote nao estiver instalado no projeto atual.

npm cache verify

Verifica a integridade do cache local do npm.

Nao instala dependencias; ajuda a investigar problemas no cache.

npm config get cache

Mostra a pasta usada como cache pelo npm.

Use para investigar EACCES no cache.

npm config get prefix

Mostra o prefixo usado pelo npm para instalacoes globais.

Ajuda a diagnosticar PATH/permissoes de pacotes globais.

npm install

Instala as dependencias listadas em package.json e atualiza node_modules/lockfile conforme o contexto.

Se sem rede, pode falhar se faltar pacote local/cache. Leia a mensagem exata.

npm install --offline

Tenta instalar apenas usando dados/cache local, sem buscar na rede.

Falha se alguma dependencia nao estiver no cache.

npm install ./packages/arquivo.tgz

Instala um pacote a partir de um arquivo tarball local.

O arquivo precisa existir e ser o pacote correto; dependencias transitivas tambem precisam estar disponiveis.

npm start

Executa o script “start” definido no package.json.

Se nao existe script start, o npm informa. Confira “scripts” no package.json.

node --check treino/app.js

Verifica sintaxe JavaScript do arquivo sem iniciar o servidor.

Nao valida conexao a banco, JWT ou funcionamento das rotas.

node treino/app.js

Executa diretamente o arquivo JavaScript.

Use o caminho real do arquivo; o terminal fica preso enquanto o servidor roda.

sudo chown -R "$USER":"$USER" PASTA

No Linux, muda recursivamente o dono/grupo dos arquivos para o usuario atual.

Use somente no projeto/cache sob seu controle; nao execute em pastas de sistema indiscriminadamente.

npm install express / pg / jsonwebtoken

Instala bibliotecas para rotas HTTP, acesso ao PostgreSQL e JWT, respectivamente.

Na prova sem internet, confira se pacotes e cache estao disponiveis antes.

MODULE_NOT_FOUND

Nao e comando: erro Node que indica um modulo importado nao encontrado.

Confira nome do pacote, pasta do projeto, package.json e resultado de npm ls.

EACCES / EADDRINUSE / ECONNREFUSED

Codigos de erro do sistema no Node: sem permissao / porta ocupada / conexao recusada.

Identifique a camada pelo codigo antes de mudar o codigo da aplicacao.

A.6 - Java, JDK, Maven, Gradle e Spring Boot

Comando / mensagem

O que faz / significa

Como interpretar / cuidado

java -version

Mostra a versao do runtime Java que sera usado para executar programas.

Nao garante que o compilador esteja instalado; confira tambem javac -version.

javac -version

Mostra a versao do compilador do JDK.

Se java existe mas javac nao, pode haver apenas runtime ou PATH/JAVA_HOME incorreto.

echo "$JAVA_HOME"

Mostra o valor da variavel de ambiente JAVA_HOME no shell Linux.

Normalmente aponta para a raiz do JDK, nao para a pasta bin/java.

mvn -v

Mostra a versao do Maven e o Java que ele esta usando.

Compare o Java indicado com java -version; diferenca pode causar build inconsistente.

gradle -v

Mostra a versao do Gradle instalado e o Java usado.

Se o projeto tem wrapper, verifique ./gradlew; ele seleciona a versao do projeto.

./mvnw -o -DskipTests compile

Usa o Maven Wrapper para compilar sem executar testes e sem internet.

Requer wrapper e dependencias/plugins locais. Falha pode ser JDK, codigo ou dependencias ausentes.

./mvnw -o -DskipTests package

Executa o ciclo de empacotamento Maven em modo offline, pulando testes.

Procure o JAR em target/. Pular testes acelera o diagnostico, nao substitui validar requisitos.

mvn -o -DskipTests package

Faz empacotamento semelhante usando Maven instalado em vez do wrapper.

Use quando Maven estiver instalado e suas dependencias estao no repositorio local.

./gradlew --offline compileJava

Compila o codigo Java com a versao Gradle do wrapper sem acessar rede.

Erros “invalid source/target release” indicam versao de JDK incompatível com o projeto.

./gradlew --offline build

Executa a tarefa de build Gradle em modo offline.

Pode rodar testes conforme configuracao; consulte a mensagem de falha e a tarefa que falhou.

ls -1 target/*.jar

Lista os JARs gerados no diretório target do Maven.

Se nao ha arquivo, o package pode ter falhado ou o projeto usa outro artefato/pasta.

java -jar target/NOME-REAL.jar

Executa o JAR compilado.

Substitua NOME-REAL pelo arquivo real. Leia o log de inicializacao do Spring Boot.

java -jar ... --debug

Inicia o Spring Boot com diagnostico adicional de condicoes de configuracao.

Ajuda em falhas de auto-configuracao; nao corrige sozinho a causa.

./mvnw -o spring-boot:run 2>&1 | tee saep-java.log

Executa Spring Boot via plugin Maven, combina saida/erros e grava em saep-java.log enquanto exibe no terminal.

A execucao pode exigir plugin e dependencias ja baixados; abra o log para procurar Caused by/ERROR.

grep -nE ... app.log | tail -n 100

Filtra linhas importantes do log e exibe as ultimas ocorrencias.

Ajuda a encontrar causa raiz sem ler todo o arquivo; preserve contexto acima/abaixo se necessario.

application.properties: spring.datasource.url

Configuracao, nao comando terminal: define URL JDBC do PostgreSQL para a aplicacao.

A URL especifica host, porta e nome do banco. Compare com os dados testados usando psql.

application.properties: spring.datasource.username/password

Configuram credenciais usadas pelo datasource.

Senha incorreta causa falha de autenticacao; nao publique credenciais reais.

APPLICATION FAILED TO START

Mensagem resumo do Spring Boot, nao a causa raiz.

Leia Description, Action e principalmente a excecao em Caused by.

BeanCreationException / UnsatisfiedDependencyException

Excecoes de inicializacao de beans/dependencias do Spring.

Siga a cadeia de Caused by; a causa pode ser propriedade, bean ausente, repository ou conexao DB.

DataIntegrityViolationException / PSQLException

Excecoes de persistencia; PSQLException frequentemente guarda a mensagem real do PostgreSQL.

Procure SQLSTATE e restricao: FK, UNIQUE, NOT NULL, CHECK, tipo ou tabela.

Not a managed type

Erro JPA/Hibernate: classe nao reconhecida como entidade gerenciada.

Verifique @Entity, @Id, pacote escaneado e configuracao de entidades.

LazyInitializationException

Acesso a relacao lazy quando a sessao/transacao ja nao esta disponivel.

Ajuste consulta/DTO/fronteira transacional; nao force carregamento aleatorio sem entender a relacao.

A.7 - Rede, processos, portas e logs no Linux

Comando

O que faz

Como interpretar / cuidado

ss -ltnp

Lista sockets TCP em estado LISTEN, com enderecos/portas numericos e processo quando permitido.

Use para ver se 5432, 5000, 3000 ou 8080 ja estao ocupadas.

lsof -iTCP -sTCP:LISTEN -P -n

Lista processos com sockets TCP escutando; -P preserva portas numericas, -n evita resolver nomes.

Pode precisar sudo para mostrar dono/processo. Nao encerre PID sem confirmar o processo.

lsof -i :5432

Mostra processos que usam a porta 5432.

Adapte a porta: 5000 Flask, 3000 Node ou 8080 Spring sao apenas convencoes do guia.

journalctl -u postgresql --since '-10 min' --no-pager

Consulta logs recentes do servico PostgreSQL no systemd.

Use se service status nao explicar porque o banco falhou ao iniciar.

curl -i -v URL

Faz requisicao HTTP; -i inclui status/headers de resposta e -v mostra detalhes da conexao.

Connection refused sugere servidor/porta; HTTP 404 ainda prova que algum servidor respondeu.

kill PID

Envia sinal de encerramento ao processo indicado no Linux.

Somente depois de confirmar o PID; prefira encerrar graciosamente quando houver procedimento apropriado.

Test-NetConnection / ss / lsof

Sao ferramentas diferentes por sistema, usadas para provar conectividade e identificar processos/portas.

Uma porta em LISTEN nao garante que a rota ou banco dentro do processo esteja correto.

A.8 - HTTP, curl e testes de endpoints

Comando / opcao

O que faz

Como interpretar / cuidado

curl -i URL

Envia GET por padrao e mostra primeiro os headers/status HTTP.

200 significa sucesso comum; 301/302 redirecionamento; 401 login/token; 403 permissao; 404 rota/recurso; 500 erro interno.

curl -v URL

Mostra diagnostico de DNS/conexao/TLS e detalhes da requisicao.

Use quando nem sequer recebe resposta HTTP; evite publicar headers contendo segredos.

curl -X POST URL

Define o metodo HTTP como POST.

Use para enviar login/cadastro conforme o endpoint espera.

-H 'Content-Type: application/json'

Declara que o corpo enviado esta em formato JSON.

Sem esse header o backend pode nao interpretar o body ou responder 415.

-d '{...}'

Envia o corpo da requisicao. Com JSON, o texto precisa ser JSON valido.

Aspas, virgulas, chaves e tipos precisam estar corretos.

-H 'Authorization: Bearer TOKEN'

Envia o token como credencial de autenticacao HTTP no esquema Bearer.

Substitua TOKEN por token valido. Nao use a palavra TOKEN literal.

-c cookies.txt

Guarda cookies recebidos pelo servidor nesse arquivo.

Serve para fluxo de login baseado em sessao/cookie.

-b cookies.txt

Envia cookies previamente guardados ao fazer outra requisicao.

Sem manter cookie, uma sessao web pode parecer “perdida” entre login e rota protegida.

curl -i -X PATCH URL ...

Solicita alteracao parcial de um recurso, se a API implementa PATCH.

O endpoint pode usar PUT em vez de PATCH; 405 indica metodo nao permitido.

curl -i -X DELETE URL ...

Solicita exclusao do recurso indicado.

Depois valide no banco/GET se foi excluido; cuidado com IDs reais.

localhost / 127.0.0.1

Referem-se a propria maquina; 127.0.0.1 e o loopback IPv4.

Um teste local nao demonstra que outros computadores conseguem acessar o servidor.

HTTP 401 vs 403

401 indica que a autenticacao nao foi estabelecida/aceita; 403 indica acesso proibido pelas regras.

Em algumas configuracoes os detalhes variam, mas essa distincao e a primeira pista.

A.9 - PowerShell no Windows

Comando

O que faz

Como interpretar / cuidado

Get-Location

Mostra a pasta atual (equivalente aproximado a pwd).

Use para conferir de onde os caminhos relativos vao partir.

whoami

Mostra a identidade do usuario atual, se disponivel no terminal Windows.

Ajuda a entender permissoes da conta.

$PSVersionTable.PSVersion

Mostra a versao do PowerShell em uso.

Nao e a versao do Windows nem do Python.

Get-Command python,py,node,npm,java,javac,mvn,gradle,psql

Procura comandos executaveis/aliases disponiveis no PATH.

Se algo nao aparece, provavelmente nao esta instalado ou nao esta no PATH.

Get-Service *postgres*

Lista servicos cujo nome/exibicao corresponde ao padrao postgres.

Copie o Name exato retornado antes de usar Get-Service/Start-Service por nome.

Get-Service -Name NOME

Mostra o estado do servico especifico.

Status Running indica ativo; Stopped indica parado.

Start-Service -Name NOME

Inicia o servico Windows especificado.

Pode exigir PowerShell elevado e o nome correto do servico.

Test-NetConnection 127.0.0.1 -Port 5432

Testa conectividade TCP para o host/porta indicados.

TcpTestSucceeded : True indica que a porta aceita conexao TCP; nao prova autenticacao do banco.

Get-NetTCPConnection -State Listen -LocalPort ...

Lista conexoes TCP em estado LISTEN nas portas indicadas.

Mostra OwningProcess, que pode ser relacionado a Get-Process.

Select-Object LocalAddress,LocalPort,OwningProcess

Seleciona as propriedades especificas a mostrar no resultado PowerShell.

Reduz a saida para facilitar identificar porta e PID.

Get-Process -Id PID

Mostra informacoes do processo com o ID indicado.

Use para descobrir quem ocupa a porta antes de decidir se pode encerrar.

py -m venv .venv

Cria ambiente virtual com o launcher Python do Windows, se instalado.

O ambiente fica na pasta .venv; depois use o executavel dele para garantir o interpretador certo.

.\.venv\Scripts\python.exe -m pip --version

Consulta o pip do ambiente virtual usando o caminho completo do Python.

Nao depende de ativacao do venv no shell.

.\mvnw.cmd -o -DskipTests package

Executa empacotamento Maven pelo wrapper Windows, em modo offline e pulando testes.

Exige mvnw.cmd, JDK e dependencias locais.

.\gradlew.bat --offline build

Executa build Gradle pelo wrapper Windows sem buscar na internet.

Exige gradlew.bat, JDK e dependencias/cache compativeis.

psql -h 127.0.0.1 -p 5432 -U postgres -d jubal_db -W

Testa conexao TCP ao banco com host, porta, usuario e database explicitos, pedindo senha.

E mais representativo da conexao feita por uma aplicacao do que apenas chamar psql sem parametros.

A.10 - SQL e trechos que nao sao comandos de terminal

Trecho

O que faz / significa

Como interpretar / cuidado

CREATE TABLE ...

Cria uma tabela e declara suas colunas, tipos e restricoes.

Execute no psql/arquivo SQL; verifique PK, NOT NULL, UNIQUE e FK antes de inserir dados.

SERIAL PRIMARY KEY

Atalho PostgreSQL tradicional para inteiro com sequence e chave primaria.

A chave identifica linha; se inserir IDs manualmente, a sequence pode precisar ser sincronizada.

REFERENCES tabela(coluna)

Cria uma chave estrangeira para impedir referencia a registro pai inexistente.

Insira o registro pai primeiro, como cliente antes de instrumento.

INSERT INTO ... VALUES ...

Insere registro(s) numa tabela.

O numero/tipo dos valores deve combinar com as colunas; constraints podem rejeitar o registro.

SELECT ... FROM ...

Consulta e retorna dados.

Use WHERE para filtrar e SELECT COUNT(*) para contagem.

WHERE nome ILIKE '%ana%'

Filtra nomes que contenham “ana”, sem diferenciar maiusculas/minusculas no PostgreSQL.

Os % sao curingas. Parametrize a query no codigo para evitar SQL injection.

INNER JOIN ... ON ...

Combina linhas que tenham correspondencia nas tabelas relacionadas.

Se faltar relacionamento correspondente, a linha nao aparece no resultado.

ORDER BY data_agendamento ASC

Ordena da data mais antiga para a mais recente.

A lista de agendamentos precisa refletir a ordem exigida pelo criterio.

UPDATE tabela SET ... WHERE ...

Atualiza colunas de linhas que atendem ao WHERE.

Sem WHERE pode atualizar todas as linhas; teste ID e contagem antes de confirmar.

DELETE FROM tabela WHERE ...

Exclui linhas que atendem ao WHERE.

Sem WHERE pode apagar toda a tabela; observe FKs e use ID especifico.

GRANT / REVOKE

Concede ou retira privilegios no PostgreSQL.

Conceda o minimo que a aplicacao precisa. GRANT CONNECT nao significa permissao nas tabelas.

spring.datasource.url=...

Linha de configuracao de application.properties, nao comando shell.

Define URL JDBC; nao escreva essa linha no prompt do terminal.

Authorization: Bearer ...

Header HTTP de autenticacao, nao comando independente.

Deve fazer parte de uma requisicao curl/Thunder Client; token precisa ser real e valido.

{"nome":"Ana"}

Exemplo de corpo JSON enviado ao endpoint.

JSON e dado, nao comando. Envie com -d e Content-Type correto.

Traceback / Caused by / SQLSTATE

Saida de diagnostico, nao comandos.

Traceback mostra a pilha; Caused by indica causa encadeada em Java; SQLSTATE classifica o erro PostgreSQL.

A.11 - Mini exemplos de interpretacao completa

Saida observada

Interpretacao

Proximo passo

python --version -> Python 3.x.y

O interpretador respondeu e mostra sua versao.

Ainda confira python -m pip --version e se e o interpretador que executara a aplicacao.

pg_isready -> accepting connections

O servidor responde no host/porta verificados.

Teste tambem database, usuario, senha e tabela; a prontidao nao garante credenciais corretas.

Test-NetConnection ... -> TcpTestSucceeded: False

A conexao TCP para aquela porta falhou.

Servico parado, porta/host errados ou bloqueio de rede/firewall sao hipoteses.

node --check app.js -> sem saida

A sintaxe do arquivo parece valida.

Execute o servidor e teste rotas; sintaxe valida nao valida dependencias ou banco.

mvn -v -> mostra Java Home

Maven informa qual Java usa.

Se nao e o JDK esperado, corrija JAVA_HOME/PATH ou use o wrapper/configuracao autorizada.

curl -i ... -> HTTP/1.1 401

A aplicacao respondeu e recusou autenticacao.

Confira login, token, expiracao, header Bearer ou cookie de sessao; nao e falha de porta.

curl -i ... -> HTTP/1.1 500

A aplicacao respondeu, mas ocorreu erro interno.

Leia o terminal/log do backend e identifique a excecao raiz; pode ser SQL/banco.

relation "cliente" does not exist

O PostgreSQL nao encontrou a tabela no schema atual.

Confirme current_database(), current_schema(), \dt, nome singular/plural e execucao do SQL.

EADDRINUSE

A porta que o servidor tenta usar ja tem um processo escutando.

Identifique o PID e decida se encerra o processo ou muda a porta de forma consistente.

APPLICATION FAILED TO START

O Spring Boot nao conseguiu concluir a inicializacao.

Leia Description, Action e Caused by; a primeira causa raiz util vale mais que as ultimas linhas genericas.

A.12 - Regra final: o comando verifica uma camada, nao o sistema inteiro

Versao instalada nao significa servico ativo; servico ativo nao significa base criada; base criada nao significa tabelas/populacao corretas; SQL executado nao significa conexao da aplicacao correta; servidor iniciado nao significa rota funcionando; rota funcionando nao significa autenticacao/permissoes corretas. Sempre avance por camadas e confirme o resultado de cada teste.

Sequencia de diagnostico sugerida: (1) comando encontrado/versao; (2) processo e porta; (3) conexao TCP; (4) login no PostgreSQL; (5) banco/schema/tabelas; (6) compilacao/imports; (7) inicializacao do backend e leitura do log; (8) teste HTTP com curl; (9) regras funcionais do SAEP; (10) ZIP final e conteudo.

SAEP - Guia de infraestrutura e diagnostico | PostgreSQL + Java/Spring Boot + Python/Flask + Node/Express   |   Pagina 
