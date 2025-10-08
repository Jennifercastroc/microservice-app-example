# Stage 1: Build
FROM maven:3.9.3-eclipse-temurin-8 AS build
WORKDIR /app

# Copiar pom.xml y descargar dependencias (caché más eficiente)
COPY pom.xml .
RUN mvn dependency:go-offline

# Copiar código fuente y compilar
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:8-jre
WORKDIR /app

COPY --from=build /app/target/users-api-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]