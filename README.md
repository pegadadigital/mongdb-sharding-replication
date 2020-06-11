# Demonstração de Sharding e Replication no MongoDB usando containers.

## Arquitetura
![Arquitetura](https://miro.medium.com/max/1172/1*PjKoe378CdqLgQm-IHDXgQ.png)

## Criando os nós ConfigServer:
docker run --name mongo-config01 --net net-cluster-mongo -d mongo mongod --configsvr --replSet configserver --port 27017
docker run --name mongo-config02 --net net-cluster-mongo -d mongo mongod --configsvr --replSet configserver --port 27017
docker run --name mongo-config03 --net net-cluster-mongo -d mongo mongod --configsvr --replSet configserver --port 27017

## Configuração do replica-set nos ConfigServers:
docker exec -it mongo-config01 mongo
rs.initiate(
   {
      _id: "configserver",
      configsvr: true,
      version: 1,
      members: [
         { _id: 0, host : "mongo-config01:27017" },
         { _id: 1, host : "mongo-config02:27017" },
         { _id: 2, host : "mongo-config03:27017" }
      ]
   }
)


## Criando os nós shard

## Shard 1
docker run --name ade-node01 --net net-cluster-mongo -d mongo mongod --port 27018 --shardsvr --replSet shard01
docker run --name ade-node02 --net net-cluster-mongo -d mongo mongod --port 27018 --shardsvr --replSet shard01
docker run --name ade-node03 --net net-cluster-mongo -d mongo mongod --port 27018 --shardsvr --replSet shard01

## Shard 2
docker run --name ade-node11 --net net-cluster-mongo -d mongo mongod --port 27019 --shardsvr --replSet shard02
docker run --name ade-node12 --net net-cluster-mongo -d mongo mongod --port 27019 --shardsvr --replSet shard02
docker run --name ade-node13 --net net-cluster-mongo -d mongo mongod --port 27019 --shardsvr --replSet shard02

## Shard 3
docker run --name ade-node21 --net net-cluster-mongo -d mongo mongod --port 27020 --shardsvr --replSet shard03
docker run --name ade-node22 --net net-cluster-mongo -d mongo mongod --port 27020 --shardsvr --replSet shard03
docker run --name ade-node23 --net net-cluster-mongo -d mongo mongod --port 27020 --shardsvr --replSet shard03

## Shard 1
Configurando os replica-sets nos Shards.
docker exec -it ade-node01 mongo --port 27018
rs.initiate(
   {
      _id: "shard01",
      version: 1,
      members: [
         { _id: 0, host : "ade-node01:27018" },
         { _id: 1, host : "ade-node02:27018" },
         { _id: 2, host : "ade-node03:27018" },
      ]
   }
)


## Shard 2
docker exec -it ade-node11 mongo --port 27019
rs.initiate(
   {
      _id: "shard02",
      version: 1,
      members: [
         { _id: 0, host : "ade-node11:27019" },
         { _id: 1, host : "ade-node12:27019" },
         { _id: 2, host : "ade-node13:27019" },
      ]
   }
)


## Shard 3
docker exec -it ade-node21 mongo --port 27020
rs.initiate(
   {
      _id: "shard03",
      version: 1,
      members: [
         { _id: 0, host : "ade-node21:27020" },
         { _id: 1, host : "ade-node22:27020" },
         { _id: 2, host : "ade-node23:27020" },
      ]
   }
)


## Criando nó roteador:
docker run -p 27017:27017 --name mongo-router --net net-cluster-mongo -d mongo mongos --port 27017 --configdb configserver/mongo-config01:27017,mongo-config02:27017,mongo-config03:27017 --bind_ip_all

## Configurando shards no router:
docker exec -it mongo-router mongo
sh.addShard("shard01/ade-node01:27018")
sh.addShard("shard01/ade-node01:27018") 
sh.addShard("shard01/ade-node03:27018") 
sh.addShard("shard02/ade-node11:27019")
sh.addShard("shard02/ade-node12:27019")
sh.addShard("shard02/ade-node13:27019") 
sh.addShard("shard03/ade-node21:27020")
sh.addShard("shard03/ade-node22:27020")
sh.addShard("shard03/ade-node23:27020")
sh.status()


## Criando e configurando uma base e uma collection com shard
use exampleDB
sh.enableSharding("exampleDB")
db.exampleCollection.ensureIndex( { _id : "hashed" } )
sh.shardCollection( "exampleDB.exampleCollection", { "_id" : "hashed" } )

## Inserindo registros:
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 01'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 02'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 03'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 04'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 05'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 06'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 07'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 08'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 09'});
db.exampleCollection.insert({"data_criacao_database":new Date(), "motivo_solicitacao":'Laboratório de ADE 10'});

## Checando a distribuição de dados:
db.exampleCollection.getShardDistribution()

## Checando os dados direto nos shards:

## Shard 1
docker exec -it ade-node01 mongo --port 27018
use exampleDB
db.exampleCollection.find().pretty()

## Shard 2 
docker exec -it ade-node11 mongo --port 27019
use exampleDB
db.exampleCollection.find().pretty()

## Shard 3
docker exec -it ade-node21 mongo --port 27020
use exampleDB
db.exampleCollection.find().pretty()
