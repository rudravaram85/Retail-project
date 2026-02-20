# ========================
# Stage 1: Build
# ========================
FROM amazonlinux:2023 as build-env

RUN dnf -y update && \
    dnf -y install java-17-amazon-corretto-devel maven curl git unzip && \
    dnf clean all

WORKDIR /app

# Copy pom.xml and mvnw first
COPY pom.xml .
COPY mvnw .

# Make mvnw executable BEFORE copying everything else
RUN chmod +x ./mvnw

# Copy everything else
COPY . .

# Download dependencies offline
RUN ./mvnw dependency:go-offline -B -q

# Build project (skip tests for faster build)
RUN ./mvnw package -DskipTests -B

# ========================
# Stage 2: Runtime
# ========================
FROM amazonlinux:2023

RUN dnf -y install java-17-amazon-corretto-headless && dnf clean all

WORKDIR /app

COPY --from=build-env /app/target/*.jar ./app.jar

ENTRYPOINT ["java","-jar","app.jar"]