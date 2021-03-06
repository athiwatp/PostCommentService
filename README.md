# Creating a Application .NetCore 2.2 Web Api + SQL Server with Docker Compose

## Description

The main propose of this article is show how to take advantaje in docker in order to use a basic .Net Core 2.2 enviroment. Creating a ASP.Net Core Web API with access to Linux SQL Server. All the technologies used in this article are full open-source.

### The Application

It is application focoused on blog posts and comments publications. All the concerns here are regarding a backend development.

## Docker Images

The first concern on dockerizing a application is to create a docker image for your application. We´ll build two containers, one for our application and other for the database. The database image will be pulled from the cloud, and our only concern it is to pay attention with the configuration.

### The Docker Image File

https://github.com/rodrigostrj/PostCommentService/blob/master/src/PostComment/Dockerfile

```yaml
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build-env
WORKDIR /app

COPY . ./
RUN dotnet publish PostComment.Api -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.2
WORKDIR /app
COPY --from=build-env /app/PostComment.Api/out .

ENTRYPOINT ["dotnet", "PostComment.Api.dll"]
```

### Creating a Docker Image

**In the same folder**

$ docker build -t postcomment.api .

**Parsing the docker compose file as parameter**

$ docker build -f [File Path Here] -t postcomment.api .  

### Running the image as a Container

#### .Net Core 
$ docker run -d -p 8080:80 --name myapp postcomment.api

#### SQL Server 

##### Atention to the enviroment variables "-e"

docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=yourStrong(!)Password' -e 'MSSQL_DB=postcomment_db' -e 'MSSQL_USER=yourDbUser' -e 'MSSQL_PASSWORD=yourDbPassword' -p 1433:1433 -d mcmoe/mssqldocker:v2017.CU12.1

## Docker Compose

A Docker Compose it is a yaml file with instructions to create containers. It is possible to create a full application. it is easier, at least in my opinion, work in this way instead using docker run for each expected application. 
By executing one command line it is possible to perform all the previus commands even the build command for application.

## Docker Compose File

https://github.com/rodrigostrj/PostCommentService/blob/master/src/PostComment/docker-compose.yml

```yaml
version: '3'

volumes:
   dbdata:

services:
    db:
        image: mcmoe/mssqldocker:v2017.CU12.1
        environment:
          ACCEPT_EULA: Y
          SA_PASSWORD: 1p@sswordY
          MSSQL_DB: postcommentdb
          MSSQL_USER: postcommentdb_app
          MSSQL_PASSWORD: 1p@sswordY
        ports:
            - "1433:1433"

    postcomment.api:
        depends_on:
           - db
        image: postcomment.api
        build:
          context: .
        ports:
          - "8080:80"
        environment:
          - ASPNETCORE_ENVIRONMENT=staging
```

### Creating Docker Containers With Docker Compose

#### In the same  folder
$ docker-compose up -d 

#### Parsing file as parameter 
$ docker-compose -f [File Path Here] up -d 

#### Forcing container recreation 
$ docker-compose up -d --force-recreate 

## Testing the application 

After all it is easy to access the application and the data base created.

http://localhost:8080/swagger/index.html

![Api Swagger Page](https://github.com/rodrigostrj/PostCommentService/blob/master/miscellaneous/PostComent.png)



