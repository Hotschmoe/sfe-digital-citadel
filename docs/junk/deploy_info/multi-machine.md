version: "3"
services:
  traefik:
    image: traefik:latest
    container_name: traefik_prod
    ports:
      - "80:80"
      - "443:443"
    command:
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--providers.docker"
      - "--certificatesresolvers.myresolver.acme.email=you@email.com"
      - "--certificatesresolvers.myresolver.acme.storage=/acme.json"
      - "--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "./acme.json:/acme.json"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.http-catchall.rule=HostRegexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"

  # Prod container (e.g., Nextcloud)
  nextcloud:
    image: nextcloud:aio
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextcloud.rule=Host(`nextcloud.surname.com`)"
      - "traefik.http.routers.nextcloud.entrypoints=websecure"
      - "traefik.http.routers.nextcloud.tls.certresolver=myresolver"
      - "traefik.http.services.nextcloud.loadbalancer.server.port=80"

  # Dev forwarding to Server B
  dev-proxy:
    image: traefik:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dev.rule=HostRegexp(`{host:.+\\.dev\\.surname\\.com}`)"
      - "traefik.http.routers.dev.entrypoints=websecure"
      - "traefik.http.routers.dev.tls.certresolver=myresolver"
      - "traefik.http.services.dev.loadbalancer.server.url=https://192.168.1.11:443"

  # LLM forwarding to Server C
  llm-proxy:
    image: traefik:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.llm.rule=Host(`llm.surname.com`)"
      - "traefik.http.routers.llm.entrypoints=websecure"
      - "traefik.http.routers.llm.tls.certresolver=myresolver"
      - "traefik.http.services.llm.loadbalancer.server.url=http://192.168.1.12:5000"