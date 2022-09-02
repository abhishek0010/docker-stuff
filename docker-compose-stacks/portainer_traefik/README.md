# Deploying in a Docker Standalone scenario
To deploy Portainer behind Traefik Proxy in a Docker standalone scenario you must use a Docker Compose file. In the following docker-compose.yml you will find the configuration for Portainer Traefik with SSL support and the Portainer Server.
```yaml
version: "3.3"

services:
  traefik:
    container_name: traefik
    image: "traefik:latest"
    command:
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --providers.docker
      - --log.level=ERROR
      - --certificatesresolvers.leresolver.acme.httpchallenge=true
      - --certificatesresolvers.leresolver.acme.email=your-email #Set your email address here, is for the generation of SSL certificates with Let's Encrypt. 
      - --certificatesresolvers.leresolver.acme.storage=/ssl_certs/acme.json
      - --certificatesresolvers.leresolver.acme.httpchallenge.entrypoint=web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "traefik_ssl_certs:/ssl_certs"
    labels:
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"

  portainer:
    image: portainer/portainer-ce:latest
    command: -H unix:///var/run/docker.sock
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    labels:
      # Frontend
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`portainer.yourdomain.com`)"
      - "traefik.http.routers.frontend.entrypoints=websecure"
      - "traefik.http.services.frontend.loadbalancer.server.port=9000"
      - "traefik.http.routers.frontend.service=frontend"
      - "traefik.http.routers.frontend.tls.certresolver=leresolver"

      # Edge
      - "traefik.http.routers.edge.rule=Host(`edge.yourdomain.com`)"
      - "traefik.http.routers.edge.entrypoints=websecure"
      - "traefik.http.services.edge.loadbalancer.server.port=8000"
      - "traefik.http.routers.edge.service=edge"
      - "traefik.http.routers.edge.tls.certresolver=leresolver"


volumes:
  portainer_data:
    name: portainer_data
  traefik-ssl-certs:
    name: traefik_ssl_certs
```

I have created a volume so that certs dont get discarded
In the volumes and command section of the Traefik Proxy container:
```yaml
- "traefik_ssl_certs:/ssl_certs"
- --certificatesresolvers.leresolver.acme.storage=/ssl_certs/acme.json
```
You also need to enter your email address for Let's Encrypt registration.
```yaml
- --certificatesresolvers.leresolver.acme.email=your-email
```
Next, customize some labels in the Traefik container. The following labels need to be updated with the URL that you want use to access Portainer:
```yaml
- "traefik.http.routers.frontend.rule=Host(`portainer.yourdomain.com`)"
- "traefik.http.routers.edge.rule=Host(`edge.yourdomain.com`)"
```
Once this is done, you're ready to deploy Portainer:
```shell
docker-compose up -d
```
After the images have been downloaded and deployed you will able to access Portainer from the URL you defined earlier, for example: https://portainer.yourdomain.com.

Refer https://docs.portainer.io/advanced/reverse-proxy/traefik
