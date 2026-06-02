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
Agora o GitHub vai renderizar automaticamente o README com todos os blocos de código, tabelas, emojis e formatação correta.

Caso queira incluir também os arquivos do projeto (código fonte), é só adicioná-los ao repositório. O README já serve como documentação oficial.

Agora você tem um repositório profissional para estudar offline (basta clonar) e compartilhar! 🎉

como coloco no git
We need to answer "como coloco no git" - the user wants to know how to put the README and project on Git (GitHub). Provide step-by-step instructions for initializing a Git repository, adding files, committing, and pushing to GitHub. Also include creating a remote repository on GitHub. Keep it clear, concise, and in Portuguese.

Claro! Vou te ensinar passo a passo como colocar todo o projeto (incluindo o README.md) no GitHub.

📦 Como enviar seu projeto FastAPI + MongoDB para o GitHub
Pré‑requisitos
Ter o Git instalado no seu computador (baixar aqui)

Ter uma conta no GitHub

1️⃣ Criar um repositório novo no GitHub
Faça login no GitHub.

Clique no botão “New” (ou acesse https://github.com/new).

Escolha um nome, ex: fastapi-mongo-api.

Deixe como público ou privado – à sua escolha.

Não marque as opções “Add a README”, “.gitignore” ou “license” (vamos fazer tudo localmente).

Clique em “Create repository”.

Após criar, o GitHub mostrará uma tela com instruções. Vamos usar o caminho “…or create a new repository on the command line”.

2️⃣ Preparar o projeto local
No terminal, navegue até a pasta do seu projeto.

bash
cd caminho/para/seu/projeto/fastapi-mongo
Agora, inicialize o Git dentro dela:

bash
git init
Isso cria uma pasta oculta .git – seu repositório local está pronto.

3️⃣ Adicionar os arquivos
Se você ainda não criou o README.md, copie o conteúdo que eu forneci e salve como README.md na raiz do projeto.

Agora adicione todos os arquivos ao “stage” do Git:

bash
git add .
Isso inclui README.md, app/, docker-compose.yml, Dockerfile, .env (cuidado: não versionar o .env – veja dica abaixo) e todos os outros.

⚠️ Importante: antes de adicionar, crie um arquivo .gitignore na raiz com o seguinte conteúdo:

text
.env
__pycache__/
*.pyc
venv/
.vscode/
.idea/
Depois rode git add .gitignore e git add . novamente.

4️⃣ Fazer o primeiro commit
bash
git commit -m "Primeiro commit: API FastAPI com MongoDB e upload de imagens"
5️⃣ Conectar ao repositório remoto (GitHub)
Volte à página do repositório que você criou. Copie a URL do repositório, que será algo como:

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
Agora você tem seu projeto salvo no GitHub e pode estudar offline (basta clonar ou baixar o ZIP), compartilhar com outras pessoas e manter o histórico de versões. 🚀

tem como ser um dowload
We need to interpret the user's last message: "tem como ser um dowload" (likely "download" with typo). They want a download of the content. The assistant already explained it can't provide direct PDF download but gave instructions. The user might be asking for a downloadable file (like the README.md or the entire project). Since the assistant cannot generate actual files, it can suggest that the user copy the content and save as file, or use a GitHub repository link. But the assistant could simulate providing a downloadable link by offering a pastebin or raw text? No, can't. Best to clarify again but also offer a direct solution: provide a single block of text that the user can save as a .md file, then they have the file locally. Or remind that the README content can be copied and saved.

Given the conversation, the user wants a download of the e-book content in PDF or the project. The assistant previously gave a long README.md. Perhaps the user wants that as a downloadable file. The assistant can say: "I cannot send a file, but you can copy the README.md content I provided, paste into a text editor, save as 'README.md', and that file is yours. Then you can convert to PDF if needed."

But the user's last message "tem como ser um dowload" is a question: "is it possible to be a download?" So assistant should answer: No, but here's how you can create the download yourself easily.

Also note the user wants "todos os tipos conhecido de bloco de codigo" and a 100-page PDF. The assistant already provided a comprehensive markdown. So answer accordingly.

Entendo que você quer fazer o download direto do conteúdo (o e-book + o README). Infelizmente, não posso enviar arquivos (PDF, ZIP, etc.) porque sou apenas um modelo de texto. Mas posso te dar tudo pronto para você mesmo baixar em 30 segundos – sem precisar instalar nada.

📥 Como obter o arquivo (equivalente a um download)
Opção 1 – Baixar o README.md pronto (recomendado)
Copie todo o conteúdo do README.md que eu postei na mensagem anterior (aquele com badges, tabelas, blocos de código).

Abra o Bloco de Notas (Windows) ou TextEdit (Mac) ou qualquer editor de texto simples.

Cole o conteúdo.

Salve o arquivo com o nome README.md (certifique-se de que a extensão seja .md, não .txt).

Agora você tem o arquivo localmente – pode enviar para o GitHub, converter para PDF ou estudar offline.

Para converter esse README.md em PDF:

Acesse https://www.mdtopdf.com/

Arraste o arquivo README.md para o site ou cole o texto

Clique em "Convert" e faça o download do PDF.

Opção 2 – Baixar o e-book completo (versão de 100+ páginas)
Se você quer o e-book em PDF direto, use o mesmo método:

Copie o e-book completo que eu postei na resposta anterior à do README (aquela com o título "📘 FASTAPI + MONGODB: O GUIA DEFINITIVO").

Cole em um arquivo .md (por exemplo, ebook.md).

Use o conversor online (mesmo link acima) para gerar o PDF.

Opção 3 – Clonar um repositório de exemplo (já com tudo)
Se você tem Git instalado, pode clonar um repositório que eu preparei (você mesmo pode criar). Mas como não posso hospedar arquivos, sugiro:

Crie um repositório vazio no GitHub.

Adicione o README.md (copiado) e os arquivos de código (você pode criar os arquivos conforme o e-book).

Depois clone o repositório para seu computador:

bash
git clone https://github.com/seu-usuario/seu-repo.git
Isso baixa todos os arquivos para sua máquina.

Opção 4 – Usar o GitHub diretamente para "download"
Se você colocar o README.md em um repositório GitHub (público), qualquer pessoa pode baixar o repositório inteiro como ZIP:

Acesse o repositório

Clique no botão verde "Code" → "Download ZIP"

Assim você tem o arquivo Markdown e também os códigos-fontes (se você criá-los).
