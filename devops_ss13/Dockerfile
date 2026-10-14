# syntax=docker/dockerfile:1

# ---------------------------------------------------------------
# Stage 1: build backend với Gradle (JDK 21)
# ---------------------------------------------------------------
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /workspace

# Copy wrapper + build script trước để cache lớp dependency
COPY backend/gradlew backend/settings.gradle backend/build.gradle ./
COPY backend/gradle ./gradle

# gradlew trên Windows có thể có CRLF, chuẩn hóa về LF trước khi chạy
RUN sed -i 's/\r$//' gradlew && chmod +x gradlew

# Copy source code
COPY backend/src ./src

# Build executable jar (tự tải dependency khi cần)
RUN ./gradlew bootJar --no-daemon --console=plain

# ---------------------------------------------------------------
# Stage 2: runtime image gọn nhẹ (JRE 21)
# ---------------------------------------------------------------
FROM eclipse-temurin:21-jre AS runtime

RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/* \
    && useradd --system --no-create-home --shell /usr/sbin/nologin app

WORKDIR /app

# Lấy jar đóng gói (bỏ qua jar "plain" nếu Gradle tạo ra)
COPY --from=builder /workspace/build/libs/ /tmp/libs/
RUN JAR=$(ls /tmp/libs/*.jar | grep -v -- '-plain\.jar$' | head -n 1) \
    && cp "$JAR" /app/app.jar \
    && rm -rf /tmp/libs \
    && chown app:app /app/app.jar

USER app

EXPOSE 8080

HEALTHCHECK --interval=15s --timeout=5s --start-period=60s --retries=5 \
    CMD curl -fsS http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
