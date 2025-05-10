# Deploy A Two Tier Application using Docker.

![](https://miro.medium.com/v2/resize:fit:700/1*kstwuNhrkun-LziGRsvviQ.png)

# Introduction

In this project, two containers will be deployed:

1. *Flask application* container.
2. *MySQL database* container.

Prerequisites:

1. Deploy the AWS EC2 instance in your AWS Console.

2. Ensure you have docker running on your EC2.

Now, let's get down to some hands-on!✨

# **Step 1: Clone the git repo into EC2.**

$ `git clone <git URL>` - this will clone your repo from the GitHub to your local project directory.

$ `git clone [https://github.com/S47sawan/two-tier-flask-app.git](https://github.com/S47sawan/two-tier-flask-app.git)`

# **Step 2: Create volume for the MySQL database**

- In your projects folder, $ `mkdir volumes`
- $ `cd volumes`
- $ `mkdir flask-app` - when you run your MySQL container, data will be stored in this folder and it will persist should you destroy your container.

![](https://miro.medium.com/v2/resize:fit:700/1*FaMUei-U6hUsUFj8ANcxSw.png)

Make sure you save your device path as it will be used to bind when creating the volume itself.

![](https://miro.medium.com/v2/resize:fit:481/1*odhS84aHie92INPdaF8lKQ.png)

- $ `cd two-tier-flask-app/` -go back to your application folder.
- Next, run this command to create the volume for the Mysql database.
- $ `docker volume create --name mysql-volume --opt type=none --opt device=/home/ubuntu/mydockerbuild/volumes --opt o=bind`
- Run $ `docker volume ls` - to view the created volume.

![](https://miro.medium.com/v2/resize:fit:700/1*H_qr0jja2YUjtEXaFxi9Og.png)

- Your volume is now ready to be mounted onto the MySQL container during the launch process.

# **Step 2: Launch the MySQL database container.**

- launch the MySQL database container with the volume mounted to it as below:

$ `docker run -d -p 3306:3306 --mount source=mysql-volume,target=/var/lib/mysql --name mysql -e MYSQL_ROOT_PASSWORD=<password> -e MYSQL_DATABASE=testdb -e MYSQL_USER=admin -e MYSQL_PASSWORD=<password> mysql:latest`

- MySQL database runs on port 3306. Name the container using the `-name` command. Naming containers makes it easier to reference them while building the network. `e` is used to refer to the environment variables. To see more docker MySQL env variable refer [here](https://hub.docker.com/_/mysql). Append the image name at the end (MySQL: latest).
- As the image is not available locally, docker pulls the image from the docker hub.

![](https://miro.medium.com/v2/resize:fit:700/1*0dpH2d_PJll0k21AMG0Cbg.png)

# **✔️Check testdb (MySQL database has been created).**

- In order to check that the database has been created, exec into the container. Run this command.

$ `docker ps` - to get the docker ID

then,

$ `docker exec -it <container id> bash`

![](https://miro.medium.com/v2/resize:fit:700/1*Y-36z0MN3urGZQ_IgOT50w.png)

- You are now inside your database container. Next run:

$ `mysql -u root -p`

- Check the database, and run this command:

![](https://miro.medium.com/v2/resize:fit:700/1*qoz9TTBWK-H4ct0MXzzQ9A.png)

$ `show databases;`

![](https://miro.medium.com/v2/resize:fit:237/1*qRFNa0V6jIhizm5HaAOIoA.png)

- Your testdb data is now created!! 😄

# **✔️Check volume has been mounted.**

- Head back to your volumes directory and cd into the flask-app directory. The flask-app directory in volumes has now been populated.

![](https://miro.medium.com/v2/resize:fit:700/1*7yGxTHEw48Pz204MOExdGQ.png)

You are now good to go to your next phase, Create the frontend, flask app container.

# **Step 3: Create your frontend (flask app container)**

- At this moment we have the mysql image. To create the flask-app image (make sure you are in the path that contains your application + Dockerfile) run:

$ `docker build -t flask-app .`

![](https://miro.medium.com/v2/resize:fit:700/1*DPCAOEhe41g5f4YwMeuSZA.png)

$ `docker images`-check image has been built.

![](https://miro.medium.com/v2/resize:fit:668/1*-Y3mNOOwFL8mvbfyHrIVjg.png)

$ `docker run -d -p 5000:5000 --name flask-app -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e MYSQL_PASSWORD=<youradminpassword> -e MYSQL_DB=testdb flask-app:latest`

- The env vars used in the above command are from the flask application, and the application is exposed on port 5000. Name the application container as,- -name flask-app.

![](https://miro.medium.com/v2/resize:fit:700/1*HobSEy_jFXmtXxjZm3KvWQ.png)

- Next open port 5000 on your EC2 security group and, check that your application is running in the browser.

![](https://miro.medium.com/v2/resize:fit:700/1*7WMy6J5qOaMjgOzhrLq1ng.png)

- As evident from the provided screenshot, an error “unknown server host ‘mysql’” is observed. This issue arises due to the current isolation of the containers. To establish communication between them, a connection must be established. By default, containers operate within a bridge network, and currently, the application is unable to reach the MySQL database container.

$ `docker network ls` - lists the currently available networks.

![](https://miro.medium.com/v2/resize:fit:699/1*2RFxov9GLrFH3P8396G78Q.png)

# **Step 4: Create a network.**

- To create a network, run the command below in your terminal:

$ `docker network create -d bridge two-tier-app-nw`

Next run:

$ `docker network ls` -lists existing networks as well as new ones.

output :

![](https://miro.medium.com/v2/resize:fit:700/1*Pbt3svrlB0KGlkc7DNcBzA.png)

- The network is now created as seen from the screenshot above, however, if we inspect the network we will observe that there are no containers attached to this network.

# **Step 5: Add both MySQL and Flask app containers to the newly created network.**

- To achieve this goal, edit the run commands for both MySql and FLask to include `-network <name of newly created network>` , as shown below:

> Destroy the previously created MySQL and flask-app containers $ docker kill <container id> , then docker rm <container id>, otherwise you will get an error message when containers are relaunch as ports are in use.
> 
- Command to relaunch MySql database with — network syntax

$ `docker run -d -p 3306:3306 --mount source=mysql-volume,target=/var/lib/mysql --name mysql --network two-tier-app-nw -e MYSQL_ROOT_PASSWORD=<your password> -e MYSQL_DATABASE=testdb -e MYSQL_USER=admin -e MYSQL_PASSWORD=<yourpassword> mysql:latest`

- $ `docker inspect <network name>` - check if the mysql container has been added to the new network created.

And it has indeed been added!!

![](https://miro.medium.com/v2/resize:fit:700/1*10fxImbdigYuE6gDGp-klQ.png)

- Command to relaunch Flask app container with — network syntax.

$ `docker run -d -p 5000:5000 --name flask-app --network two-tier-app-nw -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e MYSQL_PASSWORD=<yourpassword> -e MYSQL_DB=testdb flask-app:latest`

- Inspect that the flask app container has been launched in the new network.

$ `docker inspect <network name>`

![](https://miro.medium.com/v2/resize:fit:700/1*si469N-zBkr_9yXYVlA9ew.png)

- At this point, both the front-end and back-end containers have been launched in the same network.
- Take your ec2 IP address:5000 and run this URL in your browser.
- 🙌 Congratulations!! Your app is up and running 🎉

![](https://miro.medium.com/v2/resize:fit:700/1*wBwxTiddFHMajKsRLjscuw.png)

- Enter a few messages and check if your database table is updated:

![](https://miro.medium.com/v2/resize:fit:700/1*MgbJ-8EsHevOdGpmhnRSAQ.png)

- log into your MySQL container.

$ `docker exec -it <mysql container id>`

- Run the following commands inside your container.

$ `mysql -u root -p` -login to MySQL database

$ `show database;` -list the databases

$ `use testdb`-select the database you want to use

$ `select * from messages;` -displays all the messages populating your database

![](https://miro.medium.com/v2/resize:fit:700/1*fYIli4qFfEcX6Jdcojw6Fw.png)

- The input data has been effectively saved in the database, as evident from the displayed tables.

*Ladies and Gentlemen, there you have it! You’ve successfully crafted your two-tier architecture featuring a Flask app (front-end) and a MySQL database (back-end), all seamlessly containerized with Docker.*
