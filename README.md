# Next Level Week #1

> Repositório usado para salvar projeto desenvolvido na NLW #1

## Comandos

* npm install ts-node -D
Faz com que o Node, que só conhece JS, entenda TS tambem.

* npx
executa um pacote que foi instalado.

* npx ts-node src/server.ts 
inicia o servidor/API

* npm install typescript -D
instala o TS.

* npx tsc --init
Cria arquivo de configuração do TS.

* npm install ts-node-dev -D
Evita ter de dr ctrl+c para parar p serviço a cada mudança de código.
A cada nova atualização, o servidor se reinicia sozinho.

* npm install knex
Usa JS para manipular o BD, permitindo mudar de um BD para outro sem ter de mudar a linguagem utilizada.
Ex: Mudar SQL pra Postgress.

* npm install sqlite3
Instala o arquivo que será usado como BD.

* npx knex --knexfile knexfile.ts migrate:latest
Para o knex rodar as migrations

* knex --knexfile knexfile.ts seed:run
Para rodar o seed

* ts-node-dev --transpileOnly --ignore-watch node_modules src/server.ts
(atualziação de comando: agora ignora os modulos do nodejs que não iremos manipular, só consumir)

* npm install cors 
Instala o CORS que trata de controle de acesso url para a aplicação

* npm install @types/cors -D
Para que o typescript entenda o typo do cors


## Atalho VSCode
* ctrl + shift + p, open keybord shortcuts.
copy line down (copiar linha pra baixo)

## Usados

VSCode plugin
* Dracula
* Omni
* Material icon theme
* SQLite (ctrl + shift + p, >sqlite: open database, database.sqlite)

Web service
* whimsical.com (criar fluxograma)
* notion.so (anotações e controle de informação)

Instalados
* Insomnia
* NodeJS
* ReactJS
* React Native
* Typescript
* Knex http://knexjs.org/
* Sqlite

## Conceitos

CORS = Define na API quais endereços externos vão ter acesso pra aplicação. Define quais URL web vão ter acesso a API.
Live Reload = Atualização automática da aplicação sempre que o código sofre alteração.
