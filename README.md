# gitlab-docker-compose
guide gitlab docker compse debian 13
# 1. Install docker and docker compose in debian 13
```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
# Install package
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
# 2. Install gitlab
## 2.1 create directory
```bash
mkdir -p /opt/gitlab
touch .env
echo GITLAB_HOME=/opt/gitlab > /opt/gitlab/.env

```
## 2.2 create docker compose file and nginx
> If you are using SSH port 22, change it in your Dockerfile or in `/etc/ssh/sshd_config`
```bash
cd /opt/gitlab
nano docker-compose.yml
```
`docker-compose.yml`
Please change `gitlab.example.com` 
```yml
services:
  gitlab:
    image: gitlab/gitlab-ee:latest
    container_name: gitlab
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        # Add any other gitlab.rb configuration here, each on its own line
        external_url 'https://gitlab.example.com'
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
```
my docker compose
```yml
services:
  gitlab:
    image: gitlab/gitlab-ee:latest
    container_name: gitlab
    restart: always
    hostname: 'gitlab-nd.lord-mikrotik.ru' # Исправил опечатку gilab -> gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab-nd.lord-mikrotik.ru'
        # 1. Отключаем попытки GitLab самому выпустить сертификат
        letsencrypt['enable'] = false
        # 2. Говорим внутреннему Nginx слушать только HTTP (80 порт)
        nginx['listen_https'] = false
        nginx['listen_port'] = 80
    ports:
      # Пробрасываем только 80 порт наружу (внешний Nginx будет стучаться сюда)
      - '80:80'
      - '222:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
    shm_size: '256m'
```
# 2.3 Nginx 
Minimal conufig for add cert
```nginx
server {
    listen 80;
    server_name gitlab-nd.lord-mikrotik.ru;
    return 301 https://$server_name$request_uri; # Редирект на HTTPS
}
```
create link `ln -s /etc/nginx/sites-available/gitlab-nd.conf /etc/nginx/sites-enabled`
# 2.4 add let's encrypt
```bash
certbot --nginx -d gitlab-nd.lord-mikrotik.ru
```
Congig `gitlab-nd.conf`
```nginx
server {
    listen 80;
    server_name gitlab-nd.lord-mikrotik.ru;
    return 301 https://$server_name$request_uri; # Редирект на HTTPS
}

server {
    listen 443 ssl;
    server_name gitlab-nd.lord-mikrotik.ru;

    # Пути к вашим SSL сертификатам (Certbot их сделает сам)
    ssl_certificate /etc/letsencrypt/live/gitlab-nd.lord-mikrotik.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/gitlab-nd.lord-mikrotik.ru/privkey.pem;

    location / {
        proxy_pass http://192.168.100.6:80; # Тот самый порт из docker-compose
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        # Важно для больших пушей в Git (LFS, образы докера)
        client_max_body_size 512m;
    }
}
```
> To install the Community Edition, replace `ee` with `ce`.
# 3. Up docker compose
```bash
docker compose up -d
```
> pass root
> `docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password`
