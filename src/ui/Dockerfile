# ========================
# Stage 1: Build
# ========================
FROM amazonlinux:2023 as build-env

# Install Java, Maven, curl, git, unzip
RUN dnf -y update && \
    dnf -y install java-17-amazon-corretto-devel maven curl git unzip && \
    dnf clean all

WORKDIR /app

# Copy only Maven wrapper first
COPY mvnw .
COPY .mvn .mvn

# Make mvnw executable inside container
RUN chmod +x mvnw

# Copy pom.xml separately to leverage cache
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B -q

# Copy source code
COPY src ./src

# Build the project, skip tests
RUN ./mvnw package -DskipTests -B

# ========================
# Stage 2: Runtime
# ========================
FROM amazonlinux:2023

# Install only Java runtime
RUN dnf -y install java-17-amazon-corretto-headless && dnf clean all

WORKDIR /app

# Copy JAR from build stage
COPY --from=build-env /app/target/*.jar ./app.jar

# Run the application
ENTRYPOINT ["java","-jar","app.jar"]