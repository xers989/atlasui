## Allegro Web application UI Server

### Feature and components
This UI web application is Vue.js based web application. It connects to Allegro-node API and communicate with Json message.
<img src="/images/allegro-saas/image1.png" width="70%" height="70%">  
This is basic architecture of Web application, that is running on Docker container. You can run the container image on docker or kubernetes engine. The container is running on 8080 port, you need the port is not used.          
<img src="/images/allegro-saas/image2.png" width="70%" height="70%">  
Before create container, you need to install docker engine.
Here is installation document about Docker.   
https://docs.docker.com/engine/install/   


### Allegro-saas project
This is structure of the project (allegro-saas).    

```bash
allegro-saa $ tree               
.
├── Dockerfile
├── README.md
├── babel.config.js
├── dist
│   ├── css
│   │   ├── app.20fa4479.css
│   │   └── chunk-vendors.d1f8d1cb.css
│   ├── favicon.ico
│   ├── index.html
│   └── js
│       ├── app.1b8b7bea.js
│       ├── app.1b8b7bea.js.map
│       ├── chunk-vendors.370b3d4c.js
│       └── chunk-vendors.370b3d4c.js.map
├── docker
│   └── nginx
│       └── nginx.conf
├── oke-deployment.yaml
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   └── index.html
├── src
│   ├── App.vue
│   ├── assets
│   │   ├── logo.png
│   │   ├── logo.svg
│   │   └── styles.scss
│   ├── components
│   │   ├── CargoShip.vue
│   │   ├── CargoShipList.vue
│   │   ├── Footer.vue
│   │   ├── Header.vue
│   │   └── Statistics.vue
│   ├── main.js
│   ├── plugins
│   │   └── vuetify.js
│   ├── router
│   │   └── index.js
│   ├── store
│   │   └── index.js
│   └── views
│       ├── Main.vue
│       └── SignIn.vue
└── vue.config.js
```
API server address is in main.js file.
```javascript
axios.defaults.baseURL = 'http://terraform.cloudiam.site:3002'
```
Replace the baseURL as your allegro-node IP address.
There are two main pages (main, SingIn), and the code are in src/components folder.   


### Docker Image build
In the project folder, there is dockerfile. So, you can create image by docker build command.    
``` bash
$ docker build -t allegro:1.0 .
Sending build context to Docker daemon    556kB
Step 1/13 : FROM node:lts-alpine as build-stage
 ---> f5f48375fc5d
Step 2/13 : WORKDIR /app
 ---> Using cache
 ---> c93abda7ba95
Step 3/13 : COPY package*.json ./
 ---> Using cache
 ---> 2678922bf435
Step 4/13 : RUN npm install
 ---> Using cache
 ---> 5217e94f94df
Step 5/13 : COPY . .
 ---> Using cache
 ---> c449ddbca786
Step 6/13 : RUN npm run build
 ---> Using cache
 ---> 15c42849747d
Step 7/13 : FROM nginx:alpine as production-stage
 ---> b9e2356ea1be
Step 8/13 : RUN rm -rf /usr/share/nginx/html/*
 ---> Using cache
 ---> 0f885520c841
Step 9/13 : COPY --from=build-stage /app/dist /usr/share/nginx/html
 ---> Using cache
 ---> db246ab8e33b
Step 10/13 : RUN rm /etc/nginx/conf.d/default.conf
 ---> Using cache
 ---> b86d61977c85
Step 11/13 : COPY docker/nginx/nginx.conf /etc/nginx/conf.d
 ---> Using cache
 ---> 01b7abf7f689
Step 12/13 : EXPOSE 8080
 ---> Using cache
 ---> 28f355a14a49
Step 13/13 : CMD [ "nginx", "-g", "daemon off;" ]
 ---> Using cache
 ---> ffa67843c6c4
Successfully built ffa67843c6c4
Successfully tagged allegro:1.0
```

Now you have Allegro-saas web application image, you can verify it is in your docker repositoy.
``` bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
allegro             1.0                 ffa67843c6c4        27 minutes ago      25.5MB
...
node                latest              8f1b7f0dfc2f        4 days ago          907MB
node                lts-alpine          f5f48375fc5d        2 weeks ago         117MB
nginx               alpine              b9e2356ea1be        2 weeks ago         22.8MB
```

### Running Container from image
You can run container from the created imaged by docker run command.      
After run the container, you can verify the status by docker ps command.   

``` bash
$ docker run -d -it -p 8080:8080 allegro:1.0
faa0a44b9b6f58202a4f91688a6a622081c7186b9c716fe0b78ee10dc45322dc
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                            NAMES
faa0a44b9b6f        allegro:1.0         "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp, 0.0.0.0:8080->8080/tcp   loving_cannon
```

### Test Web Application
The application is using 8080 port. if you want to use other port, change the docker run command. 

```bash
$ docker run -d -it -p <<Port>>:8080 allegro:1.0
```
Lauch your browser and access the web application.
If you access following page, it works properly.
<img src="/images/allegro-saas/image3.png" width="70%" height="70%">  


### Kubernetes (option)
If you want to run the container in Kubernetes engine, you can deploy it with oke-deployment.yaml.    
To run it on Kubernetes, you need docker registry, and Kubernetes engine. Also, kubectl is installed in your laptop.   
First, you need to push image into docker registry. You can use public docker hub or private docker registry.   

```bash
$ docker login <<your docker registry>>
username: ***
password: ***
...
Login Succeeded
$ docker tag allegro:1.0 <<your docker registry>>/***/allegro/allegro:1.0
$ docker push <<your docker registry>>/***/allegro/allegro:1.0
```

Revise the oke-deployment.yaml about image address and secret information
image: <<your docker registry>>/***/allegro/allegro:1.0
create your scret to access docker registry by kubectl command.   

```bash
$ kubectl create secret docker-registry mysecret --docker-server=<your docker registry> --docker-username='<docker registry user>' --docker-password='<password>' --docker-email='<email-address>'
```

You can create resource by oke-deployment.yaml with kubectl tool.   

```bash
$ kubectl apply -f oke-deployment.yaml
```
Here is the running architecture in Kubernetes.
<img src="/images/allegro-saas/image4.png" width="90%" height="90%">  