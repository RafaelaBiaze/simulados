# Simulado de Prova A - Foco em Nginx, Proxy Reverso e Fundamentos Docker (Fácil)

**Disciplina:** Implementação de Software
**Tópicos:** Ciclo de Vida de Implantação, Servidores Web (Nginx), Proxy Reverso, Load Balancer e Fundamentos de Containerização.

---

## Parte I: Questões Dissertativas (Teoria e Conceitos)

**Questão 1: Ciclo de Vida e Ambientes** Descreva a principal diferença entre um ambiente de Desenvolvimento e um ambiente de Operação (Produção). Por que um código que funciona perfeitamente no ambiente de desenvolvimento local pode apresentar falhas quando implantado manualmente em um servidor de produção?

**Questão 2: Máquinas Virtuais vs. Containers** Um arquiteto de software precisa implantar 15 microserviços diferentes em um servidor. Ele está em dúvida entre utilizar Máquinas Virtuais (Hypervisor Tipo 2) ou Containers Docker. Qual abordagem apresenta menor sobrecarga (overhead) de recursos de sistema (CPU/RAM) e por quê? Explique com base na arquitetura de cada tecnologia.

**Questão 3: Nginx como Proxy Reverso e Balanceador de Carga** No contexto de alta disponibilidade, explique qual é a função diretiva `upstream` dentro do arquivo `nginx.conf`. Se um dos servidores backend configurados no `upstream` cair, como o Nginx (agindo como Load Balancer) se comporta por padrão (Round Robin) em relação às novas requisições dos clientes?

---

## Parte II: Desafio Técnico (Mão na Massa)

**Cenário:** Você foi designado para configurar a camada de roteamento de tráfego de um novo portal de notícias. Para suportar a carga, o sistema usará um proxy reverso Nginx para balancear as requisições entre duas instâncias isoladas de um servidor web estático.

### Requisitos do Desafio:

**1. Preparação do Ambiente:**
* Crie uma estrutura de diretórios chamada `simulado_proxy/app`.
* Dentro dessa estrutura, você precisará de um arquivo `docker-compose.yml` e de um arquivo `nginx.conf` personalizado.

**2. Arquivos de Configuração e Entrega (Código):**
* **`nginx.conf`**: Crie a configuração de um proxy reverso que escute na porta 80 e encaminhe o tráfego para um bloco `upstream` contendo os dois servidores backend (`web1` e `web2`).
* **`docker-compose.yml`**: Defina três serviços:
  * **proxy**: Usando a imagem `nginx:alpine`, mapeando a porta 80 do host para a 80 do container, e montando o arquivo `nginx.conf` local como volume.
  * **web1** e **web2**: Usando a imagem `nginxdemos/hello` ou `httpd:alpine`, sem expor portas diretamente para o host (isolamento de rede).
* O arquivo final de entrega deve se chamar `README.md` e conter todos os blocos de código (YAML e Conf).

**3. Evidências de Validação:**
* **Comprovação Visual**: Envio obrigatório de capturas de tela (prints).
  * Saída do comando `docker compose ps` mostrando o proxy e as duas instâncias web rodando simultaneamente.
  * Print do navegador acessando `http://localhost` e mostrando a resposta do servidor (se usar `nginxdemos/hello`, mostre que o "Server Name" altera ao atualizar a página, comprovando o balanceamento).
  * Saída do comando `docker compose logs proxy` demonstrando os acessos sendo roteados.