# 🚀 Whaticket LM2Rodas

Documentação técnica oficial para execução em produção com Docker
Agradecimentos à **LM 2 Rodas** pelo desafio proposto.

## 📌 Sobre o Projeto

Este projeto é uma versão **totalmente dockerizada** do Whaticket, adaptada para funcionar sem erros em ambientes Linux modernos (Ubuntu/Debian).

Embora a solução seja baseada em *whaticket*, que utiliza o framework **whatsapp-web.js** (em vez do Baileys), ela apresenta **todas as funcionalidades necessárias para a conclusão do desafio** e está pronta para produção com containers isolados, proxy reverso e otimizações de build.

---

# 🛠️ 1. Resumo das Alterações Técnicas

## (Changelog de Infraestrutura)

O objetivo das correções foi permitir que o Whaticket rodasse de forma estável no Linux utilizando Docker, corrigindo falhas relacionadas a builds, dependências e comunicação entre containers.

---

## 🔧 **Backend (Node.js)**

✔ **Imagem base atualizada para `node:14-bullseye`**
A versão antiga usava repositórios Debian expirados, gerando erros durante o `apt-get update`.
A imagem *bullseye* elimina esse problema.

✔ **Instalação manual de dependências Linux para rodar o Puppeteer/Chrome:**
Foram incluídas libs essenciais como:

* `libgbm-dev`
* `libdrm2`
* `libxshmfence1`
* `libnss3`
* `libatk-bridge2.0-0`
* `libgtk-3-0`
* entre outras necessárias para gerar o QR Code sem travamentos.

✔ **Alteração no comando de inicialização:**
Antes (dev):

```
nodemon server.js
```

Agora (prod):

```
node dist/server.js
```

Isso evita reinicializações desnecessárias e garante performance em ambiente produtivo.

---

## 🎨 **Frontend (React + Nginx)**

✔ **Configuração de Nginx embutida diretamente no Dockerfile**
Isso impede erros de *file not found* e garante que o container sempre tenha a configuração correta.

✔ **Proxy Reverso configurado corretamente**
Regras adicionadas para redirecionar corretamente para o backend:

* `/socket.io`
* `/auth`
* `/tickets`
* `/campaigns`
* demais rotas da API

Isso elimina erros:
⚠ 405 Method Not Allowed
⚠ Tela branca no login
⚠ Falha ao conectar WebSocket

✔ **Remoção da `default.conf` nativa do Nginx**
Evita conflito com erro clássico:

```
duplicate listen 80
```

---

## 🐳 **Docker Compose**

✔ **`restart: always` em todos os serviços**
Melhora resiliência em caso de queda do servidor.

✔ **`depends_on` no backend:**

```
depends_on:
  - mysql
```

Garante que o Node só tente conectar após o MySQL estar acessível.

---

---

# 📦 2. Guia de Deploy em Produção (Linux)

A seguir, o tutorial oficial para subir o Whaticket Dockerizado em um ambiente Ubuntu/Debian limpo.

---

## 🔹 **1. Instalar Docker + Docker Compose**

### 🔧 Atualizar o servidor

```bash
sudo apt update && sudo apt upgrade -y
```

### 🐳 Instalar Docker

```bash
curl -fsSL https://get.docker.com | bash
```

### 🔧 Instalar Docker Compose Plugin

```bash
sudo apt install docker-compose-plugin -y
```

### Verificar instalação

```bash
docker --version
docker compose version
```

---

## 🔹 **2. Clonar o Projeto e Configurar `.env`**

### 📥 Clonar o repositório

```bash
git clone https://github.com/seu-repo/whaticket-dockerizado.git
cd whaticket-dockerizado
```

### 📝 Criar o arquivo `.env`

Copie o `.env.example` ou crie manualmente.

### ⚠ IMPORTANTE

No servidor **NÃO USE `localhost`**.
Sempre use o **IP público** do servidor:

Exemplo:

```
BACKEND_URL=http://SEU_IP:8080
FRONTEND_URL=http://SEU_IP
```

Caso utilize domínio, substitua pelo domínio:

```
BACKEND_URL=https://api.seudominio.com
FRONTEND_URL=https://painel.seudominio.com
```

---

## 🔹 **3. Subir o Projeto**

### 📦 Build + Deploy

```bash
docker compose up -d --build
```

O sistema criará automaticamente:

* Container MySQL
* Container Backend (Node)
* Container Frontend (Nginx/React)
* Configurações de proxy reverso
* Build de produção otimizado

---

## 🔹 **4. Comandos Úteis**

### 📜 Ver logs

Backend:

```bash
docker compose logs -f backend
```

Frontend:

```bash
docker compose logs -f frontend
```

MySQL:

```bash
docker compose logs -f mysql
```

### 🔄 Reiniciar todos os containers

```bash
docker compose restart
```

### 🧹 Rebuild completo

```bash
docker compose down
docker compose up -d --build
```

---

# ⚙️ Key Features

* Execução 100% em Docker
* Nginx com proxy reverso configurado
* Backend otimizado para Puppeteer
* Conexão MySQL estável
* Build de produção independente
* Compatível com qualquer VPS Linux

---

# 📄 License

Este projeto segue a licença original do Whaticket.

---

# 🙏 Acknowledgments

Obrigado à LM 2 Rodas pela confiança e pela oportunidade de aprimorar meus conhecimentos.

---
