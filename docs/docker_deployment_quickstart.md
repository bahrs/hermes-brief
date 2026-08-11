based on the guide by Kovalev

```bash
mkdir -p ~/hermes-agent/.hermes
cd ~/hermes-agent
touch docker-compose.yml
#Содержимое файла:
services:
 hermes:
  build:
   context: https://github.con/NousResearch/hermes-agent.git#main
  image: nousearch/hermes-agent:latest
  container_name: hermes
  restart unless-stopped
  network_mode: host
  volumes:
   - ./.hermes:/opt/data
   - /var/run/docker/sock:/var/run/docker.sock
  environment:
   HERMES_UID: ${HERMES_UID:-10000}
   HERMES_GID: ${HERMES_GID:-10000}
  command: ["gateway", "run"]

#далее выполняем команды
HERMESUID=$(id -u) HERMES_GID=$(id -g) docker compose up -d —build
docker compose exec hermes hermes setup

#следуем инструкциям по установке
#После выполнения инструкций
docker compose restart
```
