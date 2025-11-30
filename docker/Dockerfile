# Use an official OpenJDK runtime as a parent image
FROM openjdk:17

# Set the working directory inside the container
WORKDIR /app

# Copy the built jar into the container
COPY target/student-management-0.0.1-SNAPSHOT.jar app.jar

# Expose the port the app runs on
EXPOSE 8089

# Run the jar file
ENTRYPOINT ["java","-jar","app.jar"]
