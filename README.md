# front-final-2026

Frontend del trabajo final 2026. Está hecho con React 19, TypeScript y Vite. Consume la API del repositorio `back-final-2026`.

Esta guía permite levantar el frontend desde cero. Hay dos formas:

- **Opción A: modo desarrollo** (Vite).
- **Opción B: Docker** (build estático servido con Nginx).

> **Antes de empezar:** el backend tiene que estar levantado. Seguí el README de `back-final-2026` y anotá la URL donde queda disponible (por ejemplo `http://localhost:3000`).

---

## 1. Requisitos

| Herramienta | Opción A (desarrollo) | Opción B (Docker) |
|---|---|---|
| Git | Sí | Sí |
| Node.js 22 o superior y npm | Sí | No |
| Docker | No | Sí |

## 2. Clonar el repositorio

```bash
git clone https://github.com/ricardoherbas/frontend-f-2026.git front-final-2026
cd front-final-2026
```

## 3. Variable de entorno `VITE_API_URL`

| Variable | Obligatoria | Descripción |
|---|---|---|
| `VITE_API_URL` | Sí (ver nota) | **URL donde está corriendo el backend**, sin `/api` al final. El frontend le agrega `/api` automáticamente. |

**`VITE_API_URL` debe apuntar a la URL donde esté corriendo el backend.** Esa URL se abre desde el navegador de quien usa la aplicación, así que tiene que ser accesible desde ese navegador.

Ejemplos:

| Dónde corre el backend | Valor de `VITE_API_URL` |
|---|---|
| En tu misma máquina, puerto 3000 | `http://localhost:3000` |
| En otra máquina de la red | `http://192.168.1.50:3000` |
| En un servidor con dominio | `https://api.midominio.com` |

Reglas:

- Escribila **sin barra final** y **sin `/api`**. Correcto: `http://localhost:3000`. Incorrecto: `http://localhost:3000/api` o `http://localhost:3000/`.
- Si no se define, el frontend usa `http://localhost:3000` por defecto.
- Vite lee la variable **al iniciar o al compilar**. Si la cambiás, reiniciá `npm run dev` o volvé a construir la imagen.

El backend también necesita conocer la URL del frontend en su variable `FRONTEND_URL`. Se usa para armar el enlace de recuperación de contraseña. Debe coincidir con la URL donde abras este frontend (por ejemplo `http://localhost:5173` en modo desarrollo).

---

## 4. Levantar el proyecto

### Opción A: modo desarrollo

**4.A.1. Instalar dependencias**

```bash
npm install
```

(El repositorio incluye un `.npmrc` con `legacy-peer-deps=true`; se aplica solo.)

**4.A.2. Crear el archivo `.env`**

```bash
cp .env.example .env
```

Editá `.env` y poné la URL de tu backend:

```env
VITE_API_URL=http://localhost:3000
```

**4.A.3. Iniciar el servidor de desarrollo**

```bash
npm run dev
```

Abrí en el navegador la dirección que muestra la consola, normalmente **http://localhost:5173**.

### Opción B: con Docker

El `Dockerfile` compila la aplicación (Node 22) y la sirve con Nginx en el puerto 80 del contenedor.

**4.B.1. Construir la imagen**

```bash
docker build -t front-final-2026 .
```

**4.B.2. Ejecutar el contenedor**

```bash
docker run -d --name front_final_app -p 5173:80 front-final-2026
```

Abrí **http://localhost:5173** (podés cambiar `5173` por otro puerto libre de tu máquina).

Para detenerlo y eliminarlo:

```bash
docker stop front_final_app && docker rm front_final_app
```

> **Limitación actual con Docker:** `VITE_API_URL` se incorpora a la aplicación en el momento de **compilar**, y el `Dockerfile` no recibe esa variable (además, `.dockerignore` excluye el `.env`). Por eso la imagen siempre apunta a `http://localhost:3000`.
> Esto funciona si el backend corre en la misma máquina, en el puerto `3000`. Si el backend está en otra URL, usá la **Opción A** o pedí que se ajuste el `Dockerfile` para aceptar `VITE_API_URL` como argumento de build.

---

## 5. Probar que funciona

1. Con el backend levantado, abrí el frontend en el navegador.
2. Entrá a `/login`.
3. Iniciá sesión con el **email y la contraseña del administrador inicial**, que se definen en `ADMIN_EMAIL` y `ADMIN_PASSWORD` del `.env` de `back-final-2026` (el administrador se crea con `npm run seed`; ver ese README).

Si el login responde con error de red, revisá el paso 6.

## 6. Problemas frecuentes

| Síntoma | Causa y solución |
|---|---|
| Error de red o `Network Error` al iniciar sesión | El backend no está levantado, o `VITE_API_URL` no apunta a él. Verificá la URL abriéndola en el navegador y reiniciá `npm run dev` tras cambiar el `.env`. |
| Las peticiones van a `.../api/api/...` | `VITE_API_URL` termina en `/api`. Quitalo. |
| Cambié `.env` y no se nota | Vite lee el `.env` al iniciar. Detené y volvé a ejecutar `npm run dev` (o reconstruí la imagen). |
| El enlace de recuperación de contraseña del correo no abre esta aplicación | `FRONTEND_URL` del backend no coincide con la URL real del frontend. |
| `401 Email o contraseña incorrectos` | El administrador no fue creado (falta `npm run seed` en el backend) o las credenciales no coinciden con `ADMIN_*`. |
| Al recargar una ruta aparece 404 | En Docker, `nginx.conf` ya redirige todo a `index.html`. En otros hostings, configurá esa redirección (en Vercel ya está en `vercel.json`). |
| `port is already allocated` | Usá otro puerto en `-p <puerto>:80`. |

## 7. Otros comandos

```bash
npm run build       # chequea tipos y genera la versión de producción en dist/
npm run preview     # sirve dist/ localmente para probar la compilación
npm run typecheck   # revisa los tipos con TypeScript
npm run lint        # revisa el código con ESLint
```