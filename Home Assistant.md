    services:
      app:
        image: ugreen/home-assistant:v2
        volumes:
          - /volume1/homeassistant:/config
        environment:
          - TZ=Europe/London
        ports:
          - "8123:8123"
        restart: always
        network_mode: host
