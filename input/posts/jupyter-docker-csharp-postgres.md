Title: Docker controlled from Jupyter Notebook C# with PostgresDB
Published: 12/11/2019 23:18
Tags: [CSharp, Postgresql, Docker, Dotnet try, Jupyter notebook, Database] 
---

In the context of Docker and Jupyter Notebook, it's interesting to note that there exists a Nuget that allows C# to control docker. So, yes, it is possible to launch a Postgresql database, on docker, inside a Jupyter notebook!

This assumes you have [Docker](https://hub.docker.com/?overlay=onboarding), [Dotnet try](https://github.com/dotnet/try/blob/master/DotNetTryLocal.md), [Jupyter notebook](https://jupyter.org/) and follow the setup of the [C# kernel for Jupyter](/posts/jupyter-notebook-csharp-r). 

If you don't want to wait, you can find my [complete notebook here](https://github.com/ewinnington/noteb/blob/master/DockerInteraction.ipynb).

Microsoft has created a [C# Client library for talking to Docker](https://github.com/microsoft/Docker.DotNet), so we will be taking advantage of it. Much of the magic docker code is pulled from the Docker.DotNet repository.
I'm using the [Npgsql drivers](https://www.npgsql.org/index.html) for accessing the PostgreSQL database. 
![pgsql01](/posts/images/jupyter-notebook-csharp-r/docker-pg-01.png)

The real magic moment is when you access the Docker instance, if it is on your local machine on windows, you can use the ```npipe://./pipe/docker_engine``` Uri. If you are on Linux, use ```unix:///var/run/docker.sock``` (at this time, I haven't tried it on linux, but if you do, please tell me). 
![pgsql02](/posts/images/jupyter-notebook-csharp-r/docker-pg-02.png)

In block 3, we select a random port to host the pgSQL database. Then list the local image names that are available (you should get postgres:latest on your machine to run this). We create and start up the container, passing the environment variables for the password, user and initial schema. Once the container is started, we detach ourselves from it, so it runs in the background. Finally, I wait until I'm pretty sure the container and database is ready (10s sleep at the end). You can reduce that sleep time. On my Surface laptop 1, I sometimes have an issue when I've got too many other containers running.
![pgsql03](/posts/images/jupyter-notebook-csharp-r/docker-pg-03.png)

I'm connecting to the database and validating the connection name. 
![pgsql04](/posts/images/jupyter-notebook-csharp-r/docker-pg-04.png)

I'm creating a database schema and a user for this particular schema, then reconning with the new user. 
![pgsql05](/posts/images/jupyter-notebook-csharp-r/docker-pg-05.png)

Creating a table and inserting two rows using direct strings and string concatenation. In production, you should never be using string concatenation for your SQL statements. Please always use Bound variables as described below. 
![pgsql06](/posts/images/jupyter-notebook-csharp-r/docker-pg-06.png)

This is how you should interact with the PostgreSQL database if you are using direct SQL statements. You should be using Bound commands with parameters.
![pgsql07](/posts/images/jupyter-notebook-csharp-r/docker-pg-07.png)

Finally, I check that all three insertions were successful. 
![pgsql08](/posts/images/jupyter-notebook-csharp-r/docker-pg-08.png)

You can watch the container run in the [docker extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) of [Visual studio Code](https://code.visualstudio.com/). It's a great way of monitoring what is currently running, as well as deleting old containers that might be still present. 
![pgsql10](/posts/images/jupyter-notebook-csharp-r/docker-pg-10.png)

Talking of deleting old containers, this is how you shut them down and delete them at the end of the notebook. I first close the Db connection and dispose of it before asking Docker.DotNet to stop the containers.
![pgsql09](/posts/images/jupyter-notebook-csharp-r/docker-pg-09.png)

It would be cleaner if I knew how to enfore a ```Finally``` in Jupyter Notebook, but at this time, I don't know. If you do, drop me a line on [twitter](https://twitter.com/ThrowATwit) or a pull request on this blog post.