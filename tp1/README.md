# Docker et Docker Compose


Installation sur macOS

```bash
  arch -arm64 brew install 'docker'
  arch -arm64 brew install 'docker-compose'
```

## TP

The first command to pull the docker image is $
```bash
docker pull 'nginx'
```

To see if the image was goodly pulled we can execute the following command
```bash
docker image ls
```

To create the index file we can use the command `touch` in bash followed by the file name.
ex:
```bash
touch 'index.html'
```

To create a volume for docker we use
```bash
docker volume create 'volume_name'
```

And for we can use the '-v' option to link this volume to a local folder at the start of the container.
```bash
docker run -v "{pwd}":/usr/share/nginx/html nginx
```
The first part is the folder you want to bind with you container, the second line is the path were
the local folder will be bind.
I have had option to see the display of my index my command is :
```bash
docker run -d -p '80:80' -v '"${PWD}":/usr/share/nginx/html' --name tp_docker 'nginx'
```
-d : is to detach the container from the terminal after running,  
-p : is to bind a local with a container port in this case 80 (nginx service),  
-v : is to bind local folder with a space in your container,  
--name : is simply to give a name at our container,  
nginx : is the image pulled before.  

<br>

To delete the container without stop it we can use the following command : 
```bash
docker rm 'tp_docker' --force
```

To copy file in a container you can use : 
```bash
docker cp 'local_path' 'tp_docker':'container_path'
or :
docker cp 'tp_docker':'container_path' 'local_path'
ex :
docker cp 'index.html' 'tp_docker':'/usr/share/nginx/html/index.html'
```

Content of my dockerfile :
```Dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

**The advantages** of the dockerfile is that you can personalize a basic image like here, add files modify... you can give some parameter at the container. You can run some command in a container through the lunching with a dockerfile. And by the way reduce the number of argument of your command line for run a container. And the last point is for the automation process, like testing building and deployment.

**The disadvantage** for me is just the fact that at on every change of the dockerfile you must build a new image.


To download the mysql:7.5 on macOS M1 we must specify a platform to emulate the container with the good architecture because this mysql version doesn't match with the M1 chip.
```bash
docker pull mysql:5.7
```

For the phpmyadmin image the command is :
```bash
docker pull phpmyadmin/phpmyadmin
```
We run our container with the environment variable required (see the [mysql-doc](https://hub.docker.com/_/mysql) and [phpmyadmin-doc](https://docs.phpmyadmin.net/fr/latest/setup.html#docker-examples)) :

(I use latest version because the are conflicts between version and M1 chip on Mac)

```bash
docker run -d --name 'dbmysql' -e 'MYSQL_ROOT_PASSWORD=azerty' -p  'mysql'

docker run -d --name 'php' -p '80:80' -e 'PMA_HOST=IP-DB' 'phpmyadmin/phpmyadmin'
```

-e : is to set an env variable
MYSQL_ROOT_PASSWORD : DB paswword
PMA_HOSTS: Define the ip of the db (in our case the container).  

Here our credential will be id:root & pw:azerty.

For the part docker compose :
```yml
version: '3.1'

services:
  db:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - 33060:33060

  phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 8080:80
    environment:
      - PMA_ARBITRARY=1
```

***The advantages*** of the docker compose is that you can configure all variable networks containers volumes,
and that so simply. That simplify the command line because instead -d -p -e etc... you just have to enter the command docker compose up

***The disadvantage*** I don't see disadvantages.

8)b. We can set theses variables by the environment argument in the service configuration.

To create different network inside docker compose we can use networks :
```yaml
version: '3.1'

services:
  web:
    image: praqma/network-multitool:latest
    ports:
      - "80:80"
    expose:
      - "80"
    networks:
      - frontend
  app:
    image: praqma/network-multitool:latest
    ports:
      - "1180:1180"
    expose:
      - "1180"
    networks:
      - backend
  db:
    image: praqma/network-multitool:latest
    ports:
      - "11443:11443"
    expose:
      - "11443"
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

by the network frontend and backend the different services will be on different subnet.
to verify that is right we can use 
```bash
docker inspect 'our container name'
```
And check the line IPAddress
or do a command like :
```bash
docker inspect 'container1' 'container2' 'container3' | grep "IPAddress"
````

And compare the result like this :
```bash
☁  q9  docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED              STATUS              PORTS                                                 NAMES
2373665957ef   praqma/network-multitool:latest   "/bin/sh /docker/ent…"   About a minute ago   Up About a minute   443/tcp, 1180/tcp, 0.0.0.0:80->80/tcp, 11443/tcp      q9-web-1
b78eeea7750c   praqma/network-multitool:latest   "/bin/sh /docker/ent…"   About a minute ago   Up About a minute   80/tcp, 443/tcp, 11443/tcp, 0.0.0.0:1180->1180/tcp    q9-app-1
0641a5639eaf   praqma/network-multitool:latest   "/bin/sh /docker/ent…"   About a minute ago   Up About a minute   80/tcp, 443/tcp, 1180/tcp, 0.0.0.0:11443->11443/tcp   q9-db-1
☁  q9  docker inspect 2373665957ef b78eeea7750c 0641a5639eaf  | grep "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.22.0.2",
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.21.0.2",
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.21.0.3",
```

To the end this networks configuration could be use to deploy an application with secure rules to isolate or db for example of specify only frontend container
can interact with the backend etc...