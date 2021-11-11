on 2021-11-10 the cloud service exprired 
after pay and restart the software on the server
run licode in the docker, the rest on the host
# 1. docker
docker container licodede
docker exec -it licodedev bash

# 2. licode
```
cd licode
mongod --dbpath build/db --logpath build/mongoa.log --fork
scripts/initLicode.sh
scripts/initBasicExample
```
https://node.kademlia.network:3001
# 3. licode admin
```
cd /sfu/licode-demos
pm2 start app.js
```
https://node.kademlia.network:8443/Admin
# 4. socket-io
```
cd /sfu/socket-io
pm2 start room.js
```
https://node.kademlia.network:3002

# 5. push service
```
cd /appdeploy/push-api/
./start.sh
```
database is in push_chain @ mysql
