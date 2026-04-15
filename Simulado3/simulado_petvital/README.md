# Entrega Simulado C - Arquitetura PetVital (Microserviços)

**Aluno(a):** Rafaela Bianor de Azevedo
**RA:** 6324518
**Disciplina:** Implementação de Sistemas

---

# Respostas Dissertativas

1. **Questão 1: Segurança e Isolamento de Redes**
    * Como configurar: Crie duas redes separadas (ex: `rede_front` e `rede_back`). Coloque o Frontend apenas na `rede_front` e o Banco de Dados apenas na `rede_back`.

    * O truque: Coloque a API conectada a ambas as redes.

    * Por quê: A API atua como uma ponte exclusiva. Como o Frontend e o Banco não compartilham a mesma rede, é fisicamente impossível que se comuniquem diretamente.

2. **Questão 2: Resiliência e Monitoramento (Healthcheck)**
    * Papel do Healthcheck: O `restart: always` só funciona se o contêiner "morrer" de vez. O `HEALTHCHECK` serve para testar se a aplicação travou internamente (processo zumbi), mesmo com o contêiner ainda ligado.

    * Ação do Docker: Se a aplicação falhar nos testes e ficar `unhealthy`, o Docker marca o contêiner como defeituoso, o que permite a sistemas de orquestração "matarem" e recriarem esse contêiner automaticamente.

3. **Questão 3: Roteamento de Caminhos (Path Routing) no Nginx**
    * `location /` (Caminho Raiz): É a rota genérica. Qualquer pessoa que acessar o domínio principal cai aqui. Geralmente direciona o tráfego para a interface visual (Frontend).

    * `location /api/` (Caminho Específico): Intercepta apenas o tráfego que tiver `/api/` na URL. Serve para separar os chamados de sistema e direcionar os dados diretamente para o Backend, permitindo que ambos coexistam na mesma porta (80/8080).

---

# Respostas Prática

## 1. Arquitetura e Arquivos de Configuração

Abaixo estão os manifestos utilizados para subir a infraestrutura da clínica PetVital. O ambiente conta com isolamento de redes (Frontend e Backend), um banco de dados protegido, uma imagem customizada com Healthcheck e um Proxy Reverso roteando os caminhos.

### docker-compose.yml

```yaml
version: '3.8'

services:
  proxy:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - rede_petvital_frontend
      - rede_petvital_backend
    depends_on:
      - backend
      - frontend

  frontend: 
    build: .
    networks:
      - rede_petvital_frontend

  backend: 
    image: nginxdemos/hello
    networks:
      - rede_petvital_backend
  
  database:
    image: postgres:13-alpine
    env_file: 
      - .env
    environment:
      POSTGRES_PASSWORD: ${PG_PASSWORD}
    volumes:
      - database_petvital:/var/lib/postgresql/data
    networks:
      - rede_petvital_backend

volumes:
  database_petvital:

networks:
  rede_petvital_frontend:
    driver: bridge
  rede_petvital_backend:
    driver: bridge
```

### nginx.conf (Proxy Reverso com Path Routing)
```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name localhost;

        # Rota para o site estático (Frontend)
        location / {
            proxy_pass http://frontend:80;
        }

        # Rota para a API (Backend)
        location /api/ {
            proxy_pass http://backend:80;
        }
    }
}
```

### Dockerfile (Frontend Customizado)
```dockerfile
FROM nginx:alpine

# Instalação do curl para o monitoramento
RUN apk add --no-cache curl

# Cópia do site estático
COPY index.html /usr/share/nginx/html/index.html

# Monitoramento de saúde do container
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

EXPOSE 80
```

### .env
```text
PG_PASSWORD=123456
```

## 2. Comandos de Execução e Validação
Para construir a imagem customizada, subir o ambiente e validar o roteamento de caminhos, os seguintes comandos foram executados no terminal WSL:

1. Validação de Sintaxe YAML:
```bash
docker compose config
```

2. Construção da Imagem e Subida do Ambiente:
```bash
docker compose up -d --build
```

3. Verificação do Status e Healthcheck:
```bash
docker compose ps
```

## 3. Evidências de Validação

1. Print saída do comando docker compose up -d --build demonstrando o build do frontend e a criação correta das redes e volumes.
  ![docker-compose-up](./imgs/Captura%20de%20tela%202026-04-15%20155246.png)

2. Print do comando docker compose ps mostrando todos os 4 serviços rodando (e o frontend constando como healthy).
  ![docker-compose-healthy](./imgs/Captura%20de%20tela%202026-04-15%20155309.png)

3. Print do navegador acessando http://localhost:8080/ (mostrando o site da clínica) e outro acessando http://localhost:8080/api/ (mostrando a tela da imagem nginxdemos/hello)
  * Pagina inicial 
  ![prints-home](./imgs/Captura%20de%20tela%202026-04-15%20155425.png)
  * /api
  ![prints-api](./imgs/Captura%20de%20tela%202026-04-15%20155446.png)