FROM node:14-bullseye as build-deps

# Instala TODAS as dependências possíveis para o Chrome/Puppeteer no Debian Bullseye
RUN apt-get update && apt-get install -y \
    wget \
    gnupg \
    ca-certificates \
    procps \
    libxss1 \
    libasound2 \
    libatk-bridge2.0-0 \
    libgtk-3-0 \
    libnss3 \
    libx11-xcb1 \
    libxtst6 \
    lsb-release \
    xdg-utils \
    libgbm-dev \
    libdrm2 \
    libxshmfence1 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM node:14-bullseye
WORKDIR /usr/src/app
COPY package*.json ./

# Instalar dependências de produção
RUN npm install --only=production

COPY --from=build-deps /usr/src/app/dist ./dist
COPY --from=build-deps /usr/src/app/public ./public
# COPY --from=build-deps /usr/src/app/.wwebjs_auth ./.wwebjs_auth 

ENV PORT=3000
EXPOSE 3000

# Repete a instalação das dependências na imagem final para garantir
RUN apt-get update && apt-get install -y \
    libxss1 \
    libasound2 \
    libatk-bridge2.0-0 \
    libgtk-3-0 \
    libnss3 \
    libx11-xcb1 \
    libxtst6 \
    libgbm-dev \
    libdrm2 \
    libxshmfence1 \
    && rm -rf /var/lib/apt/lists/*

CMD [ "node", "dist/server.js" ]