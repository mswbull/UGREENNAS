    services:
      tailscale:
        container_name: tailscale
        image: tailscale/tailscale:latest
        restart: unless-stopped
        volumes:
          - ./tun:/dev/net/tun
          - ./lib:/var/lib
        environment:
          - TS_AUTH_KEY=
          - TS_STATE_DIR=/var/lib/tailscale
          - TS_ROUTES=192.168.
        network_mode: host
        privileged: true
