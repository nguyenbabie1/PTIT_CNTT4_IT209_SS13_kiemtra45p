# Multi-stage Dockerfile for Campus Hub Backend
# Stage 1: Build stage
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app

# Copy gradle wrapper and configurations first to leverage Docker layer caching
COPY gradlew gradlew.bat settings.gradle build.gradle ./
COPY gradle gradle

# Grant execution rights on gradlew
RUN chmod +x gradlew

# Download dependencies
RUN ./gradlew dependencies --no-daemon || true

# Copy application source code
COPY src src

# Build executable Spring Boot JAR file without re-running tests
RUN ./gradlew bootJar -x test --no-daemon

# Stage 2: Minimal runtime image
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Run as non-root user for security best practices
RUN addgroup -S campushub && adduser -S campushub -G campushub

# Copy artifact from builder stage
COPY --from=builder /app/build/libs/campus-hub-*.jar app.jar

# Adjust ownership
RUN chown -R campushub:campushub /app

USER campushub

# Expose backend service port
EXPOSE 8080

# Environment variables with sensible defaults
ENV SERVER_PORT=8080 \
    DB_URL=jdbc:mysql://mysql:3306/campus_hub?createDatabaseIfNotExist=true&useUnicode=true&characterEncoding=UTF-8&allowPublicKeyRetrieval=true&useSSL=false \
    DB_USERNAME=root \
    DB_PASSWORD=rootpassword

# Healthcheck probe
HEALTHCHECK --interval=20s --timeout=5s --start-period=30s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
