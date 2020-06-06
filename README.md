# Next Level Week #1

> Repositório usado para salvar projeto desenvolvido na NLW #1

## Comandos

terminal
* npx knex --knexfile knexfile.ts migrate:latest
Para o knex rodar as migrations

* knex --knexfile knexfile.ts seed:run
Para rodar o seed

* ts-node-dev --transpileOnly --ignore-watch node_modules src/server.ts
(atualziação de comando: agora ignora os modulos do nodejs que não iremos manipular, só consumir)



VSCode
* ctrl + shift + p, open keybord shortcuts)
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

Install
* Insomnia
* NodeJS
* ReactJS
* React Native
* Typescript
* Knex
* Sqlite

## Anotações

server.ts
```ts
import express from 'express';

const app = express();

app.use(express.json()); //Usado para o express entender o formato JSON do corpo da requisição

//Rota: Endereço completo da requisição
//Recurso: Qual entidade estamos acessando do sistema

//Request param: parâmetro que vem da própria rota e identifica um recurso
//Query param: parâmetros que vem da própria geralmente opcionais para filtros, paginação, etc
const users = [  
    "Guilherme",
    "Amariles",
    "Niara",
    "Amor"
]

app.get("/users", (request,response)=>{
    const search = String(request.query.search); //Forçado a ser String pra evitar vir um array

    const filteredUsers = search ? users.filter(user => user.includes(search)) : users;

    //Use return, pois impede do código prosseguir pra baixo do responde 
    return response.json(filteredUsers);
});

app.get("/users/:id", (request,response)=>{
    const id = Number(request.params.id); //Cast pra Number (2) porq vem como string ("2") 

    const user = users[id];

    return response.json(user);
});

app.post("/users", (request,response)=>{
    const data  = request.body; 

    console.log(data);

    const user = {
        name: data.name,
        email: data.email
    }

    return response.json(user);
});


app.listen(3333);
```

routes.ts
```ts

import express from 'express';

const routes = express.Router();

routes.get("/", (request,response)=>{
    return response.json({message: "Hello World!"});
});

export default routes;  //Exportar as rotas para serem importadas no outro arquivo
```

connection.ts
```ts
import knex from 'knex'; 
import path from 'path'; //Usado para que o Node entenda caminhos (path)

const connection = knex({
    client: 'sqlite3',
    connection: {
        filename: path.resolve(__dirname, 'database.sqlite'), /*__dirname é uma variavel global que retorna 
        o caminho da pasta que contem o arquivo que o chama. Neste caso, retorna database. O retorno é baseado
        no S.O. então não se deve pasar o path diretamente.*/
    }
});

export default connection;
```

migrations/00_create_point.ts
```ts
import Knex from 'knex'
/* Esta com K maiusculo porq quando se quer referir ao tipo da variavle knex
 e não a variavel em si (que neste caso é a mesma coisa), bota-se em letra maiuscula. 
 Os tipos do typescript geralmente começam com letras maiuscula, os tipos que não são 
 primitivos da linguagem, como String ou outros tipos de bibliotecas. 
*/

//MIGRATION = Histórico do banco de dados

export async function up(knex: Knex){
    //CRIA A TABELA
    return knex.schema.createTable('points', table => {
        table.increments('id').primary();
        table.string('mage').notNullable;
        table.string('name').notNullable;
        table.string('email').notNullable;
        table.string('whatsapp').notNullable;
        table.decimal('latitude').notNullable;
        table.decimal('longitude').notNullable
        table.string('city').notNullable;
        table.string('uf',2).notNullable;
    })
}

export function down(knex: Knex){
    //DELETA A TABELA
    return knex.schema.dropTable('points');
}
```

knexfile.ts
```ts
import path from 'path'

module.exports = { /* Usa modelu.exports porq esse arquivo, mesmo sendo arquivo ts, não permite 
    usar a sintaxe import default pois o knex, infelizmente, não suporta essa sintaxe na hora
    de ler esse arquivo via terminal. */
    client: 'sqlite3',
    connection: {
        filename: path.resolve(__dirname, 'src', 'database', 'database.sqlite') 
        // o caminho seria algo do tipo: server/src/database/database.sqlite (a depender do S.O.)
    },
    migrations: { //É o nome da pasta que foi criada para os migrations
        directory: path.resolve(__dirname, 'src', 'database', 'migrations')
    },
    useNullAsDefault: true, //Usado para remover mensagem de que o sqlite não suporta valores padrão.
};
```

(atualização no server.ts)
```ts
app.use('/uploads', express.static(path.resolve(__dirname, '..', 'uploads'))) //Função do express.static() usada pra servir arquivos estáticos (imagens, pdf, docuemntos pra download, etc)
```

(atualização no routes.ts)
```ts
import express from 'express';
import knex from './database/connection';

const routes = express.Router();

routes.get("/items", async (request,response)=>{ //Async porq mais abaixo usa-se um wait

    const items = await knex('items').select('*'); //Wait porq é um retorno que deve ser esperado sua concluir pra avançar

    const serializedItems = items.map(item =>{
        return {
            title: item.title,
            imagem_url: `http://localhost:3333/uploads/${item.image}`,
        };
    }) //Map percorre todo os itens retornados do BD e modifica-os conforme necessidade.
    /*Serialized porq as informações do banco não estão exatamente da maneira que se precisa retornar 
    pro front-end/cliente. Quando ocorre o processo de transformar essas informações, esse processo 
    tem o nome de serialização. Serialização de dados. Transformando os dados para ficar mais acessível 
    a quem está requisitando.*/
    return response.json(serializedItems);
});

routes.get("/points", async (request,response)=>{ 

    const { //É o mesmo qeu fazer const name = request.body.name; const email = request.body.email; ... ; 
        name,
        email,
        whatsapp,
        latitude,
        longitude,
        city,
        uf,
        itens
    } = request.body;

    await knex('points').insert({ /* Short Sintax: quando o nome da variável é igual ao nome da propriedade 
        do objeto, é possível omitir e usar desta forma. */
        image: 'image-fake',
        name,
        email,
        whatsapp,
        latitude,
        longitude,
        city,
        uf,
        itens
    })

    return response.json({success: true});
});

export default routes;
```





















































































