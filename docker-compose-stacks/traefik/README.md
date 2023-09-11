# Traefik Deployment

To Traefik Proxy in a Docker standalone scenario you must use a Docker Compose file. In the following docker-compose.yml you will find the configuration for Traefik with SSL support.

It needs a docker bridge network created manually

```shell
$ docker network create reverse_proxy_nw
```

Next step, is to gather apikeys from your dns provider. Place then within `.env` file with appropriate labels.
Refer : https://doc.traefik.io/traefik/https/acme/

Change the references for your dns provider within `docker-compose.yml` labels & `traefik.yml` configs.
You also need to enter your email address for Let's Encrypt registration.

```yaml
- --certificatesresolvers.leresolver.acme.email=your-email
```

Next, customize some labels in the Traefik container

```yaml
- "traefik.http.routers.<service>.rule=Host(`<sub-domain>.yourdomain.com`)"
- "traefik.http.routers.edge.rule=Host(`edge.yourdomain.com`)"
```

Once this is done, you're ready to deploy Portainer:

```shell
docker-compose up -d
```
