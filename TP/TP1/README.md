### 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?

This way you don't write information like the password in the Dockerfile allowing for better security. And also writting the variables in the Dockerfile doesn't allow to create different users for example so we have less flexibility.

### 1-2 Why do we need a volume to be attached to our postgres container?

A volume allows to save locally the data from the container. This can be usefull if the container needs to be rebuilt for any reason without losing its content.

### 1-3 Document your database container essentials: commands and Dockerfile.

#### Dockerfile
```
FROM postgres:17.2-alpine


COPY CreateScheme.sql /docker-entrypoint-initdb.d/
COPY InsertData.sql /docker-entrypoint-initdb.d/
```
#### Then for the commands:
- Create the network
```
docker network create app-network
```
- Create the image from the Dockerfile
```
docker build -t my-postgres-image .
```
- Create the Postgres container with a volume
```
docker run -d --name my-postgres-container --network app-network -e POSTGRES_DB=mydb -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword -v "C:\Users\dbouq\OneDrive - Fondation EPF\4A\DevOps\DevOps\TP\TP1\Postgres\data:/var/lib/postgresql/data" my-postgres-image
```
- Create the Adminer container
```
docker run -p "8090:8080" --net=app-network --name=adminer  -d adminer
```
 
### 1-4 Why do we need a multistage build? And explain each step of this dockerfile.

A multistage build allows to only keep what is needed to run the app. For example once the code is compiled, there is no need to keep the tools of the JDK in the image produced by a Dockerfile.

#### Dockerfile
```
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run stage
FROM eclipse-temurin:21-jre-alpine
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

The first part is used to import tools like jdk21 and Maven to compile the code and create a .jar with the compiled code:
```
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests
```
Then in the second part a JRE is imported that only has the minimum requierments to run the app. Then, only the .jar and JRE are copied to the container.
```
# Run stage
FROM eclipse-temurin:21-jre-alpine
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT ["java", "-jar", "myapp.jar"]
```