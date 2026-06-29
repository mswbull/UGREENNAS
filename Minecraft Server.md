version: "latest"
services:
  minecraft:
    image: itzg/minecraft-bedrock-server
    environment:
      EULA: "TRUE"
      GAMEMODE: survival
      DIFFICULTY: normal
      VERSION: "latest"
    volumes:
      - ./data:/data
    network_mode: host
    restart: unless-stopped
