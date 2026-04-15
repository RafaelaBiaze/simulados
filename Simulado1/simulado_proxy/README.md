# Entrega Simulado A - Proxy Reverso e Load Balancer

**Aluno(a):** Rafaela Bianor de Azevedo
**RA:** 6324518
**Disciplina:** Implementação de Sistemas

---

# Respostas Dissertativas:
1. **Questão 1: Ciclo de Vida e Ambientes**
  * Diferença: Desenvolvimento é flexível e usa dados de teste. Produção exige estabilidade, segurança e usa dados reais.

  * Falha no Deploy: Ocorre por conflito de ambiente (diferenças de dependências e configuração entre a máquina local e o servidor).

2. **Questão 2: Máquinas Virtuais vs. Containers**
  * Menor sobrecarga: Containers Docker.

  * Por quê: Containers compartilham o Kernel do sistema Host. VMs instalam um sistema operacional inteiro (Guest OS) para cada aplicação, o que desperdiça muita CPU e RAM.

3. **Questão 3: Nginx (Proxy Reverso / Load Balancer)**
  * Função do upstream: Criar um grupo de servidores (cluster) para dividir o tráfego.

  * Se um servidor cair: O Nginx percebe a falha, o remove temporariamente da rota e passa a enviar os usuários apenas para os servidores saudáveis.

---
# Respostas Prática:

## 1. Arquitetura e Arquivos de Configuração

Abaixo estão os manifestos utilizados para subir a infraestrutura, garantindo o isolamento da rede para os servidores web e a exposição apenas do Proxy Reverso (Nginx).

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
      - rede_proxy
  
  web1: 
    image: nginxdemos/hello
    networks:
      - rede_proxy
  
  web2:
    image: nginxdemos/hello
    networks:
      - rede_proxy

networks:
  rede_proxy:
    driver: bridge
```

### nginx.conf (Load Balancer)

```nginx
events {
    worker_connections 1024;
}

http {
    upstream meu_cluster {
        server web1:80;
        server web2:80;
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://meu_cluster;
        }
    }
}
```

## 2. Comandos de Execução e Validação

Para subir o ambiente e validar o balanceamento de carga, os seguintes comandos foram executados no terminal WSL:

1. Validação de Sintaxe:
```bash
docker compose config
```

2. Subida do Ambiente em Background:
```bash
docker compose up -d
```

3. Verificação dos Contêineres Ativos:
```bash
docker compose ps
```

## 3. Evidências de Validação

1. Print saída do comando `docker compose ps` mostrando o proxy e as duas instâncias web rodando simultaneamente.
  ![docker-compose-ps](./imgs/Captura%20de%20tela%202026-04-15%20141144.png)

2. Print do navegador acessando http://localhost e mostrando a resposta do servidor (usando nginxdemos/hello, mostra que o "Server Name" altera ao atualizar a página, comprovando o balanceamento).
  * Web1 
  ![prints-web1](./imgs/Captura%20de%20tela%202026-04-15%20141213.png)
  * Web2
  ![prints-web2](./imgs/Captura%20de%20tela%202026-04-15%20141227.png)

3. Saída do comando docker compose logs proxy demonstrando os acessos sendo roteados.
  ![docker-compose-logs](./imgs/Captura%20de%20tela%202026-04-15%20141340.png)