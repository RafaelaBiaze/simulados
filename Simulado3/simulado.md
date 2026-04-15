# Simulado de Prova C - Arquitetura Completa, Redes Isoladas e Resiliência (Difícil)

**Disciplina:** Implementação de Software
**Tópicos:** Docker Compose Avançado, Redes Isoladas (Bridge), Nginx Routing, Persistência e Healthchecks.

---

## Parte I: Questões Dissertativas (Teoria e Conceitos)

**Questão 1: Segurança e Isolamento de Redes** Em uma arquitetura orquestrada pelo Docker Compose contendo um Frontend, uma API (Backend) e um Banco de Dados, é uma boa prática de segurança criar redes separadas (ex: `rede_frontend` e `rede_backend`). Explique teoricamente como você configuraria essas redes para garantir que o contêiner do Frontend consiga se comunicar com a API, mas seja fisicamente impossível que ele acesse o Banco de Dados diretamente.

**Questão 2: Resiliência e Monitoramento (Healthcheck)** Você configurou o seu `docker-compose.yml` com a diretiva `restart: always` para o serviço da sua aplicação. Porém, o professor explicou que apenas isso não garante que a aplicação esteja funcionando, pois o processo interno do Node.js/Python pode travar, mesmo com o contêiner de pé. Qual é o papel da instrução `HEALTHCHECK` nesse cenário e como o Docker age quando um contêiner é classificado como *unhealthy*?

**Questão 3: Roteamento de Caminhos (Path Routing) no Nginx** Em laboratórios anteriores, usamos o Nginx para balancear carga. Agora, precisamos usá-lo para rotear caminhos diferentes para contêineres diferentes no mesmo domínio. Explique a diferença lógica entre configurar um bloco `location /` e um bloco `location /api/` dentro do arquivo `nginx.conf`.

---

## Parte II: Desafio Técnico (Mão na Massa)

**Cenário:** A clínica veterinária **PetVital** está modernizando sua infraestrutura e precisa implantar o seu novo sistema de agendamento de consultas. A aplicação foi desenhada em microsserviços. Sua missão é orquestrar todo o ambiente garantindo segurança, persistência e roteamento correto do tráfego.

### Requisitos do Desafio:

**1. Preparação do Ambiente:**
* Crie a pasta `simulado_petvital`.
* Dentro dela, crie um `Dockerfile` (para o frontend), um `nginx.conf` (para o proxy) e o manifesto `docker-compose.yml`.
* Crie um arquivo `index.html` simples com uma mensagem de boas-vindas da clínica para simular o site.

**2. Arquivos de Configuração e Entrega (Código):**
* **Dockerfile (Frontend):** Crie uma imagem personalizada baseada no `nginx:alpine` que copie o seu `index.html` para `/usr/share/nginx/html`. Adicione um `HEALTHCHECK` simples usando `curl` para testar o `localhost`.
* **nginx.conf (Proxy Reverso Edge):** Configure o Nginx para escutar na porta 80 e fazer o roteamento:
  * Requisições para a raiz (`/`) devem ser direcionadas para o contêiner do frontend.
  * Requisições para `/api/` devem ser direcionadas para o contêiner do backend.
* **docker-compose.yml (A Orquestração Mestra):**
  * **Serviço 1 (proxy):** Usa a imagem oficial `nginx:alpine`, expõe a porta 8080 do host para a 80 do contêiner, e monta o `nginx.conf` via volume.
  * **Serviço 2 (frontend):** Faz o build do Dockerfile local. Não expõe portas para o host.
  * **Serviço 3 (backend):** Usa a imagem `nginxdemos/hello` para simular as respostas de uma API em formato JSON/texto. Não expõe portas.
  * **Serviço 4 (database):** Usa a imagem `postgres:13-alpine`. Configure a senha via variável de ambiente. Adicione um volume nomeado para persistir os dados do Postgres.
  * **Redes:** Crie e aplique restrições: o `proxy` só enxerga o `frontend` e `backend`. O `database` só enxerga o `backend`.

**3. Evidências de Validação (Prints Obrigatórios):**
* Saída do comando `docker compose up -d --build` demonstrando o build do frontend e a criação correta das redes e volumes.
* Print do comando `docker compose ps` mostrando todos os 4 serviços rodando (e o frontend constando como `healthy`).
* Print do navegador acessando `http://localhost:8080/` (mostrando o site da clínica) e outro acessando `http://localhost:8080/api/` (mostrando a tela da imagem `nginxdemos/hello`).