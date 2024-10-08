docker exec -t configSrv mongosh --port 27017 --quiet <<EOF
rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
)
EOF

docker exec -it shard1 mongosh --port 27018 --quiet <<EOF
rs.initiate(
    {
      _id : "shard1",
      members: [
        { _id : 0, host : "shard1:27018" },
      ]
    }
)
EOF

docker exec -it shard2 mongosh --port 27021 --quiet <<EOF
rs.initiate(
    {
      _id : "shard2",
      members: [
        { _id : 1, host : "shard2:27021" }
      ]
    }
  )
EOF

docker exec -it mongos_router mongosh --port 27024 --quiet <<EOF
sh.addShard( "shard1/shard1:27018")
sh.addShard( "shard2/shard2:27021")
sh.enableSharding("somedb")
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )
use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})
db.helloDoc.countDocuments() 
EOF

docker exec -it shard1 mongosh --port 27018 --quiet <<EOF
rs.initiate({_id: "shard1", members: [
{_id: 0, host: "shard1:27018"},
{_id: 1, host: "shard1_mongodb2:27019"},
{_id: 2, host: "shard1_mongodb3:27020"}
]})
EOF

docker exec -it shard2 mongosh --port 27021 --quiet <<EOF
rs.initiate({_id: "shard2", members: [
{_id: 0, host: "shard2:27018"},
{_id: 1, host: "shard2_mongodb2:27022"},
{_id: 2, host: "shard2_mongodb3:27023"}
]})
EOF