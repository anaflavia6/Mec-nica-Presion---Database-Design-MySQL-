# Mecanica Presion - Database Design (MySQL)
Projeto de modelagem e implementação de banco de dados relacional em MySQL para uma oficina mecânica.

---

## Project Overview  Business Context

A Mecânica Precision realizava o controle de clientes, veículos e serviços de forma manual, o que resultava em perda de informações técnicas, baixa rastreabilidade e falhas no controle financeiro.

Este projeto propõe a construção de um **banco de dados relacional estruturado**, capaz de organizar o fluxo operacional da oficina e servir como base para sistemas de gestão.

---

## Project Goal  Technical Objective

Desenvolver o esquema de banco de dados **BD Oficina**, permitindo o rastreio completo do serviço:

O modelo garante integridade referencial, separação de responsabilidades e controle financeiro detalhado.

---

## Business Rules Core Constraints

|Regra de Negócio|Implementação no Modelo|
|---|---|
|Cliente pode possuir vários veículos|Relacionamento 1:N (Cliente → Veículo)|
|Veículo pertence a um único cliente|FK obrigatória em `Veiculo`|
|Cada serviço possui um mecânico responsável|FK em `Ordem_de_Servico`|
|Registro técnico obrigatório|Tabela `Laudo`|
|Custos separados|Campos distintos para peças e mão de obra|
|Fluxo controlado por status|ENUM `processo`|

---

## Workflow Control  Ordem de Serviço

A tabela `Ordem_de_Servico` representa o **núcleo operacional do sistema**.

Estados possíveis do processo:

> **Key Insight:** O uso de `ENUM` impede estados inválidos e mantém consistência no fluxo de trabalho.

---

## Database Environment  Technical Requirements

- **SGBD:** MySQL 8.0  
- **Ferramenta:** MySQL Workbench  
- **SO:** Windows 11  
- **Charset:** `utf8`  
- **Collation:** `utf8_general_ci`  
- **Engine:** InnoDB  


---

## Physical Model  Database Creation

<img width="580" height="366" alt="image" src="https://github.com/user-attachments/assets/c5b03ec4-3314-4553-a224-0e5143ee38f4" />
<img width="445" height="426" alt="image" src="https://github.com/user-attachments/assets/17802f78-d05c-4db2-bdf9-7abfb35ef3e5" />


```sql
-- 
CREATE DATABASE IF NOT EXISTS mecanica_precision
CHARACTER SET utf8
COLLATE utf8_general_ci;

USE mecanica_precision;

--  CRIAÇÃO DAS TABELAS (DDL)


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
    FOREIGN KEY (ID_Cliente) REFERENCES Cliente(ID_Cliente)
) ENGINE=InnoDB;

CREATE TABLE Mecanico (
    ID_Mecanico INT AUTO_INCREMENT PRIMARY KEY,
    Nome VARCHAR(100) NOT NULL,
    Especialidade VARCHAR(50) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Peca (
    ID_Peca INT AUTO_INCREMENT PRIMARY KEY,
    Nome_Peca VARCHAR(100) NOT NULL,
    Descricao MEDIUMTEXT,
    Preco_Unit DECIMAL(10,2) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Ordem_de_Servico (
    ID_Ordem INT AUTO_INCREMENT PRIMARY KEY,
    ID_Veiculo INT NOT NULL,
    ID_Mecanico INT NOT NULL,
    processo ENUM('Agendado', 'Em_analise', 'Aguardando_pecas', 'Finalizado', 'Cancelado') NOT NULL DEFAULT 'Agendado',
    Valor_Pecas DECIMAL(10,2) DEFAULT 0.00,
    Valor_Mao_Obra DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    Data_Abertura TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    Data_Fechamento TIMESTAMP NULL,
    FOREIGN KEY (ID_Veiculo) REFERENCES Veiculo(ID_Veiculo),
    FOREIGN KEY (ID_Mecanico) REFERENCES Mecanico(ID_Mecanico)
) ENGINE=InnoDB;

CREATE TABLE Ordem_Peca (
    ID_Ordem INT NOT NULL,
    ID_Peca INT NOT NULL,
    Quantidade INT NOT NULL,
    Valor_Total DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (ID_Ordem, ID_Peca),
    FOREIGN KEY (ID_Ordem) REFERENCES Ordem_de_Servico(ID_Ordem),
    FOREIGN KEY (ID_Peca) REFERENCES Peca(ID_Peca)
) ENGINE=InnoDB;

CREATE TABLE Laudo (
    ID_Laudo INT AUTO_INCREMENT PRIMARY KEY,
    ID_Ordem INT NOT NULL,
    Diagnostico MEDIUMTEXT NOT NULL,
    Procedimento MEDIUMTEXT NOT NULL,
    Data_Registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ID_Ordem) REFERENCES Ordem_de_Servico(ID_Ordem) ON DELETE CASCADE
) ENGINE=InnoDB;

-- 3. INSERÇÃO DE DADOS (DML) - CENÁRIOS REAIS
-- Aviso: Dados fictícios para fins de teste.

INSERT INTO Mecanico (Nome, Especialidade) VALUES
('João Silva','Motor'), ('Carlos Lima','Suspensão'),
('Ana Souza','Elétrica'), ('Pedro Rocha','Freios');

INSERT INTO Cliente (Nome_Completo, CPF, Email, Telefone) VALUES
('Rebeca Santana Nogueira', '37941111092', 'rebeca.nogueira@gmail.com', '81992340121'),
('Gabriel Silva Menezes', '11138161184', 'gabriel.menezes@outlook.com', '81993450232'),
('Arthur Henrique Costa', '11268451177', 'arthurhcosta@gmail.com', '81994560343');
-- [Adicione aqui os demais 27 clientes do seu script original]

INSERT INTO Veiculo (Placa, Marca, Modelo, Ano, ID_Cliente) VALUES
('KHP3A21','Fiat','Palio',2012,1),
('PEZ7421','Volkswagen','Gol',2010,2),
('QYA8B44','Chevrolet','Onix',2019,3);
-- [Adicione aqui os demais 27 veículos do seu script original]

INSERT INTO Peca (Nome_Peca, Descricao, Preco_Unit) VALUES
('Óleo Sintético 5W30', 'Lubrificante de alta performance - Galão 1L', 68.90),
('Filtro de Óleo Sedan', 'Filtro de óleo blindado original', 42.00),
('Kit Pastilha de Freio', 'Par de pastilhas dianteiras cerâmica', 185.50),
('Disco de Freio Ventilado', 'Disco de freio dianteiro (unidade)', 240.00),
('Bateria 60Ah', 'Bateria selada livre de manutenção', 480.00);

INSERT INTO Ordem_de_Servico (ID_Veiculo, ID_Mecanico, processo, Valor_Pecas, Valor_Mao_Obra) VALUES
(1, 1, 'Finalizado', 317.60, 150.00),
(2, 2, 'Em_analise', 0.00, 0.00),
(3, 4, 'Finalizado', 665.50, 250.00);

-- Validação N:M: Ligando peças às Ordens de Serviço
INSERT INTO Ordem_Peca (ID_Ordem, ID_Peca, Quantidade, Valor_Total) VALUES
(1, 1, 4, 275.60), -- 4L de Óleo na OS 1
(1, 2, 1, 42.00),   -- 1 Filtro na OS 1
(3, 3, 1, 185.50), -- 1 Kit Pastilha na OS 3
(3, 4, 2, 480.00); -- 2 Discos de Freio na OS 3


SELECT 
    p.Nome_Peca, 
    SUM(op.Quantidade) AS Unidades_Vendidas,
    SUM(op.Valor_Total) AS Receita_Total
FROM Peca p
JOIN Ordem_Peca op ON p.ID_Peca = op.ID_Peca
GROUP BY p.ID_Peca
ORDER BY Receita_Total DESC;


SELECT 
    ID_Ordem, 
    SUM(Quantidade) AS Total_Itens,
    ROUND(SUM(Valor_Total), 2) AS Valor_Total_Pecas
FROM Ordem_Peca
GROUP BY ID_Ordem;


SELECT c.Nome_Completo, v.Modelo, v.Placa
FROM Cliente c
JOIN Veiculo v ON c.ID_Cliente = v.ID_Cliente;

---

Data Validation  Test Load (DML)
Para validação do modelo, foi realizada uma carga de testes contendo:
30 clientes
30 veículos
30 ordens de serviço
4 mecânicos
A carga valida integridade referencial, fluxo de status e comportamento do modelo em cenário real.
Glossary  Technical Terms
ENUM: Tipo de dado controlado para padronização de estados
Referential Integrity: Garantia de consistência entre tabelas
Operational Core: Entidade central do fluxo de dados
DDL / DML: Linguagens de definição e manipulação de dados
