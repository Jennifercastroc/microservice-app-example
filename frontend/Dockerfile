# Imagen base con Node 8
FROM node:8.17.0

# Directorio de trabajo dentro del contenedor
WORKDIR /app

# Copiar dependencias
COPY package*.json ./
RUN npm install

# Copiar el resto del código
COPY . .

# Exponer el puerto que usa el servidor de desarrollo
EXPOSE 8080

# Comando para correr el frontend
CMD ["npm", "run", "dev"]