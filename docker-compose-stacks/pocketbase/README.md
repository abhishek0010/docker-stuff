Build and deploy a Pocketbase project with Docker and Fly.io
#
webdev
#
backend
#
tutorial
#
docker
PocketBase
Today I will talk about pocketbase, and how to develop a backend with this tool.
Pocketbase is an open source backend which is distributed in a single Go binary. This new tool is similar to Firebase.

This project is created to use SQLite as a database with a REST api. Some of the functions that this tool integrates are the following:

Real time database (Powered by SQLite)
File storage: integrates with S3, more object storage support is coming.
Autenticación
Full-featured admin UI - to manage users, collections and more
Extensible via Go programming language to extend functionality
This tool is perfect for making simple backends for small projects, hobbies, practice, and more.

Requirements
You must have knowledge of the following topics:

Docker
Web development
Linux
Step 1: Create an image with Docker
If you want to save time you can use my Docker image, for that you can skip to step 2.

The first thing will be to create an image with Docker, for this you must first have it installed, then we are going to open a terminal.

First we create a folder where we will store our project:

~$ mkdir pocketbase-docker 
~$ cd pocketbase-docker
Then we are going to create our Dockerfile so for this we can open our preferred editor, in my case vscode and already in it we are going to create the file and add the following:

FROM alpine:3 as downloader

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=0.2.8

ENV BUILDX_ARCH="${TARGETOS:-linux}_${TARGETARCH:-amd64}"

# Install the dependencies
RUN apk add --no-cache \
    ca-certificates \
    unzip \
    wget \
    zip \
    zlib-dev

RUN wget https://github.com/pocketbase/pocketbase/releases/download/v${VERSION}/pocketbase_${VERSION}_${BUILDX_ARCH}.zip \
    && unzip pocketbase_${VERSION}_${BUILDX_ARCH}.zip \
    && chmod +x /pocketbase

FROM scratch

EXPOSE 8090

COPY --from=downloader /pocketbase /usr/local/bin/pocketbase
CMD ["/usr/local/bin/pocketbase", "serve", "--http=0.0.0.0:8090"]
Ok, once we have this, let's go back to the terminal to create our Docker image:

# We must be in the terminal positioned in the folder where we have our Dockerfile
~$ docker build -t pocketbase .
After this we can test the image to verify that it works, you can execute the following command:

# You need to make sure you have the '-it' flag and expose the port
~$ docker run -it --rm -p 8090:8090 pocketbase
If this went well you can open your browser at localhost:8090/_/ and see the pocketbase admin panel.

Now that we verify that it works we are going to publish our image in our docker account, if you do not have one I recommend you create it and log in from your terminal with the following command:

~$ docker login -u <your-username> -p <your-password>
Now let's deploy our image:

# We must add a tag to our image before deploying, for simplicity I will leave it as 'latest'
~$ docker tag pocketbase:latest <your-docker-user>/pocketbase:latest
# Now let's roll out
~$ docker push <your-docker-user>/pocketbase:latest
With this you have your image uploaded in docker 😎

Step 2: Deploy our Docker image to fly.io
For this, the first thing you need is to install fly.io in our operating system, then I leave you the commands for each operating system:

# Windows:
~$ iwr https://fly.io/install.ps1 -useb | iex
# Linux
~$ curl -L https://fly.io/install.sh | sh
# Mac
~$ brew install flyctl
# or
~$ curl -L https://fly.io/install.sh | sh
Later you will need to sign up, for this, you can execute the following command in the terminal:

~$ flyctl auth signup
This command will open a window in the browser where you can register and then it will ask you to enter a payment method, don't worry, Fly will not charge you unless your application exceeds the established free limit, and for the purposes of this tutorial it will help you to perfection.

Now you need to log in:

~$ flyctl auth login
Well, once you have logged in we are going to proceed to deploy:

For this we are going to need our Docker image that we deployed.
First we must create a necessary file to deploy, but in my case I will let Fly create it for me, for this in the terminal we execute the following command:

~$ flyctl launch --image <your-docker-user>/pocketbase:latest
# if you decided to use my image you can copy the following command:
~$ flyctl launch --image marcoa16b/pocketbase:latest
At this point fly will ask us a couple of questions, we will give our project a name and select a region for our server.

Very well, with this we will have our fly.toml file, we must damage some modifications to this file, since we need to expose the port of the Docker container so that we can access it from the domain that fly creates for us, for this we modify the following:

...
[experimental]
  allowed_public_ports = [8090]
  ...
...
[[services]]
  ...
  internal_port = 8090
  [[services.ports]]
    handlers = ["http"]
    port = 8090
...
The rest you can leave as is.

Once you have modified these ports, you can run flyctl deploy and if you get the following output, everything is fine:

 1 desired, 1 placed, 1 healthy, 0 unhealthy [health checks: 1 total, 1 passing]
--> v0 deployed successfully
Now you can go to the fly.io page and verify that you have your project there, when you open it you have the link of your project and the first time you open it in the address yourproject.fly.dev/_/ you will be able to create your user administrator and login to start managing your backend.

In the same pocketbase administrator panel you can find a simple but very complete documentation to be able to use this tool in your Javascript projects.
