# Use a smaller JRE base image
FROM eclipse-temurin:17-jre-alpine

# Set working directory
WORKDIR /app

# Copy only the JAR
COPY target/*.jar app.jar

# Optional: expose port (if your app runs on 8080)
EXPOSE 8080

# Run the app
ENTRYPOINT ["java", "-jar", "app.jar"]
