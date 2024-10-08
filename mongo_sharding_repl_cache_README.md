docker exec -it shard1 mongosh  mongodb://127.0.0.1:27018

rs.initiate({_id: "shard1", members: [ {_id: 0, host: "shard1:27018"}, {_id: 1, host: "shard1_mongodb2:27019"}, {_id: 2, host: "shard1_mongodb3:27020"}]}) 

exit

docker exec -it shard2 mongosh mongodb://127.0.0.1:27021

rs.initiate({_id: "shard2", members: [ {_id: 0, host: "shard2:27021"}, {_id: 1, host: "shard2_mongodb2:27022"}, {_id: 2, host: "shard2_mongodb3:27023"}]})

exit

docker exec -it configsrv mongosh mongodb://127.0.0.1:27017
exit

docker exec -it mongos_router mongosh mongodb://127.0.0.1:27024
sh.addShard( "shard1/shard1:27018")
sh.addShard( "shard2/shard2:27021")
exit
