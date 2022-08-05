# Welcome
This is me taking notes on github while learning `Docker`. I will probably going to need this document when I forget how docker works. If you find someting usefull, be my guest. I will try to update this whenever I study.

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


## Introducing Volumes
Let's say your web application creates a file and you want to save this file for later. Usually when you shutdown the container, all the data will be lost. To prevent that, we can use `VOLUME` command with a path in where the file is located.

dokerfile with a volume example:
```
FROM node

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 3000

VOLUME ["/app/feedback"]

CMD ["npm", "start"]
```

After specifying the `VOLUME`, the file will not be lost after you stop the container, **unless you delete the container**. Docker will save the file on our local but does not let you access. But, **docker will give volume a unique name each time you run the container** and that causes us to not use it like we want to use.

Therefore, there are 2 types of volumes: **anonymous volumes** and **named volumes**.

> Anonymous volumes are closely attached to the container but, named volumes are not.

To create named volume, we will specify it on our `docker run ...` command with a `-v` tag. **We cannot create named volumes on our docker file**.
```
$ docker run -p <exposedPortForLocal>:<exposedPortYouMentionedOnDockerFile> -d --rm --name <containerName> -v <volumeName>:/<folderPath> <imageName>:<imageTag>
```

Example:
```
$ docker run -p 3000:3000 -d --rm --name feedback-app -v feedback:/app/feedback feedback-node:volumes
```

The example above, we can create a volume named `feedback`, which saves the **feedback** folder located under `/app`. Keep in mind that, `/app` is our root directory in the container, we specified this on our docker file.

After running our container with given command we mentioned in the example above, and then stop the container, which is removes the container because of the `--rm` tag, we still be able to have our volume. And if we run a new container with the same volume name and path, we will be able to access this volume file.

To list volumes:
```
$ docker volume ls
```

To delete a volume:
```
$ docker volume rm <volumeName>
```

To delete all volumes:
```
$ docker volume prune
```


## Bind Mounts
Let's say you have a web application to run with a docker. Since we **cannot change the container**, we also cannot edit our web application on docker. At least without building a new image and then running a container based on that image. Therefore, **bind mounts helps us make changes on out application inside our running container**.

> For windows user; we first need to follow the instructions [here](https://devblogs.microsoft.com/commandline/access-linux-filesystems-in-windows-and-wsl-2/)

Before we begin, make sure docker can access your application located on your host machine. If you are using `windows` operating systems, you do not have anything to worry about, it is already handled. But, if you are using `macOS`, simply open the **settings** on your docker application, click on **resources** and define your project path as accessible under **file sharing**.

Moreover, we **cannot define bind mounts in our docker file**. So, just like we did with the volumes, we need to define bind mounts in our run command.

To define bind mounts, we will be using `-v` tags just like we used for **volume**:
```
$ docker run -p <portYouWantForLocal>:<theExposedPortYouMentionedOnDockerFile> -d --rm --name <containerName> -v <volumeName>:/<filePath> -v "<absoluteFileOrFolderPathOnYourHostMachine>:/<workdirPath>" <imamgeName>:<imageTag>
```

Example:
```
$ docker run -p 3000:3000 -d --rm --name feedback-app -v feedback:/app/feedback -v "C:\Users\<userName>\Documents\GitHub\data-volumes-01-starting-setup\:/app" feedback-node:volumes
```

Since the absolute paths are long; we can just use `"&cd&"` for **windows**:
```
$ docker run -p 3000:3000 -d --rm --name feedback-app -v feedback:/app/feedback -v "%cd%":/app feedback-node:volumes
```

And `$(pwd)` for **Linux** / **macOS**:
```
$ docker run -p 3000:3000 -d --rm --name feedback-app -v feedback:/app/feedback -v $(pwd):/app feedback-node:volumes
```

**There is a problem with this command though**. Imagine we are running a web application again. We already mentioned that we need to install all the dependencies, such as `node_modules` folder. And to do that we also typed `RUN npm install` inside of our docker file. But since we run the container with the command above, we overwrite the application files all over with the bind mount. And that means we lost our dependencies, which will crash our application.

To prevent this problem, we will need to use another volume with a `-v` tag. We can also do this by typing `VOLUME [ "/app/node_modules" ]` but, that would mean we need to re-build our image. And if you have not already noticed, we will be using anonymous volume.
```
$ docker run -p <portYouWantForLocal>:<theExposedPortYouMentionedOnDockerFile> -d --rm --name <containerName> -v <volumeName>:/<filePath> -v $(pwd):/<workdir> -v /<workdir>/node_modules/ <imageName>:<imageTag>
```

Example:
```
$ docker run -p 3000:3000 -d --rm --name feedback-app -v feedback:/app/feedback -v $(pwd):/app -v /app/node-modules/ feedback-node:volumes
```

With the command above; we are running a container which will be accessable from port `3000`; will be on deattached mode with a `-d` tag; will remove the container once the container stops with the `--rm` tag; will name the container as **feedback-app** with a `--name` tag; will save the folder which located under `/app/feedback` on our container to our host machine with a `-v` tag; will allow us to edit the application with a `-v` tag again; will prevent us to overwrite `node_modules` folder with a `-v` tag again; and this container will be based on **feedback-node** image which has a **volumes** tag.

So we ran our web application with volumes and bind mounts. Is it enough? No. As you may know, if you worked with node project, even if you can see the front-end changes, you still will not be able to see your back-end changes. This is a common issue with node projects. And there is also a solution for it. It is called `nodemon`. Nodemon will re-start your web application when you refresh the page and it helps us to get the back-end changes too.

To do so; we will add a new dependency on our application:
1. Open package.json file
2. Add the whole `devDependencies` part as shown below
3. Also add `scripts` part as shown below
4. Make sure you have `CMD ["npm", "start"]`, instead of `CMD ["npm", "server.js"]` on your docker file.
```
{
  "name": "data-volume-example",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "start": "nodemon server.js"
  },
  "dependencies": {
    "body-parser": "^1.19.0",
    "express": "^4.17.1"
  },
  "devDependencies": {
    "nodemon": "2.0.19"
  }
}
```

### Volumes & Bind Mounts Differences

**Anonymous Volume**, which let the container save `data` folder:
```
$ docker run -v /app/data ...
```
> Created specifially for a singe container.
> Survives when container stops unless we delete the container.
> Can not be shared across containers.
> Since it's anonymous, it cannot be re-used.
> Helpfull for preventing overwriting.


**Named Volume**, which let the container save `data` folder and can be used again:
```
$ docker run -v data:/app/data ...
```
> Created in general, not tied to any container.
> Survives when container stops, even after we delete the container.
> Can be shared across containers.
> Can be re-used for same containers.


**Bind Mount**, which let the docker update the container when we make changes on our host:
```
$ docker run -v /path/to/code:/app/code ...
```
> Location on host file system, not tied to any container.
> Survives when container stops, even after we delete the container.
> Can be shared across containers.
> Can be re-used for same containers.

## Read-Only Volumes
So we now have a proper bind mount. The idea of the bind mount is that we want to make changes while the container is running. So we have **read** and **write** permit for our container. But, the container also has read and work permit for our app inside of the container. We do not want that. What we want is, we should be the only one making code changes. So we will give the bind mount a **read-only** permit.
> Container can't affect the project which is located in our host, it can only affect the project inside of that container.

Before we get into that; we could also let our container to write for some folders. So we will give a read-only permit for the application but, we will **exclude some folders** so we can use the data inside of those folders later.

To do so, we will just add `:ro` at the end of our bind mount and to exlude folders, we will just use `anonymous volumes`:
```
$ docker run -p <portYouWantForLocal>:<theExposedPortYouMentionedOnDockerFile> -d --rm --name <containerName> -v <volumeName>:/<filePath> -v $(pwd):/<workdir>:ro -v /<workdir>/node_modules/ <imageName>:<imageTag>
```

Example:
```
$ docker run -p 3000:3000 -d --rm --name feedback-app -v feedback:/app/feedback -v $(pwd):/app:ro -v /app/temp -v /app/node-modules/ feedback-node:volumes
```
So with the example above; we gave the bind mount a read-only permit. And we also let the container write for both `/app/feedback` and `/app/temp` paths.

> If we check the volume list with a `docker volume ls` command, we can see our all 3 volumes. 2 anonymous and 1 named. Keep in mind that, we cannot see our bind mount. It is not listed as **volume**.

## Important Note
We are ran our container with a **volumes** and **bind mount** so far. And we know we are loading our application to container with that **bind mount**. So why do we have still `COPY . ./app` on our docker file, can we delete it from our docker file?

The answer is **yes**, but don't. Our application will run properly either way. But the thing is, we should not start our container with a bint mount if we are working on production. Because we might not be able to give a absolute path for the production. So we want to start our container with a snapshot (with `COPY` command).

## Important Note
We mentioned about `COPY . ./app` command before. Which means we copy all the files inside of the image. But, what if we do not want that. Our application might contain sensitive data and copying that data might be a serious mistake. Or we could just ignore some files, such as `node_modules`.

To ignore files/folders for docker, we simply create `.dockerignore` file on our root directory like we created our dockerfile. The usage of this file is same is `.gitignore`.
> While creating `.dockerignore`, do not forget about the dot at the beggining of the name.

# Arguments & Environment Variables

## Environment Variables & .env Files
To specify a environment variable, we will be using `ENV` command. Let's say we want our port as a environment variable. First thing first, we will add like `ENV <name> <value>`. So, our docker file should look like this:
```
FROM node

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

ENV PORT 3000

EXPOSE $PORT

CMD ["npm", "start"]
```
In the example above, we can see we specified our port as `ENV PORT 3000`. And we also used that variable ın our docker file like `EXPOSE $PORT`.

But that's not enough. In our main class, we still have it as hard-coded. So we will replace that as `process.env.PORT`, like the example below:
```
app.listen(process.env.PORT);
```

We can also clerify variables in our run command. If we want to over-write our port from **3000** to **8000**, we can use `--env` or `-e` tags.

Example:
```
$  docker run -p 3000:8000 --env PORT=8000 -d --rm --name feedback-app -v feedback:/app/feedback -v $(pwd):/app/ro -v /app/node_modules -v /app/temp feedback-node:env
```
So we clerified our PORT variable as **8000**. Just do not forget that, after changing the port, we also need to change our exposed port source.

We can also give variables from a file. For such things, we create a file on our root project directory named `.env`. We then, can clerify the port as shown below in `.env` file:
```
PORT=8000
```

After doing so, we can simple show our .env file in our run command with a `--env-file` tag like this:
```
$ docker run -p 3000:8000 --env-file ./.env -d --rm --name feedback-app -v feedback:/app/feedback -v $(pwd):/app/ro -v /app/node_modules -v /app/temp feedback-node:env
```

## Important Note
If your project has private keys, database urls etc. You might want to prefer using a separate environment variables file, like `.env` file. And make sure you added that file as ignored in your `.dockerignore` file. Otherwise anyone can see them with a simple `docker history <image>`.

## Build Arguments (ARG)
We already changed clerified our port as environment but, we can still make it better. We can set a default value and use that value for our environment value **and** we can also change that value with a build command.

To do that, we will add `ARG` command like this:
```
FROM node

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

ARG DEFAULT_PORT=3000

ENV PORT $DEFAULT_PORT

EXPOSE $PORT

CMD ["npm", "start"]
```
So we clefied our default port with a `ARG DEFAULT_PORT=3000` and we used that argument in our environment value as `ENV PORT $DEFAULT_PORT`.

> We cannot use argument values in other classes, we can only use them in our docker file.

After clarifying our argument, we can now change the default port however we want with a `--build-arg` tag in our build command like this:
```
$ docker build -t feedback-node:arg --build-arg DEFAULT_PORT=8000
```
This will over-write our port as 8000 and we can access our app through this port.


# Containers & Network Requests
Let's say your node application has API requests, like GET and POST requests. And maybe you have a local database and you want your dockerized application to communicate with this local application. It is also possible to dockerize a database and make them communicate each other. To do that, we will be focusing 3 different network requests:

## Requests from container to network
This is the case when you want to use API. To manage APIs, you do not have to do anything different. Containers can manage that as long as your application is working.

Example:
```
mongodb://localhost:27017/swfavorites
```

## Requsts from container to your local
In this case, let's say your application needs to communicate with your local database. Your application would run if you were running on your local but, when you dockerize your application, it will not work.

To fix this, you simply need to change your connection url. We are assuming your database url is something like `mongodb://localhost:27017/swfavorites`. That will not work. Because docker will not recognize `local`. What docker recognize is: `host.docker.internal`. So all you have to do is to change your url as mentioned.

> As summary: use `host.docker.internal` rather than `localhost` on your local connetion urls.

Example:
```
mongodb://host.docker.internal:27017/swfavorites
```

## Request from container to other container
So, in this case, let's say we want our mongodb database to run on a different container, and access that container from another container.

And since we can run node on containers without even installing on our local, we can also run mongo on docker without installing it.

To run mongo on container, we will simple run the command below:
```
$ docker run -d --name mongodb mongo
```
> The command above will look at our images if we have `mongo` image. And since we do not, it will pull the official image from docker itself. If you are not certain how to call images, you can google as `docker mongodb` etc.

Just keep in mind that, if you run docker inside of a container, neither `localhost`, nor `host.docker.internal` will work. Because our mongodb is not in our local. So we need to set our containers IP address.

To find our containers IP address, we will use `inspect`:
```
$ docker container inspect <coontainerName>
```

> Look for the tag called `IPAddress`, under `NetworkSettings`.

After setting the IP as show above, you will be able to use your database and your application.

But, this is hard-coded. There is a better way. So, we already managed to run the application just fine but, the IP we used for communicating between one container and other might change. So to prevent that issue and also to make our application more secure, we will be using **Docker Network**.

Docker Network is simply puts the container in a specified network. You can add other containers into that network. With that, all the containers inside of that network, will be able to communicate.

To do so; we first need to create a network. To do create a network, we will be using `--network` tag:
```
$ docker network create <networkName>
```

After creating the network, we will be changing the url of our connection inside of our application. The part in our application, the `localhost`, or `host.docker.internal` part will actually be our **container name which we want to communicate**. So if you build your mongo container named like `mongodb`, you need to set the url as `mongodb`.

Example:
```
mongodb://mongodb:27017/swfavorites
```

But that's not all. We created the network but, we have not put the containers inside yet. To do that, we will be again, using `--network` tag on our run command, just like below:
```
$ docker run -p 3000:3000 -d --rm --name <containerName> --network <networkName> <imageName>
```

> Docker Network also support differet drivers. The default drive is `bridge` but, if you want to change that you can simple use `--driver` tag as: `docker network create --driver bridge my-net`.

 
# Building Multi-Container Application
Let's say we want to run a web applicaiton. We want to run mongodb on a one container, back-end on an another container and, frontend on a different container. We also want to save a log file from backand application. And we also want to save the data from our mongodb. Plus, we also want our containers connected to each other.

## Starting with network:
```
 $ docker network create <networkName>
```

## Then, mongodb:
```
$ docker run -d --rm --name mongodb --network <networkName> mongo
```

## Then, backend:
> Before building the backend container, **make sure you are using `container name` which you started your mongo rather than using `localhost`**.
> We also need to expose a port with `-p` tag for react to connect our backend application. We will be discuss this in the frontend part, right below.
Dockerfile example:
```
FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

CMD ["npm", "start"]
```

```
$ cd backend
$ docker build -t <imageNameforBackend> .
$ docker run -d --rm --name <containerNameforBackend> -p 80:80 --network <networkName> <imageNameforBackend>
$ cd ..
```

## And then, frontend:
> We are using `react` for frontend. That means the javascript code will run on the browser, not in the container. And that means, changing `localhost` to `container name` **will not work**. So, instead of using container name, we will use something the browser can understand. Which is `localhost`.
Dockerfile example:
```
FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

> Keep in mind that, when running a container with `react`, you should use `-it` tags. We are not going to interact with it on console or monitore it but, this is how react works.

> Since react does not run on docker, it runs on browser, there is also no use of adding `--network` tag on the run command. Because it won't matter anyway. Therefore, to connect react to our backend application, we already exposed a port. That is because react cannot run in container, and we couldn't put it into any network. So we need to make sure react can reach the backend with localhost ip with a exposed port.
```
$ cd frontend
$ docker build -t <imageNameforFrontend> .
$ docker run -d --rm --name <containerNameforFrontend> -it <imageNameforFrontend>
$ cd ..
```
 
So right now, we successfully managed to start our application and it is working. But, that is not all. We can add data on our database, but when we stop our mongo container, our all stored data will be lost. So we need to save our data with a `named volume`. We are not using **anonymous volume** because it will be impossible to re-use it after re-starting the container. And, we cannot use **bind mount** because we do not have the absolute path. So, **named volume** will just save our data on our host, and when we re-start our container, it will re-load our data, and if there are not data, it will create one.

> To use `named volume`, we need a path where mongo saves data. And to find where it is, we can look up to [docker mongo](https://hub.docker.com/_/mongo) page.

So, we will run our mongo container as this:
```
$ docker run -d --rm --name mongodb -v data:/data/db --network <networkName> mongo
```

## Securing database access
We make sure our data will not get lost and removed even after we delete our database container. But, we also need to secure our database access. To do that, we will be using `-e` tags to give a **username** and a **password** on our run command like this:
```
$ docker run -d --rm --name mongodb -v data:/data/db --network <networkName> -e MONGO_INITDB_ROOT_USERNAME=<username> -e MONGO_INITDB_ROOT_PASSWORD=<password> mongo
```
> The tags we used as `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD` are again in the [docker mongo](https://hub.docker.com/_/mongo) page.

If we run our container just like this, event our application cannot be able to access our database. So, we need to modify our db url too.
Our database url was, let's say `mongodb://mongodb:27017/course-goals`. Now we need to modify as `mongodb://<username>:<password>@mongodb:27017/course-goals?authSource=admin` as shown in [Mongodb Documentation](https://www.mongodb.com/docs/manual/reference/connection-string/) page.

> **IMPORTANT NOTE:** If you get an `Authentication failed` error, just delete the volume we created before while running mongo.

# Docker Compose
So far, we learned how to start containers and and build images with our dockerfile and we also learned how to add different options on our `build` and `run` commands. Like we did in the last practice, we used `-e`, `-v`, `--network` tags and so on. Since we had lots of things to add, these commands might get a bit long and it may increase to typo and fail. So, to make this easier and efficient, we will use **Docker Compose**. Which will help us **not to use long commands**. 

> Docker compose **does not** replace `Dockerfiles`.

> Docker compose **does not** replace `Images` or `Containers`.

> Docker compose is **not suited** for multiple containers on different hosts/machines.

## To Start
> If you are using `Linux`, you should install docker compose separately by following the [Install Docker Compose](https://docs.docker.com/compose/install/) page.

We will first create a `yaml`/`yml` file named `docker-compose` on our root directory. So at the end, we will have a `docker-compose.yaml` file.

In this file, we will be using a different syntax. And this file has some required parameters. Such as, `version` and `services`. These parameters are special, you do not want to shorten or modify the name, it should be just as it is.

It is also important to check your blanks. Imagine as a JSON file without brackets.

> At this point, it would be very helpfull to add `Docker` extension on your IDE because, it will help you while writing.

Example:
```
version: '3.8'
services:
  <containerName_1>:
    image: 'mongo'
    volumes:
      - data:/data/db
    environment:
      #- MONGO_INITDB_ROOT_USERNAME=user
      MONGO_INITDB_ROOT_USERNAME: user
      MONGO_INITDB_ROOT_PASSWORD: secret
    env_file:
      - ./env/mongo.env
    networks:
      - goals-net
  <containerName_2>:
  <containerName_3>:
volumes:
  data:
```

- As you can see, we started with `version`, which defined as 3.8. And then `services`, which contains our `containers`. So, we can say **services** is actually **parent** of our **containers**.

- We then defined images for our containers like `image: 'mongo'`.

- Volumes are the same way we used in our commands

- Environments are both acceptable for both example above. You can also use this as `env_file` as shown above. You just need to show the `env` file path which contains the variables such as `MONGO_INITDB_ROOT_USERNAME=username`.
> `#` is used for comment lines.

- Networks is just to clarify the network name.
> Since docker composer will start all these containers together. We do not actually need to clearify a network because, they will be allowed to communicate. But there is nothing wrong with defining again.

> If you noticed, we defined `volumes` also as upper title. That is because YAML want us to define **only** the `named volumes` again, just so we can use it in another containers too. **Anonymous volumes** and **bind mounts** are not included here.

> Before running the compose, **clear** the `images`, `containers`, `volumes`. And start the compose from the root directory of your project file, which is also the same file your `docker-compose.yaml` located.

So, after finishing with our docker compose file, we can start the compose with the command below. But, we also want this compose on deattach mode with the same `-d` tag. The stop command for compose is actually removes all the containers and images but, it does not removes `volumes`. To also remove those, we will also use `-v` tag.
```
$ docker-compose up -d -v
```

To stop docker compose:
```
# docker-compose down
```

## Working With Multiple Containers
We learned how to start a docker compose which includes a container. Now it is time for adding 2 more container which are actually our node/backend and react/frontend containers we already talked about before.

Example:
```
version: '3.8'
services:
  mongodb:
    image: 'mongo'
    volumes:
      - data:/data/db
    env_file:
      - ./env/mongo.env
  backend:
    build: ./backend
    #build:
    #  context: ./backend
    #  dockerfile: Dockerfile
    #  args:
    #    some-arg: 1
    ports:
      - '80:80'
    volumes:
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    env_file:
      - ./env/backend.env
    depends_on:
      - mongodb
  <containerName_3>:
volumes:
  data:
```

So as you can see, we are added `backend` container now. So, first thing first, we used shared image for our `mongodb`. But, we do not have any images for backend. 

- To tell compose to build the image, we use `build` tag. The example abone we told compose that, for backend, compose will go to `backend` folder which is also located under root directory, and look for `Dockerfile` spesificly. 

> `Build` tag has two types. Long and short. Short version is just a un-commented part on the example above and long version is commented. The difference is you can give spesifics with longer version.

- Ports, is just as it is.

- Volumes, we have 3 volumes here. At the top we can see the ``named volume` as we did before. The second one is `bind mount`. As you remember, in bind mounts, we need to use absolute paths but in our docker compose, we do not need that. So we just show the path we want to use. And lastly, we have a `anonymous volume` which is same as it is.

- Depends On is, we tell compose this container is depending on the `mongodb` container. So, compose will know which one to build first.

And now, we will add our `frontend` container.

Example:
```
version: '3.8'
services:
  mongodb:
    image: 'mongo'
    volumes:
      - data:/data/db
    env_file:
      - ./env/mongo.env
  backend:
    build: ./backend
    #build:
    #  context: ./backend
    #  dockerfile: Dockerfile
    #  args:
    #    some-arg: 1
    ports:
      - '80:80'
    volumes:
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    env_file:
      - ./env/backend.env
    depends_on:
      - mongodb
  frontend:
    build: ./frontend
    ports:
      - '3000:3000'
    volumes:
      - ./fontend/src:/app/src
    stdin_open: true
    tty: true
    depends_on:
      - backend
volumes:
  data:
  logs:
```

- So now, we told docker to look for a `Dockerfile` under `frontend` folder.

- The `ports` are as is.

- We have a bind mount for the path `/app/src`

- So as we discussed before, react is run as interactive. We were running this container with `-it` tags. So, to do same on our docker compose, we are using both `stdin_open` tag for telling, this container will be able to get inputs, and `tty` tag for telling this container will need a terminal.
> We start the docker composer on deattach mode so we will not be seeing this terminal for the react, but the terminal will be there on browser.

## Also
We learned `docker-compose run` will build our images and run the containers. However, if the images already builded or are public images, compose will not build the images again. To forse compose **re-build** images every time we start, we can use `build`:
```
docker-compose build 
```

If you noticed, when we start our containers with docker compose, the compose will assign names to each container. The names will include our container names but, not the same name as we defined. To set the names as we want, we can add `container_name: <containerName>` to our `docker-compose.yaml` file under each container.

## In conclusion
So at the end, our whole `docker-compose.yaml` file will be looking like this:
```
version: "3.8"
services:
  mongodb:
    image: 'mongo'
    volumes: 
      - data:/data/db
    # environment: 
    #   MONGO_INITDB_ROOT_USERNAME: max
    #   MONGO_INITDB_ROOT_PASSWORD: secret
      # - MONGO_INITDB_ROOT_USERNAME=max
    env_file: 
      - ./env/mongo.env
    container_name: mongodb
  backend:
    build: ./backend
    # build:
    #   context: ./backend
    #   dockerfile: Dockerfile
    #   args:
    #     some-arg: 1
    ports:
      - '80:80'
    volumes: 
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    env_file: 
      - ./env/backend.env
    depends_on:
      - mongodb
    container_name: backend
  frontend:
    build: ./frontend
    ports: 
      - '3000:3000'
    volumes: 
      - ./frontend/src:/app/src
    stdin_open: true
    tty: true
    depends_on: 
      - backend
    container_name: frontend
volumes: 
  data:
  logs:
```

# VoltDB on Docker
Let's run 3 node local `VoltDB` cluster:
- Create a network so nodes can connect to each others.
- Start each node one by one.
- List exposed ports to our local.
```
$ docker network create -d bridge <networkName>
$ docker run -d -P -e HOST_COUNT=3 -e HOSTS=node1,node2,node3 --name=node1 --network=<networkName> voltdb/voltdb-community
$ docker run -d -P -e HOST_COUNT=3 -e HOSTS=node1,node2,node3 --name=node2 --network=<networkName> voltdb/voltdb-community
$ docker run -d -P -e HOST_COUNT=3 -e HOSTS=node1,node2,node3 --name=node3 --network=<networkName> voltdb/voltdb-community
$ docker port node1
```

> You can connect through browser with the url `http://localhost:<port>`. To see the correct port, you should use the exposed port for **Web Interface Port (httpd)**, which is `8080` by default. **NOT THE `8080` ITSELF BUT, THE EXPOSED PORT FOR `8080`**.

> To connect with `sqlcmd`; open a terminal from the container, you can easily open a terminal from containers tab if you installed `Docker Desktop`. You can connect by using `sqlcmd --servers=<containerName>`, we used `node1`, `node2` and `node3`. Or `sqlcmd --port=<clientPort>`, the client port is `21212` by default.

The main idea about which port to use is that, if you are connecting the voltdb from **outside** of the container, you should use the exposed ports for your local. And if you are connecting from **inside** of the container, you can simple use the default voltdb ports.

> For more info, check out [Docker Hub](https://hub.docker.com/r/voltdb/voltdb-community). And [this](https://vimeo.com/378332643) video might help too.

To start a 1 node VoltDB, use the run command like this: `docker run -d -P -e HOST_COUNT=1 -e HOSTS=node1 --name=node1 --network=voltLocalCluster voltdb/voltdb-community:6.6`, and that is it.

# Utility Containers
We already covered how we can set environments and run applications with docker. But we can also use docker just to set environmets, such as node.

To give a simple example; if we run node with a `docker run -it -d node`, the node application will run and wait for the input. Therefore, while the node application waiting for input, we can run a command without actually attaching the container with the `exec` command:
```
$ docker exec <containerName> <specifiedCommand>
```

> With `exec`, we can execute commands in our running container without interrupting default commands, which starts when the container starts.

Let's say you want to build a node application template while the node container running. After you run the container with `docker run -it -d node` command, you can now use the `exec` command like this:
```
$ docker exec -it <containerName> npm init
```

> `npm init` is a npm command that helps us create node application template very easily.

We can also interrupt the default code. The default command for node is a node itself and letting us use the node, such as calculator. And if we want to run specified code before that, we can use the code after the image name like this:
```
$ docker run -it <imageName> <specifiedCommand>
```

Example:
```
$ docker run -it node npm init
```

> As default `docker run -it node` would start a node terminal and would exit once we finished. But, with us interrupping the default commands, node will start to build node tepmlate and then exit.

Let's create a project with nothing inside. Just create a Dockerfile and add `FROM node:<version>` and `WORKDIR /app`, like this:
```
FROM node:14-alpine

WORKDIR /app
```

Build the image with `docker build -t <imageName> .`. Then run the image with `docker run -it -v "<absolutePathOfProject>:/app" <imageName> npm init`.

The run command will start the npm init command immediately and, once we gave the inputs for npm init, we will be able to see our **package.json** file in our local, thanks to bind mount we gave.

## ENTRYPOINT
As we mentioned. Adding specified command at the end of the run command will over-ride our default commands like we use as `CMD ["executable"]`. But since we can use node command, because we specified as `FROM node`, we can only use node commands. And we can actually make our run command simpler.

To dockerfile exapmle above, we add `ENTRYPOINT` like this:
```
FROM node:14-alpine

WORKDIR /app

ENTRYPOINT [ "npm" ]
```

And after that, we do not have to simplify `npm` anymore. So our run command would be like this:
```
$ docker run -it -v "<absolutePathOfProject>:/app" <imageName> init
```

> We can also use `install` like this: `docker run -it -v "<absolutePathOfProject>:/app" <imageName> install express --save`. And since the container will stop each time, we also need to do this commands one by one.

## Using Docker Compose
At the example above, we had to use multiple run commands. We can also set a docker-compose for that.

Add a `docker-compose.yaml` file and type the run commands, which will be like this:
```
version: '3.8'
services:
  npm:
    build: ./
    stdin_open: true
    tty: true
    volumes:
      - ./:/app
```
But, there is a problem here. The problem is we are just executing only npm, because of the `ENTRYPOINT [ "npm" ]` command in our dockerfile.

`docker-compose up` is ment to bring up services to find in a **docker-compose.yaml** file. For this utility containers, we have other commands:

- `docker-compose exec` to run commands in already running containers, which were created by docker compose.

- `docker-compose run ...` allows us to run a single service on the **docker-compose.yaml** file by the service name.

So we can use that as:
```
$ docker-compose run npm init
```

> As you know, docker compose will remove the exited containers if you start the container with `docker-compose up command`. However, we are using `docker-compose run ...`. And that does not remove the containers. To remove exited containers with docker compose, we will add `--rm` command as we used before.

# Dockerized Laravel & PHP
Let's try something different. Until now, we already covered node and mongodb many times. And now we will be creating a PHP project to extend our knowledge.

In this project, we will be using 6 containers: one container for `PHP Interpreter`, one for `Nginx Web Server`, one for `MySQL Database`; and we will have 3 other containers as utility for `Composer`, `Laravel Artisan` and `NPM`. Also, we will have our source code folder in our host machine. 

## Nginx (Web Server) Container
So we have a empty folder. First thing first, create `docker-compose.yaml` file. And we will add all the 6 services to our docker compose file. And, one of them is for `nginx`. We will specify an `ìmage`, `port` and a `bind mount` for **nginx.conf** file which you do not need to know much about it, we will use a basic template. At the end, your docker compose file should look like this:
```
version: '3.8'
services:
  server:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  #php:
  #mysql:
  #composer:
  #artisan:
  #npm:
```

> We specified `/etc/nginx/nginx.conf` path for our `nginx.conf` file. That path is already given us at [Nginx Docker Hub](https://hub.docker.com/_/nginx) page. So it is not just random path.

As you can see, we included a nginx.conf file. So, create `nginx` folder and add `nginx.conf` file and that, which will have the following command:
```
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    root /var/www/html/public;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

> We used `php` as IP and then give it port as `9000`. As we learned before, the containers need the container name as IP if they are inside of the same network. And the 9000 port is default exposed port from PHP.

## PHP Container
Create a folder as `dockerfiles`. Add a docker file named `php.dockerfile`. We will add the following commands in there:
```
FROM php:8.1-fpm-alpine

WORKDIR /var/www/html

RUN docker-php-ext-install pdo pdo_mysql
```

- We use php with the 7.4 version

- We set the `/var/www/html` path which we already set in our `nginx.conf` file.

- And we install pdo_mysql

After this, we will add the following commands to our `docker-compose.yaml` file for the container `php`:
```
version: '3.8'
services:
  server:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  #mysql:
  #composer:
  #artisan:
  #npm:
```

- We set the specified the docker file detailed, because we have a `dockerfiles` folder now.

- We have a `bind mount` for `src` folder which we need to create again. And we set that folder to same folder we set in our `nginx.conf` file.

- We also added `delegated` at the end of our bind mount. That is for `optimization`. The docker will not keep sending the file or check the file so it will not use as much power as before.

## MySQL Container
First thing first, we need to check [MySQL Docker Hub](https://hub.docker.com/_/mysql) page because, MySQL has mandatory environment variables such as, rood password. We also want to add a separete user and password. But, we also want to give those variables from a `env` file. So, create a `env` folder, then inside of that folder, create a file named `mysql.env`, which will have the following variables:
```
MYSQL_DATABASE=homestead
MYSQL_USER=homestead
MYSQL_PASSWORD=secret
MYSQL_ROOT_PASSWORD=secret
```

We then, add the env file to our docker compose file and also set our image like this:
```
version: '3.8'
services:
  server:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  #composer:
  #artisan:
  #npm:
```

So far, we cannot check if we made a mistake. To check if all the thing we did working, we need `laravel`, which we will do next with our `composer` container.

## Composer Utility Container
We will start with creating a another dockerfile inside of our `dockerfiles` folder, which will be named `composer.dockerfile`.

Then, we will add the following layers:
```
FROM composer:latest

WORKDIR /var/www/html

ENTRYPOINT [ "composer", "--ignore-platform-reqs" ]
```

After that, we will set our container to our docker compose file like this:
```
version: '3.8'
services:
  server:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  composer:
    build: 
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  #artisan:
  #npm:
```

After that, we can finally try our **composer** service. To do that, we simply try the command we can find in [Laravel Documentation](https://laravel.com/docs/9.x) which is `docker-compose run --rm composer create-project --prefer-dist laravel/laravel .`.

If the application builded successfully, you can see new folders inside of your `src` file. And there, we can actually write some php code.

## Permission Error
If you are using Docker on **Linux**, you might get some permission errors. It is because bind mounts. Try the following layers on your `php.dockerfile`:
```
FROM php:8.0-fpm-alpine
 
WORKDIR /var/www/html
 
COPY src .
 
RUN docker-php-ext-install pdo pdo_mysql
 
RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel
```

So, assuming the build was success, now we need to check the `.env` file inside `src` folder. Open the file and find db connection variables.

We will change the `DB_HOST` as our container name which is `mysql`, and we will also need to change our `DB_DATABASE`, `DB_USERNAME` and `DB_PASSWORD` as we already clarifed in our `mysql.env` before. So it should look like this:
```
...
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
...
```

Now, before we start, we also need to add another bind mount for our nginx. The server we created does not know which file to make a connection yet. So, we will open the `docker-compose.yaml` file again and add another bind mount under nginx service like this:
```
version: '3.8'
services:
  server:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    volumes:
      - ./src:/var/www/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  composer:
    build: 
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  #artisan:
  #npm:
```

The bind mount we gave as `./src:/var/www/html` under nginx server has the same path we already mentioned in our `nginx.conf` file.

Now, we will just run the 3 containers which are, `server`, `php` and `mysql` with `docker-compose up -d server php mysql` command.

## Permission Error
There is a hight change of getting an permission error once you open `localhost:8000` on your browser. It is because `storage` folder inside of your `src` folder on your container. There are couple of way to fix that but, what I do is much easier:
- Open **Docker Desktop**
- Click on **Containers**
- Click on your container name
- It will bring up 3 containers to you, click on the container which has `server` somewhere
- Open CLI
- Go to `/var/www/html` which was the root path for service container.
- Give the permission with `chmod o+w ./storage/ -R` command for **storage** folder

After that, you will be able to see Lavarel Home Page when you go to `localhost:8000` on your browser.

Moreover, assuming we have a running Laravel application. We also have our nginx service but, it does not recognize our code. To do that, we will add `depends-on` layer on our `docker-compose.yaml` file, like this:
```
version: '3.8'
services:
  server:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    volumes:
      - ./src:/var/www/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - php
      - mysql
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  composer:
    build: 
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  #artisan:
  #npm:
```

With that, we made sure that once we start the application, it will build `php` and `mysql` first, then it will build `service` container. And that also make our docker compose run command a bit easier now. We can now just specify `service` and not specify `php` and `mysql` services because, our service is depends on other 2. So we can start the 3 container with just `docker-compose up -d server` command. But, since we might want to change some code in our local, and we want docker to reflet those changes on our container too, we want to add `--build` command so whenever we use the `docker-compose up ...` command, the image will be re-build.

## Adding More Utility Containers
We will add the `artisan` and `npm` container to our `docker-compose.yaml` file like this:
```
version: '3.8'
services:
  server:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    volumes:
      - ./src:/var/www/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - php
      - mysql
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  composer:
    build: 
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  artisan:
    build:
      context: .
      dockerfile: dockerfiles/php.dockerfile
    volumes:
      - ./src:/var/www/html
    entrypoint: ['php', '/var/www/html/artisan']
  npm:
    image: node:14
    working_dir: /var/www/html
    entrypoint: ['npm']
    volumes:
      - ./src:/var/www/html
```

For **artisan** service:
- We used the same dockerfile for our `artisan` as we used for our `php` service, because artisan is also uses php
- Add our source code to container with a **volume**
- And we specified artisan file with a php command as **entrypoint**, which we can find the folder in our local too

For **npm** service:
- We specified it will run **NPM** with a version 14
- It will have the working directory
- A entrypoint of NPM
- And again, our source code.

After that we will start the artisan with a `docker-compose run --rm artisan migrate` command to migrate, so we can actually see if our database is working too. After the command, we should see our table, password and etc migrated with a `DONE` status.

> Before running the artisan container, we assume your service container is already up and running.

# Deploying Docker Containers