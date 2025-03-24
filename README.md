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
    *** changes can be made for the admin credentials from the highlighted part ***

  - Now clicking the install gitea option will setup our local github server.

  - Now own organizations and repos can be created inside the launched instance.
 
    <img width="1680" alt="Screenshot 2025-03-24 at 12 23 40" src="https://github.com/user-attachments/assets/c55671be-4713-40d3-996d-2f17ef7a0cb1" />

    *** Home Page after launch ***

    <img width="1680" alt="Screenshot 2025-03-24 at 12 24 57" src="https://github.com/user-attachments/assets/4d68bc30-5e7e-4ea6-9c0e-1f209aeedfea" />

    *** creating a new organization, provide the name and make sure to manage the visibility ***

    <img width="1679" alt="Screenshot 2025-03-24 at 12 26 05" src="https://github.com/user-attachments/assets/f01a566d-9499-45ac-a0b1-d2bf61695d93" />

  - Create a new repo inside the organization

    <img width="1680" alt="Screenshot 2025-03-24 at 12 26 43" src="https://github.com/user-attachments/assets/fd4c009f-7cb4-4474-b4a0-5a7011aa3ab7" />

    *** set the name and required visibility ***

    <img width="1678" alt="Screenshot 2025-03-24 at 12 28 25" src="https://github.com/user-attachments/assets/a93f40b4-7c41-4673-b7e9-9b312934a9ca" />

  - Clone the repo add your files and push it to the repo just like in github.
 
## Setup Gitea Actions

- Inorder to setup the actions we need to have a runner running and github actions enabled. By default the actions are enabled, if not we can enable it by navigating to ```repo's settings```
  <img width="1680" alt="Screenshot 2025-03-24 at 12 34 35" src="https://github.com/user-attachments/assets/53ff91c3-2617-41c5-ae64-565acb0b5449" />

  *** scrolling down reveals the checkbox to enable actions on repository. ***

  <img width="1680" alt="Screenshot 2025-03-24 at 12 36 20" src="https://github.com/user-attachments/assets/749bee17-3397-4f0d-bd71-020db177df1b" />

- Now we need a registration token to register the container as a runner to the gitea container. For that navigate to ``` site administration ```

  <img width="1680" alt="Screenshot 2025-03-24 at 12 31 27" src="https://github.com/user-attachments/assets/d9ce024a-2e47-428a-97f5-4b2d4826c354" />

  *** it can be accessed by clicking on the drop down on your avatar on top right hand side ***

- Now navigate to ``` actions -> runners-> create a new runner ```
  <img width="1680" alt="Screenshot 2025-03-24 at 12 38 50" src="https://github.com/user-attachments/assets/0264a591-c323-4396-9b62-69aab5196b89" />

- Copy the token and now you are ready to set up the docker-compose.yml file for the runner;
  ```
  runner:
    image: docker.io/gitea/act_runner:nightly
    environment:
      GITEA_INSTANCE_URL: "http://gitea:3000"
      GITEA_RUNNER_REGISTRATION_TOKEN: "QF9bSSRXa9BGMCCc8HRHewWdxg4SsGdVab377Qjh"
      GITEA_RUNNER_NAME: "runner-1"
    volumes:
      - ./data:/data
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitea
  ```

- This along with the previous configuration adds up as;

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
    runner:
      image: docker.io/gitea/act_runner:nightly
      environment:
        GITEA_INSTANCE_URL: "<gitea server url>"
        GITEA_RUNNER_REGISTRATION_TOKEN: "<registration token from before>"
        GITEA_RUNNER_NAME: "<runner name>"
      volumes:
        - ./data:/data
        - /var/run/docker.sock:/var/run/docker.sock
      networks:
        - gitea
  ```

  - Here keep in mind the gitea server must run first and only we can access the token.
  - The gitea server url must either be the name of the service that gitea is running, for instance: http://server:3000 , which is the name of the compose service or the name of the container, for instance: http://        gitea:3000 as per the docker-compose file. Or it can be the private ip address of your host machine.
 
  - For the actions to work, the repo needs to have a directory structure as ```.gitea/workflows``` where we can add a yml file with the pipeline actions.
 
    <img width="1630" alt="Screenshot 2025-03-24 at 12 57 31" src="https://github.com/user-attachments/assets/2d63c2b6-8308-47d3-9f53-f5eea6311b7e" />

  - Sample actions:
 
    ```
    name: Gitea Actions Demo
    run-name: ${{ gitea.actor }} is testing out Gitea Actions 🚀
    on: [push]
    
    jobs:
      Explore-Gitea-Actions:
        runs-on: ubuntu-latest
        steps:
          - run: echo "🎉 The job was automatically triggered by a ${{ gitea.event_name }} event."
          - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by Gitea!"
          - run: echo "🔎 The name of your branch is ${{ gitea.ref }} and your repository is ${{ gitea.repository }}."
          - name: Check out repository code
            uses: actions/checkout@v4
          - run: echo "💡 The ${{ gitea.repository }} repository has been cloned to the runner."
          - run: echo "🖥️ The workflow is now ready to test your code on the runner."
          - name: List files in the repository
            run: |
              ls ${{ gitea.workspace }}
          - run: echo "🍏 This job's status is ${{ job.status }}."
    ```
  - This action is triggered on push to the remote repo.
 
  - After commiting and pushing the changes, the actions must be triggered.
    <img width="1677" alt="Screenshot 2025-03-24 at 13 04 50" src="https://github.com/user-attachments/assets/8d11459f-0ffa-4ee4-afa1-1ba56c26f687" />

  - But the job didnot succeed as it sent the following error:
    <img width="1679" alt="Screenshot 2025-03-24 at 13 05 28" src="https://github.com/user-attachments/assets/164e4fd6-a11f-4821-8805-f0ff5fd1a2c2" />
  - The error is seen because the ``` runs-on: ubuntu-latest ``` will launch a new docker container which will be deleted after it's operation automatically and while doing so, the container is run on seperate
    network than the gitea and gitea runner so, we need to setup a custom configuration file called config.yaml, so to generate the file:

    ```
    docker run --entrypoint="" --rm -it docker.io/gitea/act_runner:latest act_runner generate-config > config.yaml
    ```
  - This command will generate a config.yaml file on our host and we can mount it to the volume of the container. So the config file becomes:

    ```
    runner:
      image: docker.io/gitea/act_runner:nightly
      environment:
        CONFIG_FILE: /config.yaml
        GITEA_INSTANCE_URL: "http://gitea:3000"
        GITEA_RUNNER_REGISTRATION_TOKEN: "zzTAk2hBVShVESnqZz4XW5l1ZVLNVv3zHop9yowX"
        GITEA_RUNNER_NAME: "runner-1"
      volumes:
        - ./config.yaml:/config.yaml
        - ./data:/data
        - /var/run/docker.sock:/var/run/docker.sock
      networks:
        - gitea
    ```

  - Here the line ``` CONFIG_FILE ``` under ``` environment ``` and ``` - ./config.yaml ``` on ``` volumes ``` is added .
  - Change the configuration of the config.yaml, navigate to the network part and add in the name of the network your gitea and gitea-runner are running on, for this docs it's gitea.
    <img width="1672" alt="Screenshot 2025-03-24 at 13 18 50" src="https://github.com/user-attachments/assets/b6aed6d1-255d-40ef-990b-35fb59c85221" />
  - Make sure to externally create the network named gitea. 
  - Now restart the runner.
  - This time it should work as expected and the setup is complete.

    <img width="1679" alt="Screenshot 2025-03-24 at 13 33 19" src="https://github.com/user-attachments/assets/e2372867-a2cf-4093-a392-85699bcfa65d" />

  - If it doesnot work while running try resetting up the whole process once again with the config.yaml file. The final compose file is:

    ```
    version: "3"
    networks:
      gitea:
        external: true
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
      runner:
        image: docker.io/gitea/act_runner:nightly
        environment:
          CONFIG_FILE: /config.yaml
          GITEA_INSTANCE_URL: "http://server:3000"
          GITEA_RUNNER_REGISTRATION_TOKEN: "LePucjxzediYIE4PlTquywxRsxy54dPcdW3NUqm8"
          GITEA_RUNNER_NAME: "runner-1"
        volumes:
          - ./config.yaml:/config.yaml
          - ./data:/data
          - /var/run/docker.sock:/var/run/docker.sock
        networks:
          - gitea
    ```    
 
  ### Note: There wouldnot have been any problem if the private ip of the host machine was used or any other publicly accessable domains. 

    
  



