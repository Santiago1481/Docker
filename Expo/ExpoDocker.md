# 🚀 React Native + Expo con Docker (Paso a paso)

Guía práctica para desarrollar **apps Expo** dentro de un contenedor Docker con autorecarga, acceso desde el celular y sin instalar Node.js en tu máquina anfitriona.

> Funciona en **Windows (Docker Desktop + WSL2)**, **macOS** y **Linux**. Para el uso en dispositivos móviles, se recomienda iniciar Expo en **modo túnel** para evitar problemas de red locales.

---

## 🧰 Requisitos

- **Docker Desktop** instalado y corriendo.
- Un **teléfono** con **Expo Go** instalado (Android o iOS).
- (Opcional) Cuenta de **Expo** iniciada en Expo Go para un emparejamiento más estable.
- Puertos abiertos si usas LAN (ver más abajo). Con **túnel**, no necesitas abrir puertos en tu red.

> Si estás en Windows, usa **WSL2** y ten el proyecto dentro del sistema de archivos de WSL (`\\wsl$`) para mejor rendimiento.

---

## 📁 Estructura recomendada del proyecto

```
mi-app-expo/
├─ app/                # Rutas (Expo Router) o src/ si lo prefieres
├─ package.json
├─ .dockerignore
├─ Dockerfile
└─ docker-compose.yml
```

> Si aún **no tienes proyecto**, más abajo verás cómo crearlo desde cero *dentro* del contenedor.

---

## 🧾 .dockerignore

Crea un archivo **.dockerignore** en la raíz del proyecto:

```
node_modules
.expo
.expo-shared
npm-debug.log
yarn.lock.*
pnpm-lock.yaml*
.git
.gitignore
dist
build
```

---

## 🐳 Dockerfile

Usaremos una imagen ligera y variables que mejoran el live-reload dentro de contenedores (especialmente en Windows/macOS).

```Dockerfile
# Dockerfile
FROM node:20-alpine

# Dependencias útiles para desarrollo
RUN apk add --no-cache bash git

# Mejora de detección de cambios de archivos en volúmenes montados
ENV CHOKIDAR_USEPOLLING=true
ENV WATCHPACK_POLLING=true
# Permite que los servidores de desarrollo acepten conexiones externas
ENV EXPO_DEV_SERVER_LISTEN_HOST=0.0.0.0

# Directorio de trabajo
WORKDIR /app

# Copiamos sólo package.json/lock para aprovechar la cache de Docker
COPY package*.json ./

# Instala dependencias (si el proyecto ya existe)
# Si todavía no tienes proyecto, este paso se saltará o fallará
RUN if [ -f package.json ]; then npm install; fi

# Copia el resto del código (si ya existe)
COPY . .

# Exponemos puertos comunes usados por Expo y Metro
# Nota: según tu SDK, Expo/Metro puede usar 8081 y/o 19000-19006
EXPOSE 8081
EXPOSE 19000-19006

# Comando por defecto (se puede sobrescribir en docker-compose)
CMD ["npm", "run", "start:docker"]
```

---

## 🧩 docker-compose.yml

Define un servicio de desarrollo con volúmenes para el código y un volumen nombrado para `node_modules` (evita fricciones entre host y contenedor).

```yaml
# docker-compose.yml
version: "3.9"

services:
  app:
    build: .
    container_name: expo-dev
    working_dir: /app
    # Monta el código del host dentro del contenedor
    volumes:
      - .:/app
      # Mantén node_modules *dentro* del contenedor (mejor compatibilidad)
      - node_modules:/app/node_modules
    # Puertos: si usas LAN, expón; con túnel no es obligatorio
    ports:
      - "8081:8081"
      - "19000:19000"
      - "19001:19001"
      - "19002:19002"
      - "19006:19006"
    environment:
      # Fuerza servidores a escuchar en todas las interfaces
      - EXPO_DEV_SERVER_LISTEN_HOST=0.0.0.0
      # Mejor detección de cambios
      - CHOKIDAR_USEPOLLING=true
      - WATCHPACK_POLLING=true
    command: bash -lc "npm run start:docker || npx expo start --tunnel --clear --non-interactive"
    stdin_open: true
    tty: true

volumes:
  node_modules:
```

---

## 📦 Scripts en package.json

Agrega estos scripts para un arranque consistente dentro del contenedor.

```jsonc
// package.json (fragmento)
{
  "scripts": {
    "start": "expo start",
    // Ideal para Docker: usa túnel para evitar problemas de red/LAN
    "start:docker": "expo start --tunnel --clear --non-interactive",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "doctor": "expo doctor"
  },
  "devDependencies": {
    "expo": "^51.0.0"
  }
}
```

> Ajusta la versión de **expo** según tu SDK. Si usas **Expo Router**, tendrás `expo-router` y la carpeta `app/`.

---

## 🟢 Opción A — Proyecto NUEVO dentro de Docker

1. **Crea carpeta y archivos base** (vacíos): `Dockerfile`, `.dockerignore`, `docker-compose.yml`.
2. **Inicia el servicio** con una shell temporal para crear el proyecto:
   ```bash
   docker compose run --rm app bash
   ```
3. **Dentro del contenedor**, crea el proyecto:
   ```bash
   npx create-expo-app@latest . --template blank
   npm install
   ```
   *(Opcional)* si prefieres TypeScript:
   ```bash
   npx create-expo-app@latest . --template
   # elige "blank (TypeScript)"
   ```
4. **Salir** de la shell (`exit`).
5. **Levantar el entorno de desarrollo**:
   ```bash
   docker compose up
   ```
6. Abre **Expo Go** en tu teléfono y **escanea el QR** que verás en la terminal (modo **túnel**).

---

## 🔵 Opción B — Proyecto EXISTENTE

1. Copia `Dockerfile`, `.dockerignore` y `docker-compose.yml` a la raíz del proyecto.
2. Instala dependencias **dentro** del contenedor:
   ```bash
   docker compose run --rm app npm install
   ```
3. Inicia el entorno:
   ```bash
   docker compose up
   ```
4. Abre **Expo Go** y escanea el **QR** (túnel) o conéctate por **LAN** si prefieres.

---

## 📲 Probar en tu teléfono

- **Expo Go** (Android/iOS) → Escanea el **QR** que imprime `expo start`.
- **Modo túnel** (recomendado en Docker): más estable en redes corporativas o con NAT/WSL.
- **Modo LAN** (opcional): asegura tener mapeados los puertos. Si falla la auto-detección:
  - Usa `expo start --lan` y verifica que el dispositivo y tu PC estén en la **misma red**.
  - Mantén mapeados `8081` y `19000-19006` (algunas versiones de Expo/Metro usan 8081, y Expo ha usado 19000-19006 históricamente para manifest/websocket/devtools).

---

## 🔄 Live Reload y performance

- Variables ya incluidas: `CHOKIDAR_USEPOLLING=true` y `WATCHPACK_POLLING=true` para que el watcher detecte cambios en volúmenes Docker (útil en Windows/macOS).
- Mantener `node_modules` **dentro** del contenedor (volumen nombrado) evita inconsistencias entre host y Linux.
- Si notas lentitud en Windows, coloca el repo dentro de **WSL2** (`\\wsl$`) y abre el proyecto desde allí.

---

## 🧪 Comandos útiles dentro del contenedor

```bash
# Abrir una shell dentro del servicio
docker compose exec app bash

# Instalar una dependencia
docker compose exec app npm install axios

# Ejecutar scripts de Expo
docker compose exec app npm run doctor
docker compose exec app npm run web
```

---

## 🧯 Solución de problemas

- **El teléfono no conecta por LAN** → Usa `--tunnel` (ya configurado en `start:docker`).  
- **No recarga al guardar archivos** → Asegúrate de tener `CHOKIDAR_USEPOLLING=true` y que el proyecto esté en WSL2 (Windows).  
- **Error de permisos en node_modules** → Borra el volumen y reinstala:
  ```bash
  docker compose down -v
  docker compose run --rm app npm install
  docker compose up
  ```
- **Cache corrupta / pantalla en blanco** → Limpia caché:
  ```bash
  docker compose exec app npx expo start -c --non-interactive
  ```
- **Quiero abrir el proyecto en Android Studio o compilar nativo** → Eso requiere **entorno nativo** (SDKs) y generalmente **no** se ejecuta dentro de este contenedor básico. Para eso usa `eas build` en la nube o prepara un contenedor más complejo con Android SDK.

---

## 🧱 Ejemplo mínimo de `package.json`

```jsonc
{
  "name": "mi-app-expo",
  "private": true,
  "version": "1.0.0",
  "main": "expo-router/entry",
  "scripts": {
    "start": "expo start",
    "start:docker": "expo start --tunnel --clear --non-interactive",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "doctor": "expo doctor"
  },
  "dependencies": {
    "expo": "^51.0.0",
    "react": "^18.2.0",
    "react-native": "0.73.0"
  }
}
```

> Si usas **Expo Router**, instala también `expo-router` y crea la carpeta `app/`.

---

## ✅ Resumen rápido (cheatsheet)

1. `docker compose run --rm app bash` → crear proyecto con `npx create-expo-app .`  
2. `docker compose up` → iniciar Expo (túnel).  
3. Edita tu código en el host; el contenedor recarga.  
4. Usa `docker compose exec app bash` para instalar librerías o ejecutar comandos.  
5. Si algo falla, limpia caché (`expo start -c`) o baja y sube el stack (`docker compose down && up`).

---

¿Quieres que lo deje **pre-configurado** para tu repo con Expo Router y TypeScript (tsconfig, ESLint, rutas, assets y un par de pantallas de ejemplo)? Puedo generarlo todo listo para clonar y correr.
