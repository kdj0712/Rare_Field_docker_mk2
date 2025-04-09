## GCP( Google-Clouds‐Platform )

### create vm instance and than connect SSH in GCP
```
@ new project with ['rare-field']
@ Compute Engine > VM instance with ['rare-field'] (basic) > Disk size : 50G, Choose all with Firewall
@ click 'SSH' with 'rare-field'
~$ sudo apt-get update && sudo apt install -y unzip docker-compose nginx certbot python3-certbot-nginx

~$ lscpu
~$ df -h
~$ sudo systemctl status nginx
```

### DNS management (need login) with extenal IP on vm instance GCP
```
@https://dns.gabia.com/ > get Domain 
> DNS Managerment 'rare-field.shop': Host - setup extenal IP with '@' and 'www' 

@ http://34.123.194.224:80/
```

### install Docker with containers in GCP
```
~$ sudo docker system prune

~$ wget -O Rare_Field_docker.zip https://github.com/kdj0712/Rare_Field_docker/archive/refs/heads/main.zip
~$ unzip Rare_Field_docker.zip
~$ unzip docker.zip -d docker_folder && cd ./docker_folder
~/docker_folder$ sudo docker-compose build --no-cache
~/docker_folder$ sudo docker-compose --project-name rare_field up -d

~$ sudo docker ps
~$ sudo docker exec -it rare_field_springboot_3.1.1_fastapi_1 bash
```

### start fastapi server in docker
```
~# ps aux | grep uvicorn 
~# kill -9 [PID]
~# apt-get update && apt install -y nano
~# cd /apps/fastapis/apps && nano .env
/apps/fastapis# cd ../ && git pull
/apps/fastapis# nohup uvicorn mainpage:app  --host 0.0.0.0 --port 8000 --workers 2 & 
/apps/fastapis# exit

outside_docker:~$ wget http://localhost:8000
```

### start springboots server in docker
```
~# ps aux | grep gradlew 
~# kill -9 [PID]
~# cd /apps/springboots && nano ./src/main/resources/application.properties
  ...
server.address=0.0.0.0
  ...
spring.datasource.url=jdbc:mysql://rare_field_db_mysql_8_1:3306/rarefield
  ...
remote.server.url=http://rare-field.shop:80/       
root.file.folder=/apps/fastapis/data/img   

/apps/springboots# chmod +x ./gradlew && nohup ./gradlew bootRun &
/apps/springboots# exit

outside_docker:~$ wget http://localhost:8080
```

### setup https certification and start nginx in GCP
<details>

<summary>sudo nano /etc/nginx/sites-available/default</summary>

    server {
        listen 80;
        server_name rare-field.shop www.rare-field.shop;
        #return 301 https://$server_name$request_uri; # Redirect all HTTP requests to HTTPS
        location / {
            proxy_pass http://localhost:8000; # Forward all requests to localhost:8000
            proxy_set_header Host $host; # Pass the current host and port
            proxy_set_header X-Real-IP $remote_addr; # Pass the client's real IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # Pass the real user IP read by the proxy or load balancer
            proxy_set_header X-Forwarded-Proto $scheme; # The protocol being used (http or https)
        }
    }

    server {
        # SSL configuration
        listen 443 ssl;
        server_name rare-field.shop www.rare-field.shop;

        ssl_certificate /etc/letsencrypt/live/rare-field.shop/fullchain.pem; # Certificate path
        ssl_certificate_key /etc/letsencrypt/live/rare-field.shop/privkey.pem; # Key path

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout 10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        location / {
            proxy_pass http://localhost:8080; # Forward requests to the WAS running in Docker
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

</details>

```
~$ sudo certbot --nginx -d rare-field.shop -d www.rare-field.shop
~$ sudo rm /etc/nginx/sites-available/default
~$ sudo nano /etc/nginx/sites-available/default
  ... 
~$ sudo nginx -t
~$ sudo systemctl restart nginx

@ https://rare-field.shop/
@ https://www.rare-field.shop/
@ http://www.rare-field.shop:80/
@ http://rare-field.shop:80/
```


### Restart docker
```
~# sudo docker stop rare_field_springboot_3.1.1_fastapi_1
~# sudo docker start rare_field_springboot_3.1.1_fastapi_1
```
