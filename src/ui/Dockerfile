# ========================
# Stage 1: Build
# ========================
FROM amazonlinux:2023 as build-env

# Install necessary tools: Java, Maven, curl, unzip, git
RUN dnf -y update && \
    dnf -y install java-17-amazon-corretto-devel maven curl git unzip && \
    dnf clean all

# Set working directory
WORKDIR /app

# Copy only pom.xml first to leverage Docker cache
COPY pom.xml .

# Download dependencies offline
COPY . .
RUN chmod +x ./mvnw
RUN ./mvnw dependency:go-offline -B -q
# Copy source code
COPY src ./src

# Build project (skip tests for faster build)
RUN mvn package -DskipTests -B

# ========================
# Stage 2: Runtime
# ========================
FROM amazonlinux:2023

# Install Java runtime only
RUN dnf -y install java-17-amazon-corretto-headless && dnf clean all

WORKDIR /app

# Copy built JAR from build stage
COPY --from=build-env /app/target/*.jar ./app.jar

# Run the application
ENTRYPOINT ["java","-jar","app.jar"]
