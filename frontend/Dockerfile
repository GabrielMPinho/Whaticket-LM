FROM node:14 as build-deps
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
WORKDIR /usr/share/nginx/html
COPY --from=build-deps /usr/src/app/build .

# Remove a configuração padrão do Nginx para evitar conflitos
RUN rm -f /etc/nginx/conf.d/default.conf

# Cria a configuração do Nginx AUTOMATICAMENTE via comando
# Isso elimina o erro de "arquivo não encontrado" e garante as rotas certas.
RUN echo 'server { \
    listen 80; \
    server_name localhost; \
    root /usr/share/nginx/html; \
    index index.html index.htm; \
    client_max_body_size 20M; \
    \
    location / { \
        try_files $uri $uri/ /index.html; \
    } \
    \
    location /socket.io/ { \
        proxy_pass http://backend:3000; \
        proxy_http_version 1.1; \
        proxy_set_header Upgrade $http_upgrade; \
        proxy_set_header Connection "Upgrade"; \
        proxy_set_header Host $host; \
        proxy_cache_bypass $http_upgrade; \
    } \
    \
    location ~ ^/(auth|tickets|whatsapp|contacts|messages|users|queues|quick-answers|tags|settings|api-tokens|dashboards|helps|connections|medias|campaigns|schedules|chatbots|plans|subscriptions|invoices|files|prompts|queueIntegration) { \
        proxy_pass http://backend:3000; \
        proxy_set_header Host $host; \
        proxy_set_header X-Real-IP $remote_addr; \
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; \
        proxy_set_header X-Forwarded-Proto $scheme; \
    } \
}' > /etc/nginx/conf.d/default.conf

# Instala dependências extras para garantir compatibilidade
RUN apk add --no-cache jq openssl

EXPOSE 80