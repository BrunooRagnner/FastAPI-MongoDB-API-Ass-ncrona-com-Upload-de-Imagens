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

├── app/
│ ├── init.py
│ ├── main.py # entrypoint do FastAPI
│ ├── database.py # conexão com MongoDB e GridFS
│ ├── config.py # variáveis de ambiente (Pydantic settings)
│ ├── models/ # (opcional) classes para documentos
│ ├── schemas/ # esquemas Pydantic
│ │ ├── init.py
│ │ └── user.py
│ ├── routes/ # endpoints
│ │ ├── init.py
│ │ ├── users.py
│ │ ├── auth.py
│ │ └── images.py
│ └── utils/ # helpers
│ ├── init.py
│ └── auth.py
├── tests/ # testes automatizados
├── .env # variáveis de ambiente
├── .gitignore
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
└── README.md # você está aqui



## ⚙️ Pré-requisitos

- **Python 3.11+** ou **Docker** e **Docker Compose**
- MongoDB (local ou Atlas) – *se não usar Docker*

## 🚀 Configuração e Execução

### 🐳 Com Docker (recomendado)

1. **Clone o repositório**
   ```bash
   git clone https://github.com/seu-usuario/fastapi-mongo.git
   cd fastapi-mongo



   Crie o arquivo .env (veja modelo no final da tabela de variáveis)

Suba os containers

bash
docker-compose up -d
Isso iniciará:

MongoDB na porta 27017

API FastAPI na porta 8000

Acesse http://localhost:8000/docs

💻 Sem Docker (local)
Crie e ative um ambiente virtual

bash
python -m venv venv
source venv/bin/activate      # Linux/Mac
venv\Scripts\activate         # Windows
Instale as dependências

bash
pip install -r requirements.txt
Inicie o MongoDB local (ou use Atlas) e configure o .env

Execute a aplicação

bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
🧩 Endpoints Principais
Método	Rota	Descrição	Autenticação
POST	/users/	Criar usuário	Não
GET	/users/	Listar usuários	Não
GET	/users/{id}	Obter usuário por ID	Não
PATCH	/users/{id}	Atualizar usuário	Não
DELETE	/users/{id}	Deletar usuário	Não
POST	/auth/token	Login → token JWT	Não
GET	/users/me	Perfil do usuário logado	Sim (Bearer)
POST	/images/upload	Upload de imagem (GridFS)	Sim*
GET	/images/{file_id}	Recuperar imagem	Sim*
DELETE	/images/{file_id}	Deletar imagem	Sim*
*Para proteger o upload de imagens, recomenda-se adicionar Depends(get_current_user) nas rotas.

🖼️ Upload de Imagens (GridFS)
Exemplo com cURL
bash
# Upload
curl -X POST http://localhost:8000/images/upload \
  -F "file=@/caminho/para/foto.jpg"

# Resposta
{"file_id":"65a1b2c3d4e5f67890abcdef","filename":"foto.jpg"}

# Download (salva como foto_baixada.jpg)
curl http://localhost:8000/images/65a1b2c3... --output foto_baixada.jpg
Exemplo com fetch (JavaScript)
javascript
const formData = new FormData();
formData.append('file', fileInput.files[0]);

fetch('http://localhost:8000/images/upload', {
  method: 'POST',
  body: formData,
  headers: { 'Authorization': `Bearer ${token}` }
})
.then(res => res.json())
.then(data => console.log(data.file_id));
Código da rota (routes/images.py)
python
from fastapi import APIRouter, UploadFile, File, HTTPException, Depends
from fastapi.responses import Response
from app.database import fs
from app.utils.auth import get_current_user
from bson import ObjectId
import gridfs

router = APIRouter(prefix="/images", tags=["images"])

@router.post("/upload", status_code=201)
async def upload_image(
    file: UploadFile = File(...),
    current_user = Depends(get_current_user)  # protege a rota
):
    if not file.content_type.startswith("image/"):
        raise HTTPException(400, "Arquivo deve ser uma imagem")
    contents = await file.read()
    file_id = fs.upload_from_stream(
        filename=file.filename,
        source=contents,
        metadata={"content_type": file.content_type, "user_id": str(current_user["_id"])}
    )
    return {"file_id": str(file_id), "filename": file.filename}

@router.get("/{file_id}")
async def get_image(file_id: str):
    try:
        grid_out = fs.open_download_stream(ObjectId(file_id))
        contents = await grid_out.read()
        content_type = grid_out.metadata.get("content_type", "image/jpeg")
        return Response(content=contents, media_type=content_type)
    except gridfs.errors.NoFile:
        raise HTTPException(404, "Imagem não encontrada")
🔐 Autenticação JWT
Obter token
http
POST /auth/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=usuario@email.com&password=123456
Resposta:

json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer"
}
Usar token em requisições protegidas
http
GET /users/me HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Dependência de autenticação (utils/auth.py - resumo)
python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import jwt, JWTError

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(status_code=401, detail="Credenciais inválidas")
    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=[settings.algorithm])
        user_id = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    user = await user_collection.find_one({"_id": ObjectId(user_id)})
    if user is None:
        raise credentials_exception
    return user
🧪 Testes
bash
# Instalar dependências de teste
pip install pytest httpx pytest-asyncio

# Executar todos os testes
pytest -v --asyncio-mode=auto
Exemplo de teste (tests/test_users.py):

python
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_create_user():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.post("/users/", json={
            "email": "teste@exemplo.com",
            "full_name": "Teste Silva",
            "password": "123456"
        })
    assert response.status_code == 201
    assert response.json()["email"] == "teste@exemplo.com"
🔧 Variáveis de Ambiente
Crie um arquivo .env na raiz com o seguinte conteúdo:

ini
# .env
MONGODB_URL=mongodb://localhost:27017
DB_NAME=fastapi_db
SECRET_KEY=your_super_secret_key_here_change_in_production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
⚠️ Nunca versione o .env – adicione ao .gitignore.

🐳 Docker Compose completo
yaml
# docker-compose.yml
version: '3.8'
services:
  mongodb:
    image: mongo:6
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
    volumes:
      - mongo_data:/data/db
    restart: unless-stopped

  api:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - mongodb
    environment:
      MONGODB_URL: mongodb://admin:secret@mongodb:27017
      DB_NAME: fastapi_db
      SECRET_KEY: ${SECRET_KEY}
      ALGORITHM: ${ALGORITHM}
      ACCESS_TOKEN_EXPIRE_MINUTES: ${ACCESS_TOKEN_EXPIRE_MINUTES}
    restart: unless-stopped

volumes:
  mongo_data:
📦 Dockerfile
dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./app ./app

ENV PYTHONPATH=/app

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
📝 Comandos Úteis
MongoDB (Shell)
sql
-- Conectar
mongosh "mongodb://admin:secret@localhost:27017"

-- Ver bancos
show dbs

-- Usar banco
use fastapi_db

-- Listar coleções
show collections

-- Consultar usuários
db.users.find().pretty()

-- Criar índice único para email
db.users.createIndex({ "email": 1 }, { unique: true })
Utilitários
bash
# Ver logs dos containers
docker-compose logs -f api

# Reconstruir e rodar
docker-compose up -d --build

# Parar e remover volumes (cuidado!)
docker-compose down -v
📚 Documentação Interativa
Após executar, acesse:

Swagger UI: http://localhost:8000/docs

ReDoc: http://localhost:8000/redoc

🤝 Contribuição
Fork o projeto

Crie sua branch (git checkout -b feature/nova-feature)

Commit suas mudanças (git commit -m 'feat: add nova feature')

Push para a branch (git push origin feature/nova-feature)

Abra um Pull Request

📄 Licença
Este projeto está sob a licença MIT. Consulte o arquivo LICENSE para mais informações.

Desenvolvido com ❤️ usando FastAPI e MongoDB

text

---

## ✅ Como usar no GitHub

1. **Crie um repositório** no GitHub.
2. **Na raiz do seu projeto**, crie um arquivo chamado `README.md`.
3. **Copie todo o conteúdo acima** (do bloco de código Markdown) e cole no `README.md`.
4. **Faça commit e push**:
   ```bash
   git add README.md
   git commit -m "docs: adiciona README completo"
   git push origin main

 que você criou. Copie a URL do repositório, que será algo como:

text
https://github.com/seu-usuario/fastapi-mongo-api.git
Agora, no terminal, adicione essa URL como “origin”:

bash
git remote add origin https://github.com/seu-usuario/fastapi-mongo-api.git
6️⃣ Enviar (push) para o GitHub
bash
git branch -M main          # garante que a branch se chama main
git push -u origin main     # envia tudo para o GitHub
O Git pedirá seu usuário e senha do GitHub.
Se você usa autenticação em dois fatores ou token, use um Personal Access Token em vez da senha.
Como criar um token no GitHub

✅ Pronto!
Acesse https://github.com/seu-usuario/fastapi-mongo-api e você verá todo o código, o README.md já renderizado com os blocos de código, tabelas, emojis e tudo mais.

🔁 Atualizando o repositório depois de alterações
Sempre que você modificar algo:

bash
git add .
git commit -m "descrição das alterações"
git push
📌 Dica extra: clonar o repositório em outro computador
bash
git clone https://github.com/seu-usuario/fastapi-mongo-api.git
cd fastapi-mongo-api


