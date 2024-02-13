# estrutura-typescript-sequelize

### Inicie o npm com 'npm init -y'. Instale as dependencias necessarias:
```
npm i -D @types/express @types/node @types/sequelize mysql2 nodemon sequelize-cli ts-node-dev
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
    "esModuleInterop": true             /* Permite importar módulos do tipo CommonJS em código do TypeScript */
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
