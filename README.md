# estrutura-typescript-sequelize

### Inicie o npm com 'npm init -y'. Instale as dependencias necessarias:
```
npm i -D @types/express @types/node @types/sequelize mysql2 nodemon sequelize-cli chai@4.3.6 mocha@9.2.1 sinon@13.0.1 chai-http@4.3.0 sinon-chai@3.7.0 nyc@15.1.0 @types/chai@4.3.0 @types/chai-http@4.2.0 @types/mocha@9.1.0 @types/sinon@10.0.11 @types/sinon-chai@3.2.9 ts-node
npm i dotenv express sequelize typescript
```

### Crie o arquivo tsconfig.json:
```
{
  "compilerOptions": {
    "target": "es6",                    /* Especifica a versão do JavaScript que o código será convertido */
    "module": "commonjs",               /* Define como os módulos serão interpretados */
    "outDir": "./build",
    "rootDir": "./src",               /* Diretório de saída dos arquivos transpilados */
    "strict": true,                     /* Ativa as configurações de modo estrito */
    "esModuleInterop": true,
    "allowImportingTsExtensions": true,
    "declaration": true,
    "emitDeclarationOnly":true         /* Permite importar módulos do tipo CommonJS em código do TypeScript */
  },
  "include": ["src/**/*.ts"],           /* Define quais arquivos TypeScript serão compilados */
  "exclude": ["node_modules"]           /* Define quais pastas não serão incluídas na compilação */
}


```

### Crie o arquivo .sequelizerc:
```
const path = require('path');

module.exports = {
  'config': path.resolve(__dirname, 'build', 'database', 'config', 'database.js'),
  'models-path': path.resolve(__dirname, 'build', 'database', 'models'),
  'seeders-path': path.resolve(__dirname,'build','database','seeders'),
  'migrations-path': path.resolve(__dirname,'build','database','migrations'),
};
```

### Crie a pasta src e faça as pastas com a estrutura do sequelizerc, mas sem o build.

### No arquivo database.ts:
```
import 'dotenv/config';
import { Options } from 'sequelize';

const config: Options = {
  username: process.env.DB_USER || 'root',
  password: process.env.DB_PASS || '',
  database: process.env.DB_NAME || 'database_example', // substitua o nome do banco quando for utilizar em seu projeto!
  host: process.env.DB_HOST || 'localhost',
  port: Number(process.env.DB_PORT) || 3306,
  dialect: 'mysql',
};

export = config;
```

### Crie a migration do seu projeto. Exemplo:
```
import { Model, QueryInterface, DataTypes } from 'sequelize'
import { IUser } from '../../interfaces/IUser'

export default {
  up(queryInterface: QueryInterface) {
    return queryInterface.createTable<Model<IUser>>('users', {
      id: {
        type: DataTypes.INTEGER,
        autoIncrement: true,
        primaryKey: true,
      },
      name: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      email: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      password: {
        type: DataTypes.STRING,
        allowNull: false,
      }
    })
  }
}
```

### Crie o seeder. Exemplo:
```
import { QueryInterface } from 'sequelize';
export default {
  up: async (queryInterface: QueryInterface) => {
    await queryInterface.bulkInsert('users', [
      {
        name: 'Vitor',
        email: 'jv6810@gmail.com',
        password: '123456'
      }
    ], {});
  },
  down: async (queryInterface: QueryInterface) => {
    await queryInterface.bulkDelete('users', {});
  },
};
```
### Crie o index do model:
```
import { Sequelize } from 'sequelize';
import * as config from '../config/database';

export default new Sequelize(config);
```

### Crie o model do referente a sua migration. Exemplo:
```
import { Model, InferAttributes, InferCreationAttributes, DataTypes, CreationOptional } from "sequelize"
import db from '.';

class SequelizeUser extends Model<InferAttributes<SequelizeUser>,
InferCreationAttributes<SequelizeUser>> {
  declare id: CreationOptional<number>;

  declare name: string;

  declare email: string;

  declare password: string;
}

SequelizeUser.init({
  id: {
    type: DataTypes.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false,
  }
}, {
  tableName: 'users',
  modelName: 'users',
  timestamps: false,
  sequelize: db
});

export default SequelizeUser;
```

### Arquivo app.ts e server.ts:
```
import express from 'express';

class App {
  public app: express.Express;

  constructor() {
    this.app = express();

    this.config();

    // Não remover essa rota
    this.app.get('/', (req, res) => res.json({ ok: true }));

  }

  private config():void {
    const accessControl: express.RequestHandler = (_req, res, next) => {
      res.header('Access-Control-Allow-Origin', '*');
      res.header('Access-Control-Allow-Methods', 'GET,POST,DELETE,OPTIONS,PUT,PATCH');
      res.header('Access-Control-Allow-Headers', '*');
      next();
    };

    this.app.use(express.json());
    this.app.use(accessControl);
  }

  public start(PORT: string | number): void {
    this.app.listen(PORT, () => console.log(`Running on port ${PORT}`));
  }
}

export { App };
```

```
import { App } from "./app";

new App().start(3001);
```

### No package.json faça esse script:
```
 "scripts": {
    "db:reset": "npx -y tsc && npx sequelize-cli db:drop && npx sequelize-cli db:create && npx sequelize-cli db:migrate && npx sequelize-cli db:seed:all",
    "dev": "nodemon ./src/server.ts"
  },
```

### Dockerfile e docker-compose.yml:
```
# Usar a imagem node:18-alpine como base
FROM node:18-alpine
# Mudar para o diretório de trabalho /playlist-films
WORKDIR /playlist-films
# Copiar os package.json e package-lock.json para o container
COPY package*.json .
# Instalar as dependências Node
RUN npm ci
# Copiar o restante dos arquivos da aplicação para o container
COPY . .
# Sinalize que aplicação expõe a porta 3001
EXPOSE 3001
# Configurar os comandos para iniciar a aplicação de acordo com as boas práticas
ENTRYPOINT [ "npm", "run", "dev" ]
# Dica: Leia a seção Docker e Docker-compose no README para mais informações
```

```
version: '3.9'
services:
  backend:
    container_name: playlist-api
    build: .
    ports:
      - 3001:3001
     # Caso queira que o container esteja atualizado durante o desenvolvimento, sem que você precise ficar fazendo down e up dos containers, descomente as 3 linhas abaixo
    command: dev 
    volumes: 
      - ./:/playlist-films
    depends_on:
      db:
        condition: service_healthy
    environment:
      DB_USER: ${DB_USER}
      DB_PASS: ${DB_PASS}
      DB_PORT: ${DB_PORT}
      DB_HOST: ${DB_HOST}
      DB_NAME: ${DB_NAME}
    healthcheck:
      test: ["CMD", "lsof", "-t", "-i:3001"] # Caso utilize outra porta interna para o back, altere ela aqui também
      timeout: 10s
      retries: 5
  db:
    image: mysql:8.0.32
    container_name: db
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASS}
    restart: 'always'
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"] # Deve aguardar o banco ficar operacional
      timeout: 10s
      retries: 5
    cap_add:
      - SYS_NICE # Deve omitir alertas menores
```

### Arquivo .env para que funcione o docker-compose:
```
DB_USER=root
DB_PASS=password
DB_PORT=3306
DB_HOST=db
DB_NAME=playlist-films
```

### Entre no container da api com 'docker exec -it nome_do_container sh' se 'sh' não funcionar use 'bash'. Depois use o comando 'npm run db:reset'.







