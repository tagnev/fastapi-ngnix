"# fastapi-ngnix" 

Install Python & Other packages in Centos

https://computingforgeeks.com/how-to-install-python-3-on-centos/

Install GIT

https://www.digitalocean.com/community/tutorials/how-to-install-git-on-centos-7

Install NGNIX

https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-8

Unix Run UVICORN
uvicorn app:app --host="0.0.0.0" --port=8000

```GUNICORN```
 gunicorn -k uvicorn.workers.UvicornWorker --bind "0.0.0.0:8000" --log-level debug main:app
 
 NGNIX Steps:
 1. Create proxy_params in /etc/nginx. Below is the content
      proxy_set_header Host $http_host;
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header X-Forwarded-Proto $scheme;

 2.  nginx conf file
 
 ```     location / {
                include proxy_params;
                proxy_pass http://unix:/run/gunicorn.sock;
        }
```

3. Create gunicorn.service file in /etc/systemd/system
 Below is the content,
 [root@centos-s-1vcpu-1gb-sfo3-01 system]# cat gunicorn.service
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
WorkingDirectory=/var/www/project
ExecStart=/var/www/project/venv/bin/gunicorn \
          --access-logfile - \
          --workers 4 \
          -k uvicorn.workers.UvicornWorker \
          -b unix:/var/www/project/gunicorn.sock \
          -m 007 \
          --timeout 1200 \
          main:app

[Install]
WantedBy=multi-user.target

4. create gunicorn.conf under /etc/nginx/conf.d

server {
listen 8000;
server_name <ip address>;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_pass http://unix:/run/gunicorn.sock;
        }
}

