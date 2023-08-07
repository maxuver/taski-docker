# taski-docker

external nginx file with [kittygram_final app](https://github.com/maxuver/kittygram_final) which should be placed on the virtual server in the directory:  

```sudo nano /etc/nginx/sites-enabled/default```:

```
server {
    server_name 158.160.76.223 patsyuk.servemp3.com;
    server_tokens off;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:8000;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/patsyuk.servemp3.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/patsyuk.servemp3.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    server_name 158.160.76.223 kittygram.myvnc.com;
    server_tokens off;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/kittygram.myvnc.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/kittygram.myvnc.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = kittygram.myvnc.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name 158.160.76.223 kittygram.myvnc.com;
    return 404; # managed by Certbot
}

server {
    if ($host = patsyuk.servemp3.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name 158.160.76.223 patsyuk.servemp3.com;
    return 404; # managed by Certbot
}
```

On your remote server in the root directory of the project you should also place the ```docker-compose.production.yml``` :


```
version: '3'

volumes:
  pg_data_production:
  static_volume:

services:
  db:
    image: postgres:13.10
    env_file: .env
    volumes:
      - pg_data_production:/var/lib/postgresql/data
  backend:
    image: maxuver/taski_backend # Docker Hub username / username on Docker image
    env_file: .env
    volumes:
      - static_volume:/backend_static
  frontend:
    image: maxuver/taski_frontend  # Download from Docker Hub
    env_file: .env
    command: cp -r /app/build/. /frontend_static/
    volumes:
      - static_volume:/frontend_static
  gateway:
    image: maxuver/taski_gateway  # Download from с Docker Hub
    env_file: .env
    volumes:
      - static_volume:/staticfiles/
    ports:
      - 8000:80
```
