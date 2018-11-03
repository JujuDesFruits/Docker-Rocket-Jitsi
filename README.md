# Presentation
This documentation is here to explain how to get rocketchat and jitsi on your own server using docker container.
[RocketChat](https://rocket.chat/) is a free and opensource communication tool where you can chat and share documents.
[Jitsi](https://jitsi.org/jitsi-meet/) is a free video conferencing completing RocketChat.

here is an image illstrating how Jitsi work using docker-compose
![alt text](https://335wvf48o1332cksy23mw1pj-wpengine.netdna-ssl.com/wp-content/uploads/2018/07/docker-jitsi-meet.png)


In RocketChat, docker-compose link rockerchat website to mongo database, and in the same time it link rockerchat website to hubot who is a chatbot programmable.

This project use official github repositories who use MIT Licence (mean there are open sources, alterable and free)

# Installation
you will find the command lines to install your Rocket.Chat & jitsi-meet on your machine.

## Securing the server
for CentOs:
```
sudo yum install epel-release -y
sudo yum install --enablerepo="epel" ufw -y
```
for other:
```
sudo apt-get install ufw
```
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
```
sudo ufw allow 22/tcp     # allow ssh
sudo ufw allow 443/tcp    # allow https
sudo ufw allow 3000/tcp   # allow rocketchat
sudo ufw allow 8080/tcp   # allow rocketchat-hubot
sudo ufw allow 80/tcp     # allow http
sudo ufw allow 10000/udp  # allow jitsi video brige
```
```
sudo ufw enable
sudo ufw reload
```
```
yum install fail2ban
// or
apt-get install fail2ban
```

(The SSL key are automatically created with the jitsy docker-compose file, it's a self signed certificate generated)

## Docker & docker-compose
to get docker I invite you to follow this guide
[get docker](https://docs.docker.com/cs-engine/1.13/)

to get docker-compose, just copy this two line:
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Git repositories
to install git command just type
```
yum install git
// or
apt-get install git
```

then we will download rocketchat and jitsi:
```
git clone https://github.com/RocketChat/Rocket.Chat.git
git clone https://github.com/jitsi/docker-jitsi-meet.git
```

## Set up of docker container
go to your file docker-jitsi-meet
```
docker-compose up -d
```
do the same thing in your file Rocket.Chat
at the end you should see that every container run well:
```
[root@localhost Rocket.Chat]# docker ps
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS                                      NAMES
21d0f410d46d        rocketchat/hubot-rocketchat:latest   "/bin/sh -c 'node -e…"   2 days ago          Up 12 seconds       0.0.0.0:3001->8080/tcp                     rocketchat_hubot_1
3dcaefe39753        rocketchat/rocket.chat:latest        "node main.js"           2 days ago          Up 2 days           0.0.0.0:3000->3000/tcp                     rocketchat_rocketchat_1
aca31bf1664e        mongo:3.2                            "docker-entrypoint.s…"   2 days ago          Up 2 days           27017/tcp                                  rocketchat_mongo_1
c6d672f2d3de        jitsi/jicofo                         "/init"                  2 days ago          Up 2 days                                                      docker-jitsi-meet_jicofo_1
430f5b9b44a6        jitsi/jvb                            "/init"                  2 days ago          Up 2 days           0.0.0.0:10000->10000/udp                   docker-jitsi-meet_jvb_1
df697ec213dd        jitsi/prosody                        "/init"                  2 days ago          Up 2 days           5222/tcp, 5269/tcp, 5280/tcp, 5347/tcp     docker-jitsi-meet_prosody_1
210dbc49bec0        jitsi/web                            "/init"                  2 days ago          Up 2 days           0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   docker-jitsi-meet_web_1
```

## Final touch
go to your domain name or your ip address of the machine on port 3000
"https://192.168.56.101:3000"
you will get access to your rocket server and you need to add personal information.
Then with administrator account go to Administation -> SETTING -> Video Conference
then set enable to true and complet the text field of jitsi url with your domain name or ip address on port 443

That's it ! Enjoy !

# Container
here is a a short description of each docker image used in this project

## rocketchat
We get a container using the last image of rocket.chat, and this container is in charge of the website. He depends on mongo image to save his data. His Default port is 3000

## mongo
mongo is an noSQL database. Here it is used to save data from RocketChat website. He will use an internal port 27017 to communicate with rocketchat.
You acces it using this command line:
```
docker exec -it rocketchat_mongo_1 /bin/bash
mongo
```

## hubot
hubot is an image to manage the chat bot in rocketchat. he use internal port 3001 to communicate with rocketchat.

## web
jitsi implement an image web create a website. This service use port 80 and 443 to get http and https access.

## jicofo
Conference focus is mandatory component of Jitsi Meet conferencing system next to the videobridge. It is responsible for managing media sessions between each of the participants and the videobridge.

## jvb
Jvb is a videobrideo allowing videoconferencing for jitsi. It work using port 10000 in udp.

## prosody
This service is in charge of the XMPP server (provides basic messaging, presence and XML routing). It use interal port as 5222, 5347, 5280 to link with the other jitsi services

# Update & Save
to update any container image just type these command line:
```
docker pull rocketchat/rocket.chat:develop #for example
docker-compose stop rocketchat
docker-compose rm rocketchat
docker-compose up -d rockerchat
```

to save a state of a container trhougth an image use these command:
(we gonna get rocketchat_rocketchat_1 as example)
```
[root@localhost Rocket.Chat]# docker ps
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS                          PORTS                    NAMES
57b50e09f495        rocketchat/rocket.chat:latest        "node main.js"           11 minutes ago      Restarting (1) 56 seconds ago                            rocketchat_rocketchat_1
```
```
docker commit 57b50e09f495 template-rocket
```
then you can found your docker image and use this image to buid a container:
```
[root@localhost Rocket.Chat]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
template-rocket               latest              0d365aaccf85        About a minute ago  300MB
```
