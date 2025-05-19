# Build stage
FROM eclipse-temurin:21-jdk-alpine AS builder
ENV APP_HOME=/app
WORKDIR $APP_HOME
RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Run stage
FROM eclipse-temurin:21-jre-alpine
ENV APP_HOME=/app
WORKDIR $APP_HOME

COPY --from=builder $APP_HOME/target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
