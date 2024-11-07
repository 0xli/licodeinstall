## 启动命令
```
cd /sfu/licode
./scripts/initLicode.sh 
./scripts/initBasicExample.sh 
```
# licode document - Ubuntu 20.04
Licode installation from source is officially compatible with Ubuntu 20.04.
- https://licode.readthedocs.io/en/master/from_source/

- https://github.com/lynckia/licode

- https://licode.readthedocs.io/en/stable/docker/
# licode document - Ubuntu 22.04
vi scripts/installUbuntuDeps.sh
```
check_version(){
  if [[ $(lsb_release -rs) == "18.04" ]] || [[ $(lsb_release -rs) == "20.04" ]]
  then
     gcc_version=10
  else
     gcc_version=12
  fi
}
```
## 代码分析
https://www.cnblogs.com/bloglearning/p/11930199.html

## WebRTC基础知识
- SDP

https://datatracker.ietf.org/doc/html/draft-ietf-mmusic-msid

https://webrtchacks.com/sdp-anatomy/

https://segmentfault.com/a/1190000038272539
## WebRTC debug tools
https://blog.mozilla.org/webrtc/new-tool-debugging-webrtc/

## ios Client
https://github.com/ethand91/mediasoup-ios-client.git

## 0. install on Mac OS
### compile error 
it may fail when run installErizo.sh

```
[ 42%] Building CXX object erizo/CMakeFiles/erizo.dir/DtlsTransport.cpp.o
In file included from /Users/liwei/test/sfu/licode/erizo/src/erizo/DtlsTransport.cpp:5:
In file included from /Users/liwei/test/sfu/licode/erizo/src/erizo/DtlsTransport.h:11:
In file included from /Users/liwei/test/sfu/licode/erizo/src/erizo/dtls/DtlsSocket.h:21:
In file included from /Users/liwei/test/sfu/licode/erizo/src/erizo/dtls/../logger.h:25:
In file included from /usr/local/include/log4cxx/logger.h:30:
In file included from /usr/local/include/log4cxx/helpers/appenderattachableimpl.h:27:
In file included from /usr/local/include/log4cxx/spi/appenderattachable.h:26:
In file included from /usr/local/include/log4cxx/logstring.h:28:
In file included from /usr/local/include/log4cxx/log4cxx.h:45:
/usr/local/include/log4cxx/boost-std-configuration.h:10:18: error: no type named 'shared_mutex' in namespace 'std'; did you mean 'timed_mutex'?
    typedef std::shared_mutex shared_mutex;
            ~~~~~^~~~~~~~~~~~
                 timed_mutex
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX11.3.sdk/usr/include/c++/v1/mutex:235:24: note: 'timed_mutex' declared here
class _LIBCPP_TYPE_VIS timed_mutex
                       ^
1 error generated.
make[2]: *** [erizo/CMakeFiles/erizo.dir/DtlsTransport.cpp.o] Error 1
make[1]: *** [erizo/CMakeFiles/erizo.dir/all] Error 2
make: *** [all] Error 2
```
### fix by modify the .h file or the compile flags
https://stackoverflow.com/questions/36222912/is-c17-stdshared-mutex-not-available-yet

-std=gnu++17 or -std=c++17 flag

or 
```
#vi /usr/local/include/log4cxx/boost-std-configuration.h
line 10 from 
    typedef std::shared_mutex shared_mutex;
to:    
    typedef std::shared_timed_mutex shared_mutex;

#liweideMacBook-Pro:licode liwei$ ./scripts/installErizo.sh 
```
### more errors
```
> licode-erizo-api@0.1.0 preinstall /Users/liwei/test/sfu/licode/erizoAPI
> ./lint.sh

./lint.sh: line 3: cpplint: command not found
npm ERR! code ELIFECYCLE
npm ERR! syscall spawn
npm ERR! file sh
npm ERR! errno ENOENT
npm ERR! licode-erizo-api@0.1.0 preinstall: `./lint.sh`
npm ERR! spawn ENOENT
npm ERR! 
npm ERR! Failed at the licode-erizo-api@0.1.0 preinstall script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.
npm WARN Local package.json exists, but node_modules missing, did you mean to install?

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/liwei/.npm/_logs/2021-06-24T06_13_52_050Z-debug.log

```
 brew install cpplint
 
## 1. install ubuntu in docker for development
### 1.1 install docker on centos 
```
  sudo yum update
  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  yum list docker-ce --showduplicates | sort -r
  sudo yum install docker-ce
  sudo systemctl start docker
  sudo systemctl enable docker
```
### 1.2 change data-root if needed 
```
vi /etc/docker/daemon.json
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"],
  "storage-driver": "devicemapper", 
  "data-root": "/home/allcom/docker.old"
}
only needed if the diretory is mounted by NFS, should use "storage-driver": "devicemapper", 
```
### 1.3 ubuntu docker container 
```
get localhost IP, also port and directory map
PUBLIC_IP=$(ifconfig eth0 | awk '/ *inet /{print $2}') MIN_PORT=30000; MAX_PORT=30050; sudo docker run --name licodedev -p  3000:3000 -p $MIN_PORT-$MAX_PORT:$MIN_PORT-$MAX_PORT/udp -p 3001:3001 -p 3002:3002 -p 8030:8030 -e "MIN_PORT=$MIN_PORT" -e "MAX_PORT=$MAX_PORT" -e "PUBLIC_IP=$PUBLIC_IP" -e "NETWORK_INTERFACE=eth0" -t -i -v /data/sfu:/sfu ubuntu /bin/bash

# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    7e0aa2d69a15   6 weeks ago   72.7MB
```
### 1.4 open port on the host 
```
# firewall-cmd --add-port=3000/tcp --permanent
# firewall-cmd --add-port=3001/tcp --permanent
# firewall-cmd --add-port=3004/tcp --permanent
# firewall-cmd --add-port=8030/tcp --permanent
# firewall-cmd --add-port=30000-30050/udp --permanent
# firewall-cmd --query-port=3000/tcp
# firewall-cmd --reload
```
## 2. install licode
### 2.1 install Dependencies
```
#apt-get update
#apt-get install vim curl wget git net-tools inetutils-ping -y
# vi ~/.wgetrc
proxy=http://192.168.0.163:10809
#vi ~/.curlrc
#socks5
socks5 = "127.0.0.1:1080"
#或者HTTP代理
proxy = "127.0.0.1:9999"

#git clone https://github.com/lynckia/licode.git
#cd licode

#scripts/installUbuntuDeps
modify script file
licode mongodb Cannot change ownership to uid 1000, gid 1000
tar --no-same-owner

tar --no-same-owner -zxvf /sfu/licode/scripts/../build/libdeps/mongodb-linux-x86_64-ubuntu2004-4.4.4.tgz -C /sfu/licode/scripts/../build/libdeps

```
#### delete 
```
ln: failed to create symbolic link '/usr/local/bin/install_compass': File exists
ln: failed to create symbolic link '/usr/local/bin/mongo': File exists
ln: failed to create symbolic link '/usr/local/bin/mongod': File exists
ln: failed to create symbolic link '/usr/local/bin/mongos': File exists

rm -rf /usr/local/bin/install_compass
rm -rf /usr/local/bin/mongo
rm -rf /usr/local/bin/mongod
rm -rf /usr/local/bin/mongos
```
### 2.2 install Licode
```
#./scripts/installNuve.sh 
# ./scripts/installErizo.sh 
```
#### Attentions
- scripts/installNuve.sh will change the licode_config.js every time with a new new service ID and service KEY
- erizo will use the ID and KEY to access nuve. If not correct, the basicExample will stuck at the N.API call and will not show listen on port 30001

```
config.nuve.superserviceID = '60b90bf49a80e18de6fe0bf5'; // default value: ''
config.nuve.superserviceKey = '1168'; // default value: ''

```
- minport and maxport must be consistent with the -p when create the docker container
- sudo ufw allow 30000:30050/udp
- sudo ufw allow 3000,3001,3004,8030/tcp
```
//note, this won't work with all versions of libnice. With 0 all the available ports are used
config.erizo.minport = 30000; // default value: 0
config.erizo.maxport = 30050; // default value: 0
// this should be udp port
sudo ufw allow 30000:30050/udp
sudo docker ps
860e2df40368   1edc79a7ed54   "bash"        3 days ago   Up 3 days   0.0.0.0:3001->3001/tcp, :::3001->3001/tcp, 0.0.0.0:3300->3300/tcp, :::3300->3300/tcp, 0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:30000-30050->30000-30050/udp, :::30000-30050->30000-30050/udp   licodedev

```
- public IP, turn server and SSL
```
config.erizoController.iceServers = [{'url': 'stun:stun.l.google.com:19302'}]; // default value: [{'url': 'stun:stun.l.google.com:19302'}]

// Default and max video bandwidth parameters to be used by clients for both published and subscribed streams
config.erizoController.defaultVideoBW = 300; //default value: 300
config.erizoController.maxVideoBW = 300; //default value: 300

// Public erizoController IP for websockets (useful when behind NATs)
// Use '' to automatically get IP from the interface
config.erizoController.publicIP = '192.168.0.183'; //default value: ''
config.erizoController.networkinterface = 'eth0'; //default value: ''

// This configuration is used by the clients to reach erizoController
// Use '' to use the public IP address instead of a hostname
config.erizoController.hostname = 'sfu.callpass.cn'; //default value: ''
config.erizoController.port = 8080; //default value: 8080
// Use true if clients communicate with erizoController over SSL
config.erizoController.ssl = true; //default value: false

// This configuration is used by erizoController server to listen for connections
// Use true if erizoController listens in HTTPS.
config.erizoController.listen_ssl = true; //default value: false
config.erizoController.listen_port = 8080; //default value: 8080

// Custom location for SSL certificates. Default located in /cert
//config.erizoController.ssl_key = '/full/path/to/ssl.key';
config.erizoController.ssl_key = '/sfu/licode/cert/callpass.cn.key';
config.erizoController.ssl_cert = '/sfu/licode/cert/fullchain.cer';

config.erizo.stunserver = 'XXXXX'; // default value: ''
config.erizo.stunport = 3478; // default value: 0

//TURN server IP address and port to be used by the server.
//Please note this is not needed in most cases, setting TURN in erizoController for the clients
//is the recommended configuration
//if '' is used, no relay for the server is used
config.erizo.turnserver = 'XXXXX'; // default value: ''
config.erizo.turnport = 3478; // default value: 0
config.erizo.turnusername = 'XXXX';
config.erizo.turnpass = 'XXXXX';
config.erizo.networkinterface = 'eth0'; //default value: ''

```
### 2.3 install BasicExamples
```
# ./scripts/installBasicExample.sh 
```
### 2.4 kill process
```
root     14645     1  2 10:05 pts/0    00:00:00 node nuve.js
root     14949     1  1 10:05 pts/0    00:00:00 node erizoController.js
root     15248     1  0 10:05 pts/0    00:00:00 node erizoAgent.js
root     15255 15248  1 10:05 ?        00:00:00 node ./../erizoJS/erizoJS.js 7df541ba-04dd-88e7-ca65-bf90cd686c1f 172.17.0.2 192.168.0.172 -d
root     15589 15304  3 10:06 pts/0    00:00:00 node basicServer.js

#ps -ef | grep node | grep \.js | awk '{print "kill "$2}' | sh

```

## 3. run and maintain
### 3.1 run db first
https://www.runoob.com/mongodb/mongodb-osx-install.html
```
start mongodb
#mongod --dbpath build/db --logpath build/mongoa.log --fork
command line
#mongo
>show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
nuvedb  0.000GB
>use nuvedb
>db.erizoControllers.totalSize()
36864
>db.rooms.count()
> db.rooms.find().pretty()
{
	"_id" : ObjectId("60d42a74a6ca1f38e3e320bc"),
	"name" : "basicExampleRoom",
	"data" : {
		"basicExampleRoom" : true
	},
	"mediaConfiguration" : "default",
	"erizoControllerId" : null
}
{
	"_id" : ObjectId("60ebb8fe29ad24fbdb5ba00f"),
	"name" : "ios",
	"data" : {
		"type" : "videoconference",
		"public" : true
	}
}

```

## 4. Disk usage
```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G  996K  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vda1        40G   29G  8.9G  77% /
tmpfs           783M     0  783M   0% /run/user/0
overlay          40G   29G  8.9G  77% /var/lib/docker/overlay2/86296fb49a2fbce60bdd5762dd2dda6abf43262386f76b32e95a0421cf2d877c/merged
# du -h -d 1
16K     ./plugins
4.0K    ./swarm
4.0K    ./tmp
352K    ./image
4.0K    ./runtimes
80K     ./network
88K     ./buildkit
2.7G    ./overlay2
28K     ./volumes
2.2M    ./containers
4.0K    ./trust
2.7G    .
```
