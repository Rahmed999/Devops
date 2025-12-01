FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/my-app.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
