# gerenciador-catalogos-azure
Projeto DIO Criando um Gerenciador de Catálogos da Netflix com Azure Functions e Banco de Dados

Solução baseada em Azure Functions para gerenciar um catálogo de vídeos estilo Netflix, com Streamlit como frontend e diversos serviços do Azure integrados:

```markdown
# FlixDIO - Catálogo de Vídeos com Azure Functions

Este projeto implementa uma arquitetura serverless para gerenciar um catálogo de vídeos estilo Netflix, utilizando Azure Functions, Cosmos DB, Storage Account, Redis Cache, API Management e Streamlit como frontend.

---

## 🧱 Arquitetura

- **Frontend**: Streamlit
- **Backend**: Azure Functions
- **Banco de dados**: Azure Cosmos DB for NoSQL
- **Armazenamento de mídia**: Azure Storage Account (Blob)
- **Cache**: Azure Cache for Redis
- **API Gateway**: Azure API Management

---

## 🚀 Etapas de Implementação

### 1. Criar Resource Group

```bash
az group create --name FlixDIO --location eastus
```

---

### 2. Provisionar Recursos do Azure

#### Cosmos DB for NoSQL

```bash
az cosmosdb create --name flixdiocosmos --resource-group FlixDIO --kind MongoDB
```

#### Storage Account

```bash
az storage account create --name flixdiostorage --resource-group FlixDIO --location eastus --sku Standard_LRS
```

#### Redis Cache

```bash
az redis create --name flixdiocache --resource-group FlixDIO --location eastus --sku Basic --vm-size C1
```

#### API Management

```bash
az apim create --name flixdioapim --resource-group FlixDIO --location eastus --publisher-email admin@flixdio.com --publisher-name FlixDIO
```

---

### 3. Criar Azure Functions

Crie uma Function App:

```bash
az functionapp create --resource-group FlixDIO --consumption-plan-location eastus --runtime python --functions-version 4 --name flixdiofunctions --storage-account flixdiostorage
```

Implemente as seguintes funções:

| Função                     | Descrição                                                                 |
|---------------------------|---------------------------------------------------------------------------|
| `fnPostDataStorageVideo`  | Recebe e armazena vídeos no Blob Storage                                  |
| `fnPostDataStorageThumbnail` | Recebe e armazena thumbnails no Blob Storage                          |
| `fnPostDataBaseRegistro`  | Registra metadados do vídeo no Cosmos DB                                  |
| `fnGetAllMovies`          | Retorna todos os registros de vídeos do Cosmos DB                         |
| `fnGetMoviesDetail`       | Retorna detalhes de um vídeo específico com cache Redis para escalabilidade |

> 💡 Use o Redis para armazenar os resultados de `fnGetMoviesDetail` e evitar consultas repetidas ao Cosmos DB.

---

### 4. Configurar API Management

- Importe as funções como APIs no serviço de API Management.
- Configure rotas amigáveis como:
  - `POST /videos/upload`
  - `POST /thumbnails/upload`
  - `POST /videos/register`
  - `GET /videos`
  - `GET /videos/{id}`

---

### 5. Desenvolver Frontend com Streamlit

Use Streamlit para criar uma interface de upload, visualização e navegação dos vídeos:

```python
import streamlit as st
import requests

st.title("FlixDIO - Catálogo de Vídeos")

# Exemplo de chamada à API
response = requests.get("https://flixdioapim.azure-api.net/videos")
videos = response.json()

for video in videos:
    st.video(video["url"])
    st.write(video["title"])
```

---

## 🧠 Considerações Técnicas

- Use **Azure Blob Storage SDK** para upload de vídeos e thumbnails.
- Use **Cosmos DB SDK** para registrar e consultar metadados.
- Use **Redis Python Client** (`redis-py`) para cache de detalhes de vídeos.
- Configure **CORS** no API Management para permitir chamadas do Streamlit.

---

## 📦 Requisitos

- Azure CLI
- Python 3.10+
- Streamlit
- Azure SDKs: `azure-functions`, `azure-storage-blob`, `azure-cosmos`, `redis`

---

## 📚 Referências

- [Azure Functions Documentation](https://learn.microsoft.com/en-us/azure/azure-functions/)
- [Streamlit Documentation](https://docs.streamlit.io/)
- [Azure API Management](https://learn.microsoft.com/en-us/azure/api-management/)
- [Azure Cosmos DB](https://learn.microsoft.com/en-us/azure/cosmos-db/)
- [Azure Redis Cache](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/)

---

## 🛠️ Próximos Passos

- Adicionar autenticação com Azure AD B2C
- Implementar busca por título e gênero
- Adicionar suporte a comentários e avaliações

```
