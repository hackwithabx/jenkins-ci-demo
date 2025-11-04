# Use OpenJDK
FROM eclipse-temurin:17-jdk-alpine

# Set working directory
WORKDIR /app

# Copy Maven project files
COPY pom.xml .
COPY src ./src

# Build the project
RUN mvn package -DskipTests

# Run the app
CMD ["java", "-cp", "target/classes", "com.abx.App"]
