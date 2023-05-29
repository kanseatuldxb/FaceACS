###################################################################################################################
service

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target
[Service]
User=jetson
Group=www-data
WorkingDirectory=/home/jetson/Module/Veridos
ExecStart=/home/jetson/.local/bin/gunicorn  --access-logfile - -k uvicorn.workers.UvicornWorker  --workers 3  --bind unix:/home/jetson/Module/Veridos/Veridos.sock  Veridos.asgi:application
[Install]
WantedBy=multi-user.target

###################################################################################################################
nginx/sites-available/Veridos

server {
  listen 80;
  server_name _;
  location = /favicon.ico { access_log off; log_not_found off; }
  location /static/ {
      root /home/jetson/Module/Veridos;
  }
  location / {
      include proxy_params;
      proxy_pass http://unix:/home/jetson/Module/Veridos/Veridos.sock;
  }
  location ~ /ws/ {
        proxy_pass http://unix:/home/jetson/Module/Veridos/Veridos.sock;
        proxy_http_version 1.1;
        proxy_redirect off;
        proxy_buffering off;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
###################################################################################################################
gunicornc=socket

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/home/jetson/Module/Veridos/Veridos.sock

[Install]
WantedBy=sockets.target

###################################################################################################################