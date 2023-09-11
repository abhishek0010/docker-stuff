### for user_database.yml password digest generation
````shell
$ docker run authelia/authelia:latest authelia crypto hash generate argon2 --password 'password'
````
### for configuration.yml client.client.secret digest generation
````shell
$ docker run authelia/authelia:latest authelia crypto hash generate pbkdf2 --variant sha512 --random --random.length 72 --random.charset rfc3986
````
### for public and private key generation
````shell
$ docker run -u "$(id -u):$(id -g)" -v "$(pwd)":/keys authelia/authelia:latest authelia crypto pair rsa generate --bits 4096 --directory /keys
````
### for everything else
````shell
$ docker run authelia/authelia:latest authelia crypto rand --length 128 --charset alphanumeric
````
