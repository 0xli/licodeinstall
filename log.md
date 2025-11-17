1. 03/22/2025 - disk is full, licode stoped
licode initLicode.sh return :
2025-03-23 04:46:49.539  - ERROR: RPC - message: AMQP connection error killing process, errorMsg: errno: ECONNRESET, code: ECONNRESET, syscall: read
   
sudo systemctl status rabbitmq-server
sudo systemctl restart rabbitmq-server

2. 11/17/2025 - no meeting room listed
   - pm2 status
   - pm2 stop 3 2 1 0
   - restart not solve
   - the key issues is https://sh.callt.net/getRooms/
   - <img width="1858" height="222" alt="image" src="https://github.com/user-attachments/assets/9ce7294b-145c-47fe-acd7-9e01aa805277" />
   need sudo nginx -s reload to reload the certificate , fix!!!
