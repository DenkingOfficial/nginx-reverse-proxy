# NGINX Reverse proxy

Small docker-compose.yml and nginx.conf files that will help you host different applications on multiple domains.

## How to use

1) There's a `nginx.conf` file, which you need to edit

```
events {
    worker_connections 1024;
}

http {
    # Change app name and container:port
    upstream my-first-app {
        server my-container-name:80;
    }

    server {
        listen 80;
        server_name localhost; # Change server name to your server

        location / {
            proxy_pass http://my-first-app; # Change app name if changed in line 7
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

Set your `my-first-app`, `my-container-name` and change server_name from `localhost` to server name you prefer to host app on

If you want to host multiple apps, you can add more upstreams and servers in the config, for example:

```
events {
    worker_connections 1024;
}

http {
    upstream my-first-app {
        server my-first-container-name:80;
    }

    upstream my-second-app {
        server my-second-container-name:8080;
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://my-first-app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 80;
        server_name some-domain.com;

        location / {
            proxy_pass http://my-second-app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

2) Create a network for your apps:

`docker network create my-network`

You can use any name you want

3) Set a network in docker-compose.yml file:

```
version: "3.8"

services:
  nginx-reverse-proxy:
    image: nginx:latest
    container_name: "nginx-reverse-proxy"
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - my-network

networks:
  my-network:
    external: true
```

4) Run `docker compose up --build`

5) Connect all your apps to the network you created using

`docker network connect my-network my-container-name`

6) Your app/apps should be accessible on server names you provided