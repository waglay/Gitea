# Gitea
Setting up self hosted github server

Gitea is a software which helps us to host our own github server for personal use or in a private way within an organization.

# Setup Gitea

It can be setup in two ways: either using the binary or using docker compose.
Docker Compose is used in this demonstration.

We can either choose to add a database or use sqlite for the setup, sqlite is used in this docs.

We can also have integrated actions or ci pipeline just like a github actions. In order to setup the gitea actions, we will need a runner. It can either be a seperate hosted server or another docker container, docker container is used in the docs.

## After considering all the things let's setup gitea first

We can always refer to the docs for more help: https://docs.gitea.com/category/installation

### This is the docker setup

- Simply using the image provided by gitea is enough to configure the gitea server. Below is the docker-compose.yml file:

  ```
  version: "3"
  networks:
    gitea:
      external: false
  services:
    server:
      image: docker.gitea.com/gitea:1.23.5
      container_name: gitea
      environment:
        - USER_UID=1000
        - USER_GID=1000
      restart: always
      networks:
        - gitea
      volumes:
        - ./gitea:/data
        - /etc/timezone:/etc/timezone:ro
        - /etc/localtime:/etc/localtime:ro
      ports:
        - "3000:3000"
        - "222:22"
  ```

  - After executing the ``` docker compose up -d ``` command the instance is up and running and we can start installing it.
  - We can configure the admin credentials while installation or after the instance is up, the first registered user is the admin.
 
    <img width="1680" alt="Screenshot 2025-03-24 at 12 16 02" src="https://github.com/user-attachments/assets/d41632d0-0db3-41b1-a988-5eb2423dbb8f" />
    *** this is the config after launching the gitea container ***

    <img width="1678" alt="Screenshot 2025-03-24 at 12 17 16" src="https://github.com/user-attachments/assets/b286cae7-0324-4151-9162-53907e1b2109" />
    *** changes can be made for the admin credentials from the hilighted part ***

  - Now clicking the install gitea option will setup our local github server.
