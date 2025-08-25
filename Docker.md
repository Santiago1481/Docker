# 🐳 Docker: Resumen Completo

## ¿Qué es Docker?

Docker es una plataforma que permite empaquetar aplicaciones y sus dependencias en contenedores. Estos contenedores son portables, ligeros y reproducibles, lo que garantiza que tu app se ejecute igual en cualquier entorno: desarrollo, staging o producción.

---

## 🚀 ¿Por qué usar Docker?

- Elimina el clásico “en mi máquina sí funciona”.
- Aísla servicios para evitar conflictos entre dependencias.
- Facilita el despliegue en servidores o servicios cloud.
- Ideal para entornos CI/CD y desarrollo colaborativo.

---

## ⚙️ ¿Cómo se usa Docker?

### 1. Instalar Docker

Descarga Docker Desktop desde [docker.com](https://www.docker.com) y sigue los pasos para tu sistema operativo.

### 2. Crear un Dockerfile

Ejemplo para una app Node.js:

```Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

### 🛠️ Construir la imagen
Para construir la imagen a partir del Dockerfile:

```bash
docker build -t mi-app .
```
Esto crea una imagen llamada **mi-app**.

### ▶️ Ejecutar el contenedor
Para correr la imagen en un contenedor:

```bash
docker run -p 3000:3000 mi-app
```
Esto expone el puerto **3000** del contenedor en tu máquina local.

### 🧩 Docker Compose
Si tu app necesita varios servicios (por ejemplo, backend + base de datos), usa `docker-compose.yml` para orquestarlos:

```yaml
version: '3'
services:
  backend:
    build: .
    ports:
      - "3000:3000"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: ejemplo
```

Para levantar los servicios:

```bash
docker-compose up
```

Y para detenerlos:

```bash
docker-compose down
```

### 📦 Comandos útiles

| Comando | Descripción |
|---------|-------------|
| `docker ps` | Ver contenedores activos |
| `docker images` | Ver imágenes disponibles |
| `docker stop <id>` | Detener un contenedor |
| `docker rm <id>` | Eliminar un contenedor detenido |
| `docker rmi <imagen>` | Eliminar una imagen |
| `docker logs <id>` | Ver logs del contenedor |
| `docker-compose up` | Levantar servicios definidos |
| `docker-compose down` | Detener y eliminar servicios |

---

## 🧠 Buenas prácticas

- Usa `.dockerignore` para evitar copiar archivos innecesarios.
- Mantén tus imágenes ligeras usando imágenes base **slim** o **alpine**.
- Versiona tu Dockerfile junto con tu código.
- Usa **etiquetas (tags)** para controlar versiones de tus imágenes.
- Evita instalar dependencias innecesarias en producción.
- Revisa los logs con `docker logs <id>` para depurar errores.

---

## 📚 Recursos recomendados

- [Documentación oficial de Docker](https://docs.docker.com/)
- [Play with Docker (sandbox online)](https://labs.play-with-docker.com/)
- [Docker en GitHub](https://github.com/docker)
- [Docker Compose Docs](https://docs.docker.com/compose/)

---

Este archivo está diseñado para ayudarte a comenzar rápido con Docker.  
Si estás usando **React Native + Expo** o backend con **TypeScript**, adapta el Dockerfile y `docker-compose.yml` a tu stack específico.
