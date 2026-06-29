services:
  db:
    image: mariadb:10.11
    container_name: nextcloud_db
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    networks:
      - nextcloud-net
    volumes:
      - /volume2/docker/nextcloud/db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}

  redis:
    image: redis:alpine
    container_name: nextcloud_redis
    restart: unless-stopped
    networks:
      - nextcloud-net
    command: ["redis-server", "--requirepass", "${REDIS_PASSWORD}", "--appendonly", "yes"]
    volumes:
      - /volume2/docker/nextcloud/redis:/data

  app:
    image: nextcloud:latest
    container_name: nextcloud_app
    restart: unless-stopped
    networks:
      - nextcloud-net
    ports:
      - 8080:80
    volumes:
      - /volume1/nextcloud/data:/data
      - /volume2/docker/nextcloud/config:/var/www/html/config
      - /volume2/docker/nextcloud/custom_apps:/var/www/html/custom_apps
      - /volume2/docker/nextcloud/themes:/var/www/html/themes
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=${TIMEZONE}
      - NEXTCLOUD_TRUSTED_DOMAINS=${NAS_IP_OR_DOMAIN}
      - NEXTCLOUD_DB_TYPE=mysql
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - NEXTCLOUD_ADMIN_USER=${NEXTCLOUD_ADMIN_USER}
      - NEXTCLOUD_ADMIN_PASSWORD=${NEXTCLOUD_ADMIN_PASSWORD}
      - REDIS_HOST=redis
      - REDIS_HOST_PASSWORD=${REDIS_PASSWORD}
      - NEXTCLOUD_DATADIR=/data
    depends_on:
      - db
      - redis

  cron:
    image: nextcloud:latest
    container_name: nextcloud_cron
    restart: unless-stopped
    networks:
      - nextcloud-net
    volumes:
      - /volume1/nextcloud/data:/data
      - /volume2/docker/nextcloud/config:/var/www/html/config
      - /volume2/docker/nextcloud/custom_apps:/var/www/html/custom_apps
    entrypoint: /cron.sh
    depends_on:
      - app

networks:
  nextcloud-net:
    driver: bridge

---

# Database Passwords
MYSQL_ROOT_PASSWORD=
MYSQL_PASSWORD=

# Nextcloud Admin
NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=

# Performance
REDIS_PASSWORD=

# Network Settings
NAS_IP_OR_DOMAIN=192.168.
TIMEZONE=Europe/London

---

sudo chown -R 33:33 /volume2/docker/nextcloud
sudo chown -R 33:33 /volume1/nextcloud/data

---
