# Bloque para manejar tráfico HTTP (puerto 80)
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # Redirigir todo el tráfico HTTP a HTTPS
    server_name app.devapps.com.mx www.app.devapps.com.mx;

    # Forzar redirección a HTTPS
    location / {
        return 301 https://$host$request_uri;
    }
}

# Bloque para manejar tráfico HTTPS (puerto 443)
server {
    listen 443 ssl;
    listen [::]:443 ssl ipv6only=on;

    server_name app.devapps.com.mx www.app.devapps.com.mx;

    # Configuración de certificados SSL (por Certbot)
    ssl_certificate /etc/letsencrypt/live/app.devapps.com.mx/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.devapps.com.mx/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;  # Configuración estándar de Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;  # Seguridad adicional generada por Certbot

    # Configuración de la raíz y los archivos por defecto
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    # Manejo de rutas y archivos estáticos
    location / {
        try_files $uri $uri/ =404;  # Devuelve 404 si no encuentra el archivo
    }

    # Configuración para la API (ajusta las rutas a tus servicios locales)
    location /planka {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;

        rewrite ^/planka(/.*)$ $1 break;
    }

    location /planka/socket.io/ {
        proxy_pass http://localhost:3000/socket.io/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_redirect off;
        rewrite ^/planka/socket.io/(.*)$ /socket.io/$1 break;
    }

    location /erp/api/ {
        proxy_pass http://localhost:8001/erp/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
    }
}

# Bloque adicional para manejar tráfico HTTP y forzar HTTPS en el puerto 80 (ya está en el primer bloque, pero lo dejas por si acaso)
server {
    listen 80;
    listen [::]:80;

    server_name app.devapps.com.mx www.app.devapps.com.mx;

    # Redirigir todas las rutas HTTP a HTTPS
    location / {
        return 301 https://$host$request_uri;
    }
}
