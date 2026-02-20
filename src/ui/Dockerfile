# ---------- Build Stage ----------
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

# Install Java + Maven
RUN dnf --setopt=install_weak_deps=False install -y -q \
        maven \
        java-21-amazon-corretto-headless \
        which \
        tar \
        gzip \
    && dnf clean all

WORKDIR /build

# Copy pom and download dependencies first
COPY pom.xml .
RUN mvn dependency:go-offline -B -q

# Copy source code and build
COPY src ./src
RUN mvn -DskipTests package -q \
    && mv /build/target/ui-0.0.1-SNAPSHOT.jar /build/app.jar

# ---------- Final Stage ----------
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf --setopt=install_weak_deps=False install -y -q \
        java-21-amazon-corretto-headless \
        shadow-utils \
    && dnf clean all

WORKDIR /app

# Copy the application jar
COPY --from=build-env /build/app.jar .

EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
