# Docker-Nginx-Wordpress-MySQL

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)]()

## Description:
A Nginx HTTPS reverse proxy is an intermediary proxy service which takes a client request, passes it on to one or more servers, and subsequently delivers the server’s response back to the client. While most common applications are able to run as web server on their own, the Nginx web server is able to provide a number of advanced features such as load balancing, TLS/SSL capabilities and acceleration that most specialized applications lack. By using a Nginx reverse proxy, all applications can benefit from these features.

There are significant benefits to setting up a Nginx HTTPS reverse proxy:

    Increased Security: A Nginx reverse proxy also acts as a line of defense for your backend servers. Configuring a reverse proxy ensures that the identity of your backend servers remains unknown.
    Better Performance: Nginx has been known to perform better in delivering static content file and analyse URLs
    Easy Logging and Auditing: Since there is only one single point of access when a Nginx reverse proxy is implemented, this makes logging and auditing much simpler.
    Encrypted Connection By encrypting the connection between the client and the Nginx reverse Proxy with TLS, users profit from a encrypted and securized HTTPS connection, protecting their data.
    
Nginx-proxy sets up a container running nginx and docker-gen. docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped



## 1. Docker installation

```sh
amazon-linux-extras install docker -y
systemctl start docker.service
systemctl enable docker.service
```

### > Network creation

```sh
docker network create jomynetwork
```
### > create SSL Certificate files (selfsigned used here) OR you can use selfsigned site to generate the certificate

```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout jomygeorge.xyz.key -out jomygeorge.xyz.crt
```
### > Database container creation

```
docker container run --name db -d -e MYSQL_ROOT_PASSWORD="mysqlroot@123" -e MYSQL_DATABASE="wpdb" -e MYSQL_USER="wpuser" -e MYSQL_PASSWORD="wpass@123" --network jomynetwork mysql:5.6
```

> ###  Wordpress container creation

```
docker container run --name wordpress -d --network jomynetwork -e WORDPRESS_DB_HOST="db" -e WORDPRESS_DB_USER="wpuser"  -e WORDPRESS_DB_PASSWORD="wpass@123" -e WORDPRESS_DB_NAME="wpdb" wordpress
```


### > Create a custom "default.conf" for the Nginx reverse proxy image creation

```sh
server {
    listen 80;
    return 301 https://jomygeorge.xyz;
}

server {

    listen 443 ssl;
    server_name jomygeorge.xyz;

    ssl_certificate           /etc/nginx/jomygeorge.xyz.crt;
    ssl_certificate_key       /etc/nginx/jomygeorge.xyz.key;


    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
	
      proxy_pass              http://wordpress;        ###============ This is the name of wordpress container as its in the same network
      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;
      proxy_read_timeout  90;

    }
  }

  
  ```
b. Obtain SSL certificate for your domain and save private key in cert.key and the certificate in cert.crt


### > Create Dockerfile

```sh
FROM nginx
RUN rm -rf /etc/nginx/default.d/*
COPY ./default.conf /etc/nginx/conf.d/default.conf
COPY ./jomygeorge.xyz.cert.crt /etc/nginx/cert.crt
COPY ./jomygeorge.xyz.cert.key /etc/nginx/cert.key
```


### > Build the image of Nginx

```sh
docker image build -t nginx:v1 .
```


### > Nginx proxy container creation

```sh
docker container run -d --name proxy --network jomynetwork -p 80:80 -p 443:443 nginx:v1
```

### Conclusion

Here is a simple three container with wordpress, mysql and nginx as reverse proxy setup

#### ⚙️ Connect with Me

<p align="center">
<a href="mailto:jomyambattil@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/jomygeorge11"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a> 
<a href="https://www.instagram.com/therealjomy"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a><br />
