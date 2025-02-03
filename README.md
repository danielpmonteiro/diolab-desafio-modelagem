# diolab-desafio-modelagem
Projeto Conceitual de Banco de Dados – E-COMMERCE

## Descrição
Este projeto representa um esquema de banco de dados para um sistema de venda de produtos. 

Ele foi projetado para suportar operações de clientes, pedidos, fornecedores, produtos, pagamentos e entregas. 

O modelo abrange tanto clientes pessoa física (PF) quanto pessoa jurídica (PJ), garantindo a separação adequada dos dados.

Além disso, há suporte para múltiplos endereços, fornecedores próprios e terceiros, além de múltiplas formas de pagamento.

O esquema foi criado para o banco de dados MySQL

### Criação do Banco de Dados
```sql
DROP DATABASE IF EXISTS ecommerce;
CREATE DATABASE ecommerce;
USE ecommerce;
```

### Estrutura do Banco de Dados
```sql
CREATE TABLE cliente (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    telefone VARCHAR(20),
    tipo ENUM('PF', 'PJ') NOT NULL
);

CREATE TABLE cliente_pf (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT UNIQUE NOT NULL,
    cpf VARCHAR(14) UNIQUE NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES cliente(id) ON DELETE CASCADE
);

CREATE TABLE cliente_pj (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT UNIQUE NOT NULL,
    cnpj VARCHAR(18) UNIQUE NOT NULL,
    razao_social VARCHAR(255) NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES cliente(id) ON DELETE CASCADE
);

CREATE TABLE endereco_cliente (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    logradouro VARCHAR(255) NOT NULL,
    numero VARCHAR(10),
    complemento VARCHAR(255),
    cidade VARCHAR(100) NOT NULL,
    estado CHAR(2) NOT NULL,
    cep VARCHAR(10) NOT NULL,
    principal BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (cliente_id) REFERENCES cliente(id) ON DELETE CASCADE
);

CREATE TABLE fornecedor (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    tipo ENUM('proprio', 'terceiro') NOT NULL
);

CREATE TABLE produto (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    descricao TEXT,
    preco DECIMAL(10,2) NOT NULL
);

CREATE TABLE produto_fornecedor (
    produto_id INT NOT NULL,
    fornecedor_id INT NOT NULL,
    preco DECIMAL(10,2),
    PRIMARY KEY (produto_id, fornecedor_id),
    FOREIGN KEY (produto_id) REFERENCES produto(id) ON DELETE CASCADE,
    FOREIGN KEY (fornecedor_id) REFERENCES fornecedor(id) ON DELETE CASCADE
);

CREATE TABLE pedido (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pendente', 'aprovado', 'cancelado', 'entregue') NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES cliente(id) ON DELETE CASCADE
);

CREATE TABLE pedido_produto (
    id INT AUTO_INCREMENT PRIMARY KEY,
    pedido_id INT NOT NULL,
    produto_id INT NOT NULL,
    fornecedor_id INT NOT NULL,
    quantidade INT NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (pedido_id) REFERENCES pedido(id) ON DELETE CASCADE,
    FOREIGN KEY (produto_id) REFERENCES produto(id) ON DELETE CASCADE,
    FOREIGN KEY (fornecedor_id) REFERENCES fornecedor(id) ON DELETE CASCADE
);

CREATE TABLE estoque (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT NOT NULL,
    quantidade INT NOT NULL,
    FOREIGN KEY (produto_id) REFERENCES produto(id) ON DELETE CASCADE
);

CREATE TABLE pagamento (
    id INT AUTO_INCREMENT PRIMARY KEY,
    pedido_id INT NOT NULL,
    forma_pagamento ENUM('boleto', 'pix', 'cartao') NOT NULL,
    valor DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (pedido_id) REFERENCES pedido(id) ON DELETE CASCADE
);

CREATE TABLE cartao_credito (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    numero_cartao VARBINARY(255) NOT NULL,
    nome_titular VARCHAR(255) NOT NULL,
    validade DATE NOT NULL,
    cvv VARBINARY(255) NOT NULL,
    principal BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (cliente_id) REFERENCES cliente(id) ON DELETE CASCADE
);

CREATE TABLE entrega (
    id INT AUTO_INCREMENT PRIMARY KEY,
    pedido_id INT NOT NULL,
    codigo_rastreio VARCHAR(50),
    status ENUM('em processamento', 'enviado', 'entregue', 'cancelado') NOT NULL,
    FOREIGN KEY (pedido_id) REFERENCES pedido(id) ON DELETE CASCADE
);
```

### Notas sobre Segurança
Os campos `numero_cartao` e `cvv` são armazenados como `VARBINARY(255)`, permitindo criptografia com `AES_ENCRYPT()` para maior segurança. Para criptografar e descriptografar os dados podemos usar:
```sql
-- Inserindo um cartão criptografado
INSERT INTO cartao_credito (cliente_id, numero_cartao, nome_titular, validade, cvv, principal)
VALUES (1, AES_ENCRYPT('4111111111111111', 'chave_secreta'), 'Cliente Teste', '2026-12-01', AES_ENCRYPT('123', 'chave_secreta'), TRUE);

-- Recuperando os dados descriptografados
SELECT id, cliente_id, AES_DECRYPT(numero_cartao, 'chave_secreta') AS numero_cartao, 
       nome_titular, validade, AES_DECRYPT(cvv, 'chave_secreta') AS cvv, principal
FROM cartao_credito;
```
⚠ **Não é aconselhável armazenar números completos de cartão sem criptografia. O ideal é usar provedores de pagamento para evitar armazenamento direto. A estrutura foi criada dessa forma apenas para atender um projeto básico de estudos**
