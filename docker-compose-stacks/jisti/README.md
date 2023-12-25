# Setting up Jisti meet with traefik reverse proxy

- Follow this link and setup the jisti dockerfile directory, but donet run `docker compose`
- Use `wget <link>` to download & `tar -xvzf <tar.gz file>` to extract in the current folder
- Within created .env file, comment out `HTTP_PORT`, `HTTPS_PORT` and change `TZ`, `PUBLIC_URL`
- Replace the docker file inside or make relevant changes
- Edit the traefik entrypoint settings to support :10000/udp connection. So the same with instance security group
- Add relevant entries to authelia to enable the selected subdomain for jisti
- Now you can deploy the stack using `docker compose up -d`
