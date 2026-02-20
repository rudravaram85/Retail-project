# ---------- Stage 1: Build the application ----------
FROM amazonlinux:2023.0.2 AS build-env

# Install dependencies
RUN dnf -y update && \
    dnf -y install java-17-amazon-corretto-devel git tar unzip curl && \
    dnf clean all

# Set working directory
WORKDIR /app

# Copy Maven wrapper and pom
COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .

# Make Maven wrapper executable
RUN chmod +x mvnw

# Download all dependencies offline
RUN ./mvnw dependency:go-offline -B -q

# Copy source code and build the application
COPY src ./src
RUN ./mvnw package -DskipTests -B -q

# ---------- Stage 2: Create final runtime image ----------
FROM amazonlinux:2023.0.2

# Install minimal runtime dependencies
RUN dnf -y install java-17-amazon-corretto-headless && \
    dnf clean all

WORKDIR /app

# Copy the built jar from the build stage
COPY --from=build-env /app/target/*.jar app.jar

# Expose application port (change if needed)
EXPOSE 8080

# Run the application
ENTRYPOINT ["java","-jar","app.jar"]
