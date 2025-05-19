# Use lightweight Java runtime only (no JDK needed)
FROM openjdk:21-jdk-slim

# Set working directory
WORKDIR /app

# Copy compiled class file
COPY Main.class .

# Run the program
CMD ["java", "Main"]
