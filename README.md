# Mecanica-Presion---Database-Design-MySQL-
Projeto de modelagem e implementação de banco de dados relacional em MySQL para uma oficina mecânica.


# SQL — Database Design: Mecânica Precision

Referência técnica de **modelagem e implementação de banco de dados relacional** em MySQL, construída a partir de um cenário real de oficina mecânica em expansão.

---

## Project Overview — Business Context

A Mecânica Precision realizava o controle de clientes, veículos e serviços de forma manual, o que resultava em perda de informações técnicas, baixa rastreabilidade e falhas no controle financeiro.

Este projeto propõe a construção de um **banco de dados relacional estruturado**, capaz de organizar o fluxo operacional da oficina e servir como base para sistemas de gestão.

---

## Project Goal — Technical Objective

Desenvolver o esquema de banco de dados **BD Oficina**, permitindo o rastreio completo do serviço:

O modelo garante integridade referencial, separação de responsabilidades e controle financeiro detalhado.

---

## Business Rules — Core Constraints

|Regra de Negócio|Implementação no Modelo|
|---|---|
|Cliente pode possuir vários veículos|Relacionamento 1:N (Cliente → Veículo)|
|Veículo pertence a um único cliente|FK obrigatória em `Veiculo`|
|Cada serviço possui um mecânico responsável|FK em `Ordem_de_Servico`|
|Registro técnico obrigatório|Tabela `Laudo`|
|Custos separados|Campos distintos para peças e mão de obra|
|Fluxo controlado por status|ENUM `processo`|

---

## Workflow Control — Ordem de Serviço

A tabela `Ordem_de_Servico` representa o **núcleo operacional do sistema**.

Estados possíveis do processo:

> **Key Insight:** O uso de `ENUM` impede estados inválidos e mantém consistência no fluxo de trabalho.

---

## Database Environment — Technical Requirements

- **SGBD:** MySQL 8.0  
- **Ferramenta:** MySQL Workbench  
- **SO:** Windows 11  
- **Charset:** `utf8`  
- **Collation:** `utf8_general_ci`  
- **Engine:** InnoDB  


---

## Physical Model — Database Creation

<img width="580" height="366" alt="image" src="https://github.com/user-attachments/assets/c5b03ec4-3314-4553-a224-0e5143ee38f4" />
<img width="445" height="426" alt="image" src="https://github.com/user-attachments/assets/17802f78-d05c-4db2-bdf9-7abfb35ef3e5" />


```sql
CREATE DATABASE IF NOT EXISTS mecanica_precision
CHARACTER SET utf8
COLLATE utf8_general_ci;

USE mecanica_precision;


CREATE TABLE Cliente (
    ID_Cliente INT AUTO_INCREMENT PRIMARY KEY,
    Nome_Completo VARCHAR(100) NOT NULL,
    CPF CHAR(11) NOT NULL UNIQUE,
    Email VARCHAR(100) NOT NULL,
    Telefone VARCHAR(20) NOT NULL,
    Data_Cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;


CREATE TABLE Veiculo (
    ID_Veiculo INT AUTO_INCREMENT PRIMARY KEY,
    Placa CHAR(7) NOT NULL UNIQUE,
    Marca VARCHAR(30) NOT NULL,
    Modelo VARCHAR(50) NOT NULL,
    Ano INT,
    ID_Cliente INT NOT NULL,
    FOREIGN KEY (ID_Cliente)
        REFERENCES Cliente(ID_Cliente)
) ENGINE=InnoDB;


CREATE TABLE Mecanico (
    ID_Mecanico INT AUTO_INCREMENT PRIMARY KEY,
    Nome VARCHAR(100) NOT NULL,
    Especialidade VARCHAR(50) NOT NULL
) ENGINE=InnoDB;


CREATE TABLE Ordem_de_Servico (
    ID_Ordem INT AUTO_INCREMENT PRIMARY KEY,
    ID_Veiculo INT NOT NULL,
    ID_Mecanico INT NOT NULL,
    processo ENUM(
        'Agendado',
        'Em_analise',
        'Aguardando_pecas',
        'Finalizado',
        'Cancelado'
    ) NOT NULL DEFAULT 'Agendado',
    Valor_Pecas DECIMAL(10,2) DEFAULT 0.00,
    Valor_Mao_Obra DECIMAL(10,2) DEFAULT 0.00 NOT NULL,
    Data_Abertura TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    Data_Fechamento TIMESTAMP NULL,
    FOREIGN KEY (ID_Veiculo) REFERENCES Veiculo(ID_Veiculo),
    FOREIGN KEY (ID_Mecanico) REFERENCES Mecanico(ID_Mecanico)
) ENGINE=InnoDB;


CREATE TABLE Laudo (
    ID_Laudo INT AUTO_INCREMENT PRIMARY KEY,
    ID_Ordem INT NOT NULL,
    Diagnostico MEDIUMTEXT NOT NULL,
    Procedimento MEDIUMTEXT NOT NULL,
    Data_Registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ID_Ordem)
        REFERENCES Ordem_de_Servico(ID_Ordem)
        ON DELETE CASCADE
) ENGINE=InnoDB;



Data Validation — Test Load (DML)
Para validação do modelo, foi realizada uma carga de testes contendo:
30 clientes
30 veículos
30 ordens de serviço
4 mecânicos
A carga valida integridade referencial, fluxo de status e comportamento do modelo em cenário real.
Glossary — Technical Terms
ENUM: Tipo de dado controlado para padronização de estados
Referential Integrity: Garantia de consistência entre tabelas
Operational Core: Entidade central do fluxo de dados
DDL / DML: Linguagens de definição e manipulação de dados
