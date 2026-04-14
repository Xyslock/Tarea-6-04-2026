# Entregable Actividad 2: Pipeline CI/CD con GitHub Actions

## Detalles del Desarrollo

- **Repositorio Base Consultado:** https://github.com/rasetsuvoid/prueba-actividad-2
- **Repositorio Propio de Trabajo:** https://github.com/Xyslock/Tarea-6-04-2026
- **Enlace a GitHub Pages:** https://Xyslock.github.io/Tarea-6-04-2026

---

## 1. Clonación y Adaptación del Proyecto
El proyecto inicial es una aplicación desarrollada en SvelteKit que hace uso de llamadas en el lado de servidor. Dado que *GitHub Pages* está diseñado exclusivamente para alojar **sitios web estáticos** y no ejecuta código de backend, fue necesario hacer una serie de ajustes previos a nivel de código para asegurar la compatibilidad:

1. **Instalación de `@sveltejs/adapter-static`:** 
   Se reemplazó la dependencia dinámica base (`@sveltejs/adapter-auto`) por el adaptador estático con el comando `npm install -D @sveltejs/adapter-static` para que al compilar, el proyecto pre-renderice los archivos como HTML/CSS/JS puros que GitHub Pages puede servir.
   
2. **Transformación a Client-Side Rendering (SPA):** 
   Se migró la lógica del backend (`+page.server.ts`) al frontal (`+page.ts`). Adicionalmente se le indico a SvelteKit trabajar en modo SPA (Single Page Application) creando un archivo `+layout.ts` con la instrucción `export const ssr = false;`.

---

## 2. Creación e Implementación del Pipeline de CI/CD

El pipeline, alojado en `.github/workflows/pipeline.yml`, automatiza todo el proceso de empaquetado (Build) y Publicación (Deploy). A continuación se explica su funcionamiento:

1. **Activación de los Eventos:**
   El pipeline se dispara automáticamente cada vez que se detecta un `push` a la rama `main` o `master`.

2. **Etapa 1: Build (Construcción)**
   - **Checkout del Código:** Utilizando la acción `actions/checkout@v4`, se trae una copia exacta del código actual para trabajar.
   - **Ambiente Node.js:** Mediante `actions/setup-node@v4` se configura Node.js versión 20.
   - **Instalación de Dependencias:** Se ejecuta `npm ci || npm install` para descargar todos los paquetes necesarios.
   - **Compilación del Proyecto:** Se ejecuta el script de `npm run build` definiendo la variable de entorno `BASE_PATH`, fundamental para que todos los enlaces a archivos estáticos (imágenes o estilos) apunten al subdirectorio correcto de GitHub Pages (`/Tarea-6-04-2026`).
   - **Subida de Artefacto:** Finalmente, la carpeta `/build` generada localmente es empaquetada como un artefacto y puesta a disposición por `actions/upload-pages-artifact@v3`.

3. **Etapa 2: Deploy (Despliegue)**
   - Este job depende directamente explícitamente de que la etapa de `Build` termine con éxito (`needs: build`).
   - Usa la acción oficial de GitHub `actions/deploy-pages@v4` para transferir los archivos recién generados hacia los servidores de GitHub Pages y publicar la página al instante en la URL referida al inicio.
