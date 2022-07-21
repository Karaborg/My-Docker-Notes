## Setup
First thing first, install docker extention to your IDE. It is **not a must** but, will help. To get started with docker, just open a new file and name it `Dockerfile`.


## Getting Started
Imagine you want to run a **node js** web application with docker. As you many know, the first thing you need to do before starting the application is, getting the project root folder and installing the configurations and packages, which you do with `npm install` command and then you can start the app with `npm start`. So just like that, we will be including all these steps inside of our docker file.

Example:
```
FROM node:16

WORKDIR /app

COPY . /app

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

To explain each command:
- `FROM node:16`, it means the app is **node** application like we mentioned before and we want to use verison **16** of node. The version is not mandatory, you can also say `FROM node`.

- `WORKDIR /app`, it sets the directory folder name as `/app` for our application for the image. And **all the commands will be executed under this directory**.

- `COPY . /app`, this is a basic copy command. It means it will copy the local root folder to our new directory folder named `/app` we just created in previous line.

- `RUN npm install`, this is for downloading the **node-modules**. `RUN` command is just runs the command following it. So, since we already coppied our project, it is time for downloading the configurations and packages.

- `COPY . .`, same as before, it is just a copy command. First dot is our local root folder, and the second dot is our directory path we mentioned with `WORKDIR` command.

- `EXPOSE 3000`, since we are starting a web application, we need an ip and a port to access this application. So this line exposes us a port for our local. **Not a must** because, we will clerify the port when we are starting the container again.

- `CMD ["npm", "start"]`, `npm start` is the way we start the node application. But if we just say `RUN npm start`, it would start the application on every image build. We do not want that. Instead, we will be using `CMD` command with it's different syntax.

After we are finished with our docker file, it is time for build an image.

Build the docker image (Dot is for our root folder. If your docker file is located somewhere else, you should set the path as where the docker file is):
```
$ docker build .
```

The command above will give you a **image id**. You will need that later. 

After creating our image, it is time for running the container.
> You run the container, not the image. The container runs based on image.

```
$ docker run -p <portYouWantToExposeForLocal>:<theExposedPortYouMentionedOnDockerFileOrAppHas> <imageId>
```

Check running containers (Only running containers):
```
$ docker ps
```

Check all containers (Containers that running and also the ones stopped at some point):
```
$ docker ps -a
```

Stop container:
```
$ docker stop <containerName>
```


## Also
Since each container react as a different machine, it is possible to run a software even if you do not have it on your local computer. So let's run **node**.
```
$ docker run -it node
```

With this command, the container will run node. It does not matter if the container already has node, if you have not download it before, docker will download it itself. The `-i` and `-t` flags stands for **interactive** and **terminal**. It says this container is **user interactive** and it also needs **terminal**.

If you do any changes on your projects, you will need to re-build the image.

All commands lines on docker files is a layer. And if you build the same project over and over again, docker will check the cache looking for any changes from previous build. If not, the image build will be much faster. Otherwise it will find a difference and the cache will build that particular layer again, which will take longer the build.

**Optimization Note:** Whenever we build the image, we do not want docker to check `node-modules` folders unless it is necessary with the `RUN npm install`. So instead of using the example we explained before, we will be using it like below:
```
FROM node

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 3000

CMD ["npm", "start"]
```
So at the end, docker will be building the image much faster and not check the `node-modules` all over again, because `package.json` did not change at all.


## Start & Re-Start
You can start a container with `docker run ...` and you can also re-start a container with `docker start ...` commands.

Run Command Example:
```
$ docker run -p <portYouWantForLocal>:<theExposedPortYouMentionedOnDockerFile> <imageId>
```

Re-Start Command Example:
```
$ docker start <imageId>
```


## Attach & Deattach Modes
Attach and Deattach modes stands for listening the output of containers or not. The **attach** mode will let you listen the container and **deattach** will not.

Run a container with **deattach** mode by adding `-d` flag:
```
$ docker run -p <portYouWantForLocal>:<theExposedPortYouMentionedOnDockerFile> -d <imageId>
```

**Re-Attach** container which is deattached:
```
$ docker attach <imageName>
```

**Attach** a re-started container with `logs` command and `-f` flag for follow logs:
```
$ docker logs -f <imageName>
```

Re-Start a container with **attach** mode by adding `-a` flag:
```
docker start -a <imageName>
```


## Interactive Mode
If you want to run a app that needs to get inputs from user, such as calculator like applications; you should run the container with 2 flags as we mentioned before. Which are `-i` for **interactive** and `-t` for **terminal**.
```
$ docker run -it <imageId>
```

Keep in mind, if you start a container which is calculator like application with **deattach mode**, it will fail. Because listening output of our container is closed. To **attach** the container we will be using `-a` for **attach** and `-i` for **interactive**.
```
$ docker start -a -i <imageId>
```


## Deleting Images And Containers
To delete a container (You will need to stop first):
```
$ docker rm <containerName/s>
```

Check all images:
```
$ docker images
```

To delete a image:
```
$ docker rmi <imageName>
```

To delete all containers:
```
$ docker rm prune
```

To delete all images:
```
$ docker image prune -a
```


## Removing Stopped Containers Automatically
To remove stopped containers automatically, we will be using `--rm` flag:
```
$ docker run -p <portYouWantForLocal>:<theExposedPortYouMentionedOnDockerFile> -d --rm <imageId>
```


## Inspecting Images
To inspect a image, which will let you see all the details about the image:
```
$ docker image inspect <imageId>
```


## Copy File Into & From Container
Copy from local to running container:
```
$ docker cp <filePath> <imageName>:/<destinationPath>
```

Example:
```
$ docker cp dummy/. eager_buck:/test
```
This command above will copy **dummy** folder from local, and paste it into **test** folder in **eager_buck** container. If there is not folder named **test**, it will create one.

Copy form container to local:
```
$ docker cp <imageName>:/<filePath> <destinationPath>
```

Example:
```
$ docker cp eager_buck:/test dummy
```
This command above will copy **test** folder from **eager_buck** container, and paste it into **dummy** folder in our local. If there is not folder named **test**, it will create one.

Quick Test:
```
$ mkdir dummy
- create a text file named test.txt
$ docker cp dummy/. eager_buck:/text
$ rm -r dummy/test.txt
$ docker cp eager_buck:/text/test.txt dummy 
```
This will copy back your txt file even after you deleted it.


## Naming & Tagging Containers And Images
Name a container with `--name` flag:
```
$ docker run -p <portYouWantForLocal>:<theExposedPortYouMentionedOnDockerFile> --name <containerName> <imageId>
```

We will be naming images with `-t` flag but, naming images is a bit different. You have names and you can set tags from the same images. Like when you specify on docker file such as `FROM node:14`, you mean that will be node application which will be using version 14. So when you name a image you can name them **and** give a tag so you can have different tags with same names. Also tag can be a word or number and tagging is **optional** and if you do not specify the tag, docker will set the tag as **latest**.
```
$ docker build -t <imageName>:<imageTag> .
```

## Re-Name Image
Re-Name the existing image with tag command (tags are optional again):
```
$ docker tag <existingImageName>:<existingImageTag> <newImageName>:<newImageTag>
```

## Docker Hub
- Create a [Docker Hub](https://hub.docker.com/) account
- Open a repository
- After creating the repository, it will create you a push command **with the repository name**
- Your image name should be the **same as repository**
- After re-naming the image, you can use the push command Docker Hub created for you
```
$ docker push <dockerId>/<repositoryName>:<imageTag>
```


### Login & Logout Docker Hub
To login:
```
$ docker login
```

To logout:
```
$ docker logout
```


### Pulling And Using Shared Images
To pull images (You can pull all the public shared images):
```
$ docker pull <imageName>:<imageTag>
```

You do not have to pull the image to run. Docker will search on docker hub if docker can not find in on local. So you could just enter run command without event pulling the image (for only public images).
