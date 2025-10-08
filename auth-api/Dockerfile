# ---------- Etapa 1: Build ----------
FROM golang:1.25.1 AS builder
WORKDIR /app

# Copiar módulos para cachear dependencias
COPY go.mod go.sum* ./
RUN go mod tidy

# Copiar el resto del código
COPY . .

# Compilar binario estático para Linux (compatible con Alpine)
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o auth-api .

# ---------- Etapa 2: Runtime ----------
FROM alpine:3.18 AS runtime
WORKDIR /app

# Instalar ca-certificates por si tu app hace HTTPS
RUN apk add --no-cache ca-certificates

# Copiar binario desde la etapa de build
COPY --from=builder /app/auth-api .

# Asegurarse que el binario sea ejecutable
RUN chmod +x auth-api

# Exponer puerto de la aplicación
EXPOSE 8080

# Ejecutar la aplicación
ENTRYPOINT ["./auth-api"]

