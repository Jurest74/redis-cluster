# redis-cluster

<h6>Implementación de un cluster de redis en docker y realizando pruebas con Python</h6>

<h4>Configurar instancia de ec2 AWS</h5>

- Vamos a las instancias y le damos en lanzar instancia

![image](https://user-images.githubusercontent.com/43093044/157374281-77de1620-2751-42f2-9edd-88380e0b3b02.png)

- A continuación debemos configurar la respectiva instancia de ec2, en este caso la usaremos en linux

- Debemos instalar Docker y docker-compose

DOCKER

sudo amazon-linux-extras install docker -y
sudo yum install git -y

sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -a -G docker ec2-user

DOCKER-COMPOSE

sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
exit

- Luego cree el archivo docker-compose.yml y copie el siguiente script en él para la configuración

version: '2.2'
services:
  redis01-master:
    image: redis 
    user: root
    sysctls:
      net.core.somaxconn: 20000
    ulimits:
      nofile:
        soft: 64000
        hard: 64000
    ports:
      - 6379:6379
      - 16379:16379
    volumes:
      - ./redis01-master/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /var/log/redis/redis01-master/redis.log:/var/log/redis/redis.log
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      app_net:
        ipv4_address: 173.17.0.10
  redis02-slave:
    image: redis 
    user: root
    sysctls:
      net.core.somaxconn: 20000
    ulimits:
      nofile:
        soft: 64000
        hard: 64000
    ports:
      - 6382:6379
      - 16382:16379
    volumes:
      - ./redis02-slave/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /var/log/redis/redis02-slave/redis.log:/var/log/redis/redis.log
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      app_net:
        ipv4_address: 173.17.0.50
  redis03-slave:
    image: redis
    user: root
    sysctls:
      net.core.somaxconn: 20000
    ulimits:
      nofile:
        soft: 64000
        hard: 64000
    ports:
      - 6383:6379
      - 16383:16379
    volumes:
      - ./redis03-slave/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /var/log/redis/redis03-slave/redis.log:/var/log/redis/redis.log
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      app_net:
        ipv4_address: 173.17.0.60
  redis04-slave:
    image: redis
    user: root
    sysctls:
      net.core.somaxconn: 20000
    ulimits:
      nofile:
        soft: 64000
        hard: 64000
    ports:
      - 6384:6379
      - 16384:16379
    volumes:
      - ./redis01-slave/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /var/log/redis/redis01-slave/redis.log:/var/log/redis/redis.log
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      app_net:
        ipv4_address: 173.17.0.40
  redis-commander:
    image: rediscommander/redis-commander:latest
    environment:
      - REDIS_HOSTS=redis01-master:173.17.0.10:6379,redis02-master:173.17.0.20:6380,redis03-master:173.17.0.30:6381,redis01-slave:173.17.0.40:6384,redis02-slave:173.17.0.50:6382,redis03-slave:173.17.0.60:6383
    ports:
      - 8081:8081
networks:
  app_net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 173.17.0.0/16
        - gateway: 173.17.0.1
        
        
        
        
- Configuración para los nodos de REDIS en redis.conf

bind 173.17.0.10 
modo protegido sin 
puerto 6379 
pidfile /var/run/redis_6379.pid 
clúster habilitado sí 
cluster-config-file nodes-6379.conf 
cluster-node-timeout 5000 
appendonly no 
save "" 
tcp-backlog 20000 
maxclients 10000 
logfile "/var/log/redis/redis.log" 
#logfile "/var/log/redis/redis01-master/redis.log" 
repl-diskless-sync sí 
repl-diskless-sync-delay 3


<h4>Inicializar el clúster</h4>

Una vez que todos los contenedores estén en funcionamiento, conéctese a cualquiera de los nodos de REDIS y ejecute el siguiente comando para inicializar el clúster.

redis-cli --cluster crear 173.17.0.10:6379 173.17.0.40:6384 173.17.0.50:6382 173.17.0.60:6383 --cluster-replicas 1
