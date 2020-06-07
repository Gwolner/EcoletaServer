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

* npm install cors 
Instala o CORS que trata de controle de acesso url para a aplicação

* npm install @types/cors -D
Para que o typescript entenda o typo do cors


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
* Knex http://knexjs.org/
* Sqlite

## Anotações de conceitos

CORS = Define na API quais endereços externos vão ter acesso pra aplicação. Define quais URL web vão ter acesso a API.


## Anotações de códigos

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

routes.get("/items", async (request,response)=>{ //Async porq mais abaixo usa-se um wait

    const items = await knex('items').select('*'); //Wait porq é um retorno que deve ser esperado sua concluir pra avançar

    const serializedItems = items.map(item =>{
        return {
            id:item.id,
            title: item.title,
            imagem_url: `http://localhost:3333/uploads/${item.image}`,
        };
    }) 
    return response.json(serializedItems);
});

routes.post("/points", async (request,response)=>{ 

    const { //É o mesmo qeu fazer const name = request.body.name; const email = request.body.email; ... ;
        name,
        email,
        whatsapp,
        latitude,
        longitude,
        city,
        uf,
        items
    } = request.body;

    const trx = await knex.transaction(); /* Onde se tem trx era knex, porém a segunda transação não 
    estava vinculada com a primeira. Em caso de falha, uma ocorreria sem a outra. */

    const insertedIds = await trx('points').insert({ /* insert retorna o id criado. Como e uma única criação, 
        so retorna um id para a constante ids. */
        
        /* Short Sintax: quando o nome da variável é igual ao nome da propriedade 
        do objeto, é possível omitir e usar desta forma. */
        image: "image-fake",
        name,
        email,
        whatsapp,
        latitude,
        longitude,
        city,
        uf
    });

    const point_id = insertedIds[0]; // point_id contem o valor do id criado para point na função anterior.

    const pointItems = items.map((item_id: number) => {
        return {
            item_id,
            point_id,
        };
    });

    await trx('point_items').insert(pointItems);

    return response.json({success: true});
});

export default routes;
```

PointsController.ts
```ts
import knex from '../database/connection';
import {Request, Response} from 'express'; // Lê-se "Importar Request, Response de dentro do express"

class PointController{
    async create(request: Request, response: Response){ 

        const {
            name,
            email,
            whatsapp,
            latitude,
            longitude,
            city,
            uf,
            items
        } = request.body;
    
        const trx = await knex.transaction();
    
        const point = { 
            image: "image-fake",
            name,
            email,
            whatsapp,
            latitude,
            longitude,
            city,
            uf
        }

        const insertedIds = await trx('points').insert(point);
    
        const point_id = insertedIds[0];
    
        const pointItems = items.map((item_id: number) => {
            return {
                item_id,
                point_id,
            };
        });
    
        await trx('point_items').insert(pointItems);
    
        return response.json({
            id: point_id,
            ...point
            //Spread Operator: https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_syntax
        });
    }
}

export default PointController;
```

(atualização)roter.ts
```ts
import express from 'express';
import knex from './database/connection'; //Não é mais utilizado

import PointsController from './controllers/pointsController';
import ItemsController from './controllers/itemsController';

// index, show, create, update, delete

const routes = express.Router();
const pointsController = new PointsController();
const itemsController = new ItemsController();

routes.get("/items", itemsController.index);

routes.post("/points", pointsController.create);


export default routes;

//Dar uma olhada no que é Service pattern e Repositiry pattern.
```

controller/PointsControlle.create
```ts
import {Request, Response} from 'express'; // Lê-se "Importar Request, Response de dentro do express"
import knex from '../database/connection';

class PointsController{
    async create(request: Request, response: Response){
        const {
            name,
            email,
            whatsapp,
            latitude,
            longitude,
            city,
            uf,
            items
        } = request.body;
    
        const trx = await knex.transaction();
    
        const point = {
            image: "image-fake",
            name,
            email,
            whatsapp,
            latitude,
            longitude,
            city,
            uf
        }

        const insertedIds = await trx('points').insert(point);
        console.log("INSERTED ID :"+insertedIds);
        const point_id = insertedIds[0];
    
        const pointItems = items.map((item_id: number) => {
            return {
                item_id,
                point_id,
            };
        });
    
        await trx('point_items').insert(pointItems);

        await trx.commit(); //Necessario pra garantir queocorra as transações!!

        return response.json({
            id: point_id,
            ...point
            //Spread Operator: https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_syntax
        });
    }
}

export default PointsController;
```

controller/PointsControlle.show
```ts
 async show(request: Request, response: Response){
        const { id } = request.params; // É o mesmo que const id = request.params.id;

        const point = await knex('points').where('id', id).first(); 
        //Retorna um unico registro mesmo o ID sendo unico pois sem o first(), poimnt espera um array;

        if(!point){
            return response.status(400).json({ message: "Point not found."});
        }

        const items = await knex('items')
            .join('point_items', 'items.id', '=', 'point_items.item_id')
            .where('point_items.point_id',id)
        /*Equivalente a:
          SELECT * FROM items
            JOIN point_items ON items.id = point_items.item_id
            WHERE point_items.point_id = {id}
          Retorna todos os items relacionados a este point que foi consultado.
        */   .select('items.title');

        return response.json({point, items});

    }
```
controller/PointsControlle.index
```ts
async index(request: Request, response: Response){

        const { city, uf, items } = request.query;

        const parsedItems = String(items)
            .split(",")
            .map(item => Number(item.trim()));
        /*o split separa o que ta entre "," o mapa realiza a ação pra cada 
        item e o trim() apaga os espaços na direita ou esquerda de cada item. */

        const points = await knex('points')
            .join('point_items', 'points.id', '=', 'point_items.point_id')
            .whereIn('point_items.item_id', parsedItems)
            .where('city', String(city)) //Sempre recebe pelo Query é bom informar o formato, neste caso, String! 
            .where('uf', String(uf))
            .distinct() // Evita duplicidade de retorno: [1][2][1,2] etc
            .select('points.*');

        return response.json(points);
    }
```













































































