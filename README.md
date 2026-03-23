# gitlab-docker-compose
guide gitlab docker compse debian 13
# 1. Установка докер и докер компосе на дебиан 13
```bash
# установка нужного по
sudo apt update
sudo apt install ca-certificates curl gnupg
# Добавление gpg ключа Docker 
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://docker.com | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
# Настройка репозитория
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://docker.com \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Установка Docker и Docker Compose
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```