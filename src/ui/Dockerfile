# =========================
# Stage 1: Build
# =========================
FROM amazoncorretto:17-alpine AS build-env

# Set working directory
WORKDIR /app

# Copy Maven wrapper and make sure it's executable
COPY mvnw .
COPY .mvn .mvn
RUN chmod +x mvnw

# Copy pom.xml first to leverage Docker cache
COPY pom.xml .

# Download all dependencies offline
RUN ./mvnw dependency:go-offline -B -q

# Copy source code
COPY src ./src

# Build the project
RUN ./mvnw clean package -B -DskipTests

# =========================
# Stage 2: Runtime
# =========================
FROM amazoncorretto:17-alpine

# Set working directory
WORKDIR /app

# Copy the built jar from the previous stage
COPY --from=build-env /app/target/*.jar app.jar

# Expose port if needed
EXPOSE 8080

# Run the application
ENTRYPOINT ["java","-jar","app.jar"]
