# Desenvolva um gerenciador de pedidos de um restaurante com JavaScript e Kafka


Este é um projeto simples de um restaurante utilizando uma arquitetura de microsserviços. Exitem 2 APIs (Garçom e App), onde o cliente pode fazer os seus pedidos. Essas APIs enviam o pedido para a cozinha, já separando o que é para o cozinheiro fazer e o que é para o barman fazer. Depois o cozinheiro e o barman enviam o pedido para o balcão, de onde o garçom pega e entrega na mesa, ou para o balcão do delivery, onde os pedidos são separados para serem entregues pelo motoboy.

alexrogeriodj 2021

## Rodando o Projeto com Docker e Docker Compose

Como são várias aplicações, a maneira mais fácil de rodar todas elas é com o [Docker Compose](https://docs.docker.com/compose/).

### Dependências

Antes de começar a instalar e rodar o projeto é necessário instalar algumas dependências:

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)

### Clonando os repositórios

Em primeiro lugar, devemos clonar esse repositório que contém os arquivos de configuração do Docker Compose:



Entre no diretório onde o repositório foi clonado:

```bash
cd dio-restaurant
```

Agora é preciso clonar os repositórios das outras aplicações:


```

Atenção ao definir o nome da pasta onde serão clonados os repositórios. Caso escolha outro nome para os diretórios, é preciso mudar os parâmetros `build.context` do [docker-compose.yml]

### Rodando o projeto

Agora vamos rodar as aplicações!

OBS: Para rodar o projeto da maneira que será mostrada é preciso estar sempre dentro do diretório raiz do projeto, onde clonamos o [repositório com o docker compose]
#### Primeira execução

Na primeira vez que subirmos a instância do kafka precisamos criar os tópicos que serão utilizados no projeto. Para isso, primeiro subimos a aplicação:

```bash
docker-compose up -d kafka
```

OBS: às vezes o container do kafka tem alguns problemas para subir na primeira vez, então sempre aguarde uns 5 segundos e rode o último comando novamente.

Agora precisamos criar os tópicos que serão usados. Para isso, rodamos os seguintes comandos:

```bash
docker-compose exec kafka kafka-topics --create --topic order --partitions 4 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
docker-compose exec kafka kafka-topics --create --topic balcony --partitions 4 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
docker-compose exec kafka kafka-topics --create --topic deliveryBalcony --partitions 4 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
docker-compose exec kafka kafka-topics --create --topic delivery --partitions 4 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
```

#### Executando o projeto

Agora que o kafka já está rodando e já temos os tópicos criados, podemos subir as nossas aplicações:

```bash
dkc up delivery-app waiter cooker bartender motoboy
```

Caso alguma aplicação dê o erro `"Error: Local: Broker transport failure"`, é um pequeno problema que às vezes temos com o kafka dockerizado. Para corrigir, basta parar a aplicação (`Ctrl + C`) e rodar o comando acima novamente para subir as aplicações.

#### Utilizando as APIs

A API do [Garçom] está exposta na url `http://localhost:3000` e a do [App] na url `http://localhost:4000`.

Exemplo de request para fazer o pedido para o Garçom:

```bash
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{ "table": 1, "food": ["food1", "food2"], "drinks": ["drink1", "drink2"] }' \
  http://localhost:3000/order
```

Exemplo de request para fazer o pedido para o App:

```bash
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{ "address": "address 1", "food": ["food1", "food2"], "drinks": ["drink1", "drink2"] }' \
  http://localhost:4000/order
```
