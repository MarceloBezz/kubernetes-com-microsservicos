# Kubernetes com Microsserviços

Este projeto é um ambiente de aprendizado de Kubernetes baseado em uma arquitetura de microsserviços Java/Spring Boot. O foco principal é estudar como empacotar serviços, expô-los com recursos do Kubernetes e integrar aplicações distribuídas usando `Deployment`, `StatefulSet`, `Service`, `ConfigMap`, `Secret` e `PersistentVolumeClaim`.

## Visão geral

A ideia central do projeto é oferecer uma base prática para aprender Kubernetes com um conjunto de microsserviços que se comunicam entre si:

- `gateway/` — API Gateway Spring Cloud Gateway.
- `pagamentos/` — serviço de pagamentos.
- `pedidos/` — serviço de pedidos.
- `server/` — serviço Eureka Server (registro de serviços).

A pasta `k8s/` contém os manifestos Kubernetes que orquestram os serviços e dependências:

- `app.yaml` — deployments e statefulsets dos microsserviços.
- `services.yaml` — `Service` para cada componente.
- `configMap.yaml` — configuração externa para `SERVER_HOST` e `DB_HOST`.
- `secrets.yaml` — credenciais para acesso ao MySQL.
- `mysql.yaml` — StatefulSet do banco MySQL.
- `volumes.yaml` — `PersistentVolumeClaim` para dados do MySQL.
- `loadbalancer.yaml` — recursos de balanceamento se necessário.

## Estrutura do projeto

Cada serviço é um projeto Maven isolado com seu próprio `pom.xml`, `Dockerfile` e código-fonte em `src/main/java`.

- `gateway/`
  - `Dockerfile`
  - `pom.xml`
  - `src/main/java` e `src/main/resources`
- `pagamentos/`
  - `Dockerfile`
  - `pom.xml`
  - `src/main/java` e `src/main/resources`
- `pedidos/`
  - `Dockerfile`
  - `pom.xml`
  - `src/main/java` e `src/main/resources`
- `server/`
  - `Dockerfile`
  - `pom.xml`
  - `src/main/java` e `src/main/resources`

## Como executar

### Pré-requisitos

- `kubectl`
- `docker` ou outro runtime compatível com Kubernetes
- cluster Kubernetes local (`minikube`, `kind`, `Docker Desktop`, etc.)
- `maven` ou o wrapper Maven (`mvnw` / `mvnw.cmd`)

### Passos básicos

1. Inicie o cluster Kubernetes local.

   Exemplo com `minikube`:
   ```bash
   minikube start
   eval $(minikube docker-env)
   ```

2. Construa as imagens Docker locais para cada serviço.

   No diretório raiz do projeto:
   ```bash
   docker build -t java-gateway-k8s:v3 ./gateway
   docker build -t java-pagamentos-k8s:v4 ./pagamentos
   docker build -t java-pedidos-k8s:v3 ./pedidos
   docker build -t java-server-k8s:v3 ./server
   ```

   > Observação: os manifests atuais referenciam imagens com tags do Docker Hub (`marcelobezz07/...`). Se estiver usando localmente, substitua essas referências pelos nomes de imagem que você construiu ou publique-as em um registro.

3. Aplique os manifestos Kubernetes.

   ```bash
   kubectl apply -f k8s/configMap.yaml
   kubectl apply -f k8s/secrets.yaml
   kubectl apply -f k8s/volumes.yaml
   kubectl apply -f k8s/mysql.yaml
   kubectl apply -f k8s/services.yaml
   kubectl apply -f k8s/app.yaml
   ```

4. Verifique se os pods e serviços estão em execução.

   ```bash
   kubectl get pods
   kubectl get services
   ```

### Acesso e testes

- O `gateway` expõe uma porta HTTP sob o container e geralmente pode ser acessado via serviço no cluster.
- O `server` é o registry Eureka, usado pelos outros microsserviços para descoberta.
- O MySQL é provisionado como `StatefulSet` com `PersistentVolumeClaim` para persistência de dados.

## Observações sobre o aprendizado

Este projeto é ideal para praticar:

- Deploy de microsserviços no Kubernetes
- Uso de `ConfigMap` e `Secret` para configuração e credenciais
- Deploy de banco de dados com `StatefulSet` e volume persistente
- Consumo de serviços e descoberta com Spring Cloud/Eureka
- Organização de um projeto de microsserviços em múltiplos repositórios/pastas

## Dicas

- Se usar `minikube`, ative o perfil Docker com `eval $(minikube docker-env)` antes de construir as imagens.
- Para testar sem publicar imagens, altere as referências em `k8s/app.yaml` para os nomes locais construídos.
- Caso prefira, crie um registry local ou use `kind load docker-image` para carregar imagens no cluster.

---

A proposta principal do repositório é servir de laboratório Kubernetes; a camada Java/Spring Boot está presente como cenário para aprender orquestração e deploy em cluster.
