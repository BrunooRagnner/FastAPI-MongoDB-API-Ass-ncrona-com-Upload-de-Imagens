# FastAPI-MongoDB-API-Ass-ncrona-com-Upload-de-Imagens
# 🚀 FastAPI + MongoDB: API Assíncrona com Upload de Imagens

[![FastAPI](https://img.shields.io/badge/FastAPI-0.115.0-009688?logo=fastapi)](https://fastapi.tiangolo.com)
[![MongoDB](https://img.shields.io/badge/MongoDB-6.0-47A248?logo=mongodb)](https://www.mongodb.com)
[![Motor](https://img.shields.io/badge/Motor-3.4.0-00C853)](https://motor.readthedocs.io)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

API RESTful completa usando **FastAPI** (assíncrono), **MongoDB** com **Motor** e **GridFS** para upload de imagens. Inclui autenticação JWT, testes, Docker e boas práticas.

## 📋 Sumário

- [Tecnologias](#tecnologias)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Pré-requisitos](#pré-requisitos)
- [Configuração e Execução](#configuração-e-execução)
  - [Com Docker (recomendado)](#com-docker-recomendado)
  - [Sem Docker (local)](#sem-docker-local)
- [Endpoints Principais](#endpoints-principais)
- [Upload de Imagens (GridFS)](#upload-de-imagens-gridfs)
- [Autenticação JWT](#autenticação-jwt)
- [Testes](#testes)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Comandos Úteis](#comandos-úteis)
- [Documentação Interativa](#documentação-interativa)
- [Contribuição](#contribuição)
- [Licença](#licença)

## 🧰 Tecnologias

| Tecnologia        | Versão  | Finalidade                        |
|-------------------|---------|-----------------------------------|
| Python            | 3.11+   | Linguagem                         |
| FastAPI           | 0.115.0 | Framework web                     |
| Motor             | 3.4.0   | Driver assíncrono MongoDB         |
| GridFS            | embutido| Armazenamento de arquivos         |
| Pydantic          | 2.x     | Validação de dados                |
| python-jose       | 3.3.0   | JWT                               |
| passlib[bcrypt]   | 1.7.4   | Hashing de senhas                 |
| pytest            | 7.4+    | Testes                            |
| Docker / Compose  | latest  | Containerização                   |

## 📁 Estrutura do Projeto

