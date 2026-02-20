# -------------------------------
# Build Stage
# -------------------------------
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

# Install minimal packages
RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    which \
    tar \
    gzip \
    && dnf clean all

VOLUME /tmp
WORKDIR /build

# Copy project files
COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .

# Make mvnw executable
RUN chmod +x mvnw

# Pre-download dependencies
RUN ./mvnw dependency:go-offline -B -q

# Copy source code
COPY ./src ./src

# Build the jar
RUN ./mvnw -DskipTests package -q \
    && mv /build/target/ui-0.0.1-SNAPSHOT.jar /build/app.jar

# -------------------------------
# Package Stage
# -------------------------------
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

# Install runtime dependencies
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    && dnf clean all

# Ensure full curl for telnet support
RUN dnf -q -y swap libcurl-minimal libcurl-full \
    && dnf -q -y swap curl-minimal curl-full

# Create unprivileged user
ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000
RUN useradd --home "/app" --create-home --user-group --uid "$APPUID" "$APPUSER"

ENV JAVA_TOOL_OPTIONS=
ENV SPRING_PROFILES_ACTIVE=prod

WORKDIR /app
USER appuser

# Copy license files
COPY ./ATTRIBUTION.md ./LICENSES.md

# Copy built jar from build stage
COPY --chown=appuser:appuser --from=build-env /build/app.jar .

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
