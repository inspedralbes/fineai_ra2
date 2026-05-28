# FineAI: Gestor de Finanzas Personales PWA

FineAI es una Progressive Web App (PWA) premium diseñada para el control y optimización de finanzas personales de forma 100% local e interactiva.

El desarrollo del proyecto se ha guiado bajo la metodología de Specification-Driven Development (SDD), utilizando SPEC.md como la fuente de verdad absoluta del sistema.

---

## 1. Stack Tecnológico

* **Frontend**: Nuxt.js (Vue 3, Composition API, <script setup>).
* **Estilos**: CSS nativo con diseño fosc glassmorphic premium (obsidiana y neón).
* **PWA**: Módulo oficial @vite-pwa/nuxt para soporte de Service Workers, caché offline e instalabilidad.
* **Persistencia**: LocalStorage para un entorno de datos reactivo y 100% local en cliente.

---

## 2. Estructura del Proyecto

* **SPEC.md**: Especificación técnica y funcional del sistema (fuente de verdad).
* **PROCESS.md**: Bitácora del proceso de desarrollo asistido por IA, resolución de bugs de SSR y comparativa visual con Stitch.
* **app.vue**: Componente raíz de la interfaz gráfica y gestor del estado.
* **components/TransactionList.vue**: Lista de transacciones reactivas categorizadas con balances en tiempo real.

---

## 3. Guía de Instalación y Uso Local

Para poder probar y ejecutar este proyecto en tu entorno local, asegúrate de tener instalado Node.js (versión 18 o superior) y sigue estos pasos:

### 1. Clonar o descargar el repositorio
Descarga el código fuente en tu disco local y navega hasta la carpeta del proyecto.

### 2. Instalar dependencias
Ejecuta la instalación de todos los paquetes oficiales y configuraciones preparatorias de Nuxt utilizando npm:
```bash
npm install
```

### 3. Ejecutar en entorno de desarrollo local
Inicia el servidor local de desarrollo de Nuxt para visualizar e interactuar con la aplicación:
```bash
npm run dev
```
El servidor se iniciará en `http://localhost:3000`.

### 4. Compilar para producción (Probar PWA en local)
Si deseas comprobar el rendimiento optimizado, el Service Worker y el funcionamiento offline, compila y previsualiza la versión de producción:
```bash
npm run build
npm run preview
```

---

## 4. Instrucciones para el Despliegue en Vercel

FineAI está totalmente preparado para ser desplegado en la plataforma Vercel de manera directa y gratuita.

1. Regístrate o inicia sesión en [Vercel](https://vercel.com).
2. Conecta tu cuenta de GitHub con Vercel.
3. Haz clic en **Add New -> Project** y selecciona el repositorio de github de este proyecto.
4. Vercel detectará automáticamente que se trata de un proyecto de Nuxt. Deja la configuración por defecto.
5. Haz clic en **Deploy**. 

Vercel compilará la aplicación PWA y la desplegará en la nube de manera automática en menos de un minuto, proporcionándote una URL pública segura (HTTPS) necesaria para el correcto funcionamiento de las Progressive Web Apps.
