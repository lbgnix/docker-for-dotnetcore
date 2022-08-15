#Building and Running ASP.NET Core 3.x Application Inside A Container without Database# -

1) In this task you will build ASP .NET Core 3.x application and then package and run it as a container. Change to the relevant directory ~/aspnetcore. First, we need to run dotnet build, and publish to generate the binaries for our application. This can be done manually or by leveraging a Dockerfile. In this example, we will run the commands manually to produce the artifacts in a folder called published. The Dockerfile will only contain instructions to copy the files from the published folder into the image.
```
dotnet build
dotnet publish -o published
```
2) Now that the application is ready, you will create your container image. The Dockerfile is provided to you. View the content of Dockerfile by running the nano Dockerfile command. To exit the editor press CTRL+X..

3) To create the container image run the command
```
docker build -t myaspcoreapp:3.1 .
```
Note- Notice the 3.1 tag representing the dotnet core framework version.

4) Launch the container running your application using the command
```
docker run -d -p 8090:80 myaspcoreapp:3.1
```
Note - You are now running ASP.NET Core application inside the container listening at port 80 which is mapped to port 8090 on the host.

5) To test the application, go to localhost:8090 in your Firefox browser.

Congratulations! 
You have successfully completed this practical .

Building and Running SQL Server 2017 in a Container - 

Microsoft SQL Server is one of the most commonly used database server in the market today. Microsoft has made an investment to ensure that customers moving towards containers have an ability to leverage SQL Server through a container image.
SQL Server 2017 is only available for Linux Containers and allow users to bring their own license key when starting the container.
https://hub.docker.com/_/microsoft-mssql-server


In this Practical you will work with Microsoft SQL Server 2017 container image to run a custom database that can store user related information. You will first learn how to associate relevant SQL Server database files to SQL Server container image and how to initialize the database with test data. Then, you will connect a Web Application packaged in another container to the database running inside the SQL Server container. In other words, you will end up with a web application running in a container talking to a database hosted in another container. This is very common scenario, so understanding how it works is important. Let's start by running a SQL Server Express container with the custom database.

SQL Server 2017 Container Image

1) Make sure you have a Terminal opened, and that you are logged in as root. Also, change the current directory to ~/sqlserver2017 by using the command
```
cd /sqlserver2017
```
2) Before proceeding further, let's remove all the containers from previous tasks. Run the command
```
docker rm $(docker ps -aq) -f
```
3) Look at the Dockerfile describing how to package the database
```
cat Dockerfile
```
From the Microsoft SQL Server image, we copy the local files to the container. These local files are composed of:
Users.csv - contains the test data
setup.sql - the SQL commands to create a database named LabData and the Users table
entrypoint.sh - used as an entry point in the Dockerfile. It will start the database server and run import-data.sh
import-data.sh - will wait for the server to start and will trigger the database creation, the data import and the ping command to keep the database alive. Feel free to look at each file we just described to have a better understanding of their role.

4) Run the command to build our SQL Server container image
```
docker build -t mysqlserver .
```
5) Once built, run the start your with the following command (note that we explicitly name our container)
```
docker run -e ACCEPT_EULA=Y -e SA_PASSWORD=P@ssw0rd123! -d -p 1433:1433 --name mydb mysqlserver
```

6) You can follow the database initialization with the command docker logs mydb -f until you see the ping command starting. Once it started, you can interrupt docker logs by hitting CTRL + C

7)Run the following command to open an interactive session within the database container with sqlcmd. Sqlcmd is a basic command-line utility provided by Microsoft https://docs.microsoft.com/en-us/sql/relationaldatabases/scripting/sqlcmd-use-the-utility for ad hoc, interactive execution of Transact-SQL statements and scripts
```
docker exec -it mydb /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P P@ssw0rd123! 
```
8) Let's begin by listing down all the databases available by running the command

```
sql
SELECT name FROM master.dbo.sysdatabases   
GO
```

Now let's check that the users we had in Users.csv have been properly ingested into the database at initialization

```
sql
USE LabData  
SELECT * FROM Users  
GO
```

Now let's exit the sqlcmd session. Note that the database will still be running in the background
```
exit
```

Connect Application to the SQL Server Container

Building and Running ASP.NET Core 3.x Application Inside A Container with MSSQL Database as container -

Now we will connect an ASP .NET Core Application to our SQL Server and show that it can see the data stored in the database. Change the current directory to ~/aspnetcorewithsqlserver by using the command

```
cd ~/aspnetcorewithsqlserver
```

First, we need to know what is the IP address of the container running the SQL Server so that the front end web application. Run the following command and note down the IP Address

```
docker inspect mydb
```

Now we need to update the connection string used by the web app to connect to the SQL Server back end. Open the Startup.cs and replace localhost in the connection string by the IP address we just copied. Once finish making changes press CTRL + X and then press Y when asked for confirmation to retain your changes. Finally, you will be asked for file name to write. For that press Enter (without changing the name of the file). This will close the nano text editor.

nano Startup.cs

We are now ready to build the ASP .NET Core web application. Run dotnet build to build it.
```
dotnet build
```

Then publish the artifacts in a published folder with
```
dotnet publish -o published
```

Now, we are ready to build our container that we will tag with an explicit withsql string
```
docker build -t myaspcoreapp:3.1-withmssql .
```
Finally, run the container and expose the web app on port 8082
```
docker run -d -p 8082:80 myaspcoreapp:3.1-withmssql
```

Open a browser and naviguate to localhost:8082. The web app should display the list of users that we ingested in the database.

We have reached the end of the this practical, now let's remove all the containers to leave the environment in a clean state. Run the command
```
docker rm $(docker ps -aq) -f
```
