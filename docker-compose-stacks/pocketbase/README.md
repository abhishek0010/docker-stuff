# Build and deploy a Pocketbase project with Docker
---
## PocketBase

Pocketbase is an open source backend which is distributed in a single Go binary. This new tool is similar to Firebase.

This project is created to use SQLite as a database with a REST api. Some of the functions that this tool integrates are the following:

- Real time database (Powered by SQLite)
- File storage: integrates with S3, more object storage support is coming.
- Autenticación
- Full-featured admin UI - to manage users, collections and more
- Extensible via Go programming language to extend functionality
This tool is perfect for making simple backends for small projects, hobbies, practice, and more.

### Requirements
You must have knowledge of the following topics:

- Docker
- Web development
- Linux

### Step 1: Create an image with Docker
If you want to save time you can use my Docker image, for that you can skip to step 2.

The first thing will be to create an image with Docker, for this you must first have it installed, then we are going to open a terminal.

First we create a folder where we will store our project:
```
~$ mkdir pocketbase-docker 
~$ cd pocketbase-docker
```
Then we are going to create our Dockerfile so for this we can open our preferred editor, in my case vscode and already in it we are going to create the file and add the following:
```
FROM alpine:3 as downloader

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=0.2.8

ENV BUILDX_ARCH="${TARGETOS:-linux}_${TARGETARCH:-amd64}"



RUN wget https://github.com/pocketbase/pocketbase/releases/download/v${VERSION}/pocketbase_${VERSION}_${BUILDX_ARCH}.zip \
    && unzip pocketbase_${VERSION}_${BUILDX_ARCH}.zip \
    && chmod +x /pocketbase

FROM alpine:3

# Dependencies
RUN apk update && apk add ca-certificates && rm -rf /var/cache/apk/*

EXPOSE 8090

COPY --from=downloader /pocketbase /usr/local/bin/pocketbase
CMD ["/usr/local/bin/pocketbase", "serve", "--http=0.0.0.0:8090", "--dir=/pb_data"]]
```

Ok, once we have this, let's go back to the terminal to create our Docker image:

# We must be in the terminal positioned in the folder where we have our Dockerfile
```
~$ docker build -t pocketbase .
```
After this we can test the image to verify that it works, you can execute the following command:

# You need to make sure you have the '-it' flag and expose the port
```
~$ docker run -it --rm -p 8090:8090 pocketbase
```
If this went well you can open your browser at localhost:8090/_/ and see the pocketbase admin panel.

Now that we verify that it works we are going to publish our image in our docker account, if you do not have one I recommend you create it and log in from your terminal with the following command:
```
~$ docker login -u <your-username> -p <your-password>
```
Now let's deploy our image:

# We must add a tag to our image before deploying, for simplicity I will leave it as 'latest'
```
~$ docker tag pocketbase:latest <your-docker-user>/pocketbase:latest
```
# Now let's roll out
```
~$ docker push <your-docker-user>/pocketbase:latest
```
With this you have your image uploaded in docker 😎
