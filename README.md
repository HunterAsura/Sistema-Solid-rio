# 🌱 Sistema Solidário


Plataforma colaborativa para conectar doadores, beneficiários e organizações sociais. O objetivo é fortalecer redes de apoio locais, simplificando o processo de doação e distribuição de recursos.


## 🚀 Funcionalidades


- Cadastro e autenticação de usuários (doadores, beneficiários, ONGs)
- Registro e gerenciamento de itens para doação
- Solicitação de ajuda por beneficiários
- Registro de doações e acompanhamento de status
- Relatórios e estatísticas sobre impacto social


## 🧱 Estrutura do Projeto

## 🗃️ Banco de Dados
PostgreSQL com tabelas: `usuarios`, `itens`, `solicitacoes`, `doacoes`.


## 🔧 Tecnologias
- Node.js + Express
- Knex.js (migrations e queries)
- MySQL
- JWT (autenticação)

## 2. schema.sql — Banco de Dados Completo (MySQL)


```sql
-- Criação do Banco de Dados
CREATE DATABASE sistema_solidario CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE sistema_solidario;


-- Tabela de Usuários
CREATE TABLE usuarios (
id INT AUTO_INCREMENT PRIMARY KEY,
nome VARCHAR(200) NOT NULL,
email VARCHAR(150) UNIQUE NOT NULL,
senha_hash VARCHAR(255) NOT NULL,
tipo ENUM('ADMIN','DOADOR','ONG','BENEFICIARIO') NOT NULL,
telefone VARCHAR(30),
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB;


-- Tabela de Itens
CREATE TABLE itens (
id INT AUTO_INCREMENT PRIMARY KEY,
doador_id INT,
nome VARCHAR(200) NOT NULL,
descricao TEXT,
quantidade INT NOT NULL DEFAULT 1,
status ENUM('DISPONIVEL','RESERVADO','ENTREGUE','INATIVO') DEFAULT 'DISPONIVEL',
validade DATE,
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
FOREIGN KEY (doador_id) REFERENCES usuarios(id) ON DELETE SET NULL
) ENGINE=InnoDB;


-- Tabela de Solicitações
CREATE TABLE solicitacoes (
id INT AUTO_INCREMENT PRIMARY KEY,
beneficiario_id INT,
descricao TEXT NOT NULL,
status ENUM('PENDENTE','EM_PROCESSAMENTO','ATENDIDA','CANCELADA') DEFAULT 'PENDENTE',
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
FOREIGN KEY (beneficiario_id) REFERENCES usuarios(id) ON DELETE SET NULL
) ENGINE=InnoDB;


-- Tabela de Doações
CREATE TABLE doacoes (
id INT AUTO_INCREMENT PRIMARY KEY,
item_id INT,
solicitacao_id INT,
quantidade INT NOT NULL CHECK (quantidade > 0),
data_doacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (item_id) REFERENCES itens(id) ON DELETE CASCADE,
FOREIGN KEY (solicitacao_id) REFERENCES solicitacoes(id) ON DELETE CASCADE
) ENGINE=InnoDB;


-- Índices
CREATE INDEX idx_itens_status ON itens(status);
CREATE INDEX idx_solicitacoes_status ON solicitacoes(status);

## 3.package.json
{
"name": "sistema-solidario-api",
"version": "1.0.0",
"main": "src/app.js",
"scripts": {
"dev": "nodemon src/app.js"
},
"dependencies": {
"express": "^4.18.2",
"knex": "^3.1.0",
"mysql2": "^3.9.0",
"bcrypt": "^5.1.0",
"jsonwebtoken": "^9.0.0"
},
"devDependencies": {
"nodemon": "^3.0.2"
}
}

## 🧠 src/db.js
import knex from 'knex';


export const db = knex({
client: 'mysql2',
connection: {
host: 'localhost',
user: 'root',
password: 'senha',
database: 'sistema_solidario'
}
});


## ⚙️ src/app.js
import express from 'express';
import { router as usuariosRouter } from './routes/usuarios.js';
import { router as itensRouter } from './routes/itens.js';
import { router as solicitacoesRouter } from './routes/solicitacoes.js';


const app = express();
app.use(express.json());


app.use('/api/usuarios', usuariosRouter);
app.use('/api/itens', itensRouter);
app.use('/api/solicitacoes', solicitacoesRouter);


app.listen(3000, () => console.log('Servidor rodando na porta 3000 🚀'));

## 🧾 src/routes/usuarios.js

import express from 'express';
import bcrypt from 'bcrypt';
import { db } from '../db.js';
export const router = express.Router();


// CREATE
router.post('/', async (req, res) => {
const { nome, email, senha, tipo, telefone } = req.body;
const senha_hash = await bcrypt.hash(senha, 10);
const [id] = await db('usuarios').insert({ nome, email, senha_hash, tipo, telefone });
const user = await db('usuarios').where({ id }).first();
res.status(201).json(user);
});


// READ
router.get('/', async (req, res) => {
const users = await db('usuarios').select('*');
res.json(users);
});

## 🎁 src/routes/itens.js

import express from 'express';
import { db } from '../db.js';
export const router = express.Router();


// CREATE
router.post('/', async (req, res) => {
const { doador_id, nome, descricao, quantidade } = req.body;
const [id] = await db('itens').insert({ doador_id, nome, descricao, quantidade });
const item = await db('itens').where({ id }).first();
res.status(201).json(item);
});


// READ
router.get('/', async (req, res) => {
const itens = await db('itens').select('*');
res.json(itens);
});


// UPDATE
router.put('/:id', async (req, res) => {
const { id } = req.params;
const { nome, descricao, status, quantidade } = req.body;
await db('itens').where({ id }).update({ nome, descricao, status, quantidade });
const updated = await db('itens').where({ id }).first();
res.json(updated);
});


// DELETE
router.delete('/:id', async (req, res) => {
const { id } = req.params;
await db('itens').where({ id }).del();
res.status(204).send();
});

## 💬 src/routes/solicitacoes.js

import express from 'express';
import { db } from '../db.js';
export const router = express.Router();


// CREATE
router.post('/', async (req, res) => {
const { beneficiario_id, descricao } = req.body;
const [id] = await db('solicitacoes').insert({ beneficiario_id, descricao });
const sol = await db('solicitacoes').where({ id }).first();
res.status(201).json(sol);
});


// READ
router.get('/', async (req, res) => {
const solicitacoes = await db('solicitacoes').select('*');
res.json(solicitacoes);
});

## ▶️ Execução
```bash
git clone https://github.com/HunterAsura/sistema-solidario.git
cd sistema-solidario/api
npm install
npm run dev
