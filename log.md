1. 03/22/2025 - disk is full, licode stoped
licode initLicode.sh return :
2025-03-23 04:46:49.539  - ERROR: RPC - message: AMQP connection error killing process, errorMsg: errno: ECONNRESET, code: ECONNRESET, syscall: read
   
sudo systemctl status rabbitmq-server
sudo systemctl restart rabbitmq-server
