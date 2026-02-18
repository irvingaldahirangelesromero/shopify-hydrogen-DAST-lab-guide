
# Guía de Laboratorio DAST: Wapiti + Shopify Hydrogen + Burp Suite

  

Esta guía proporciona instrucciones detalladas para levantar un entorno de laboratorio de pruebas de seguridad dinámicas (DAST). Aprenderás a configurar una tienda vulnerable con **Shopify Hydrogen**, escanearla automáticamente con **Wapiti** y analizar el tráfico manualmente con **Burp Suite**.

  

### 📋 Requisitos Previos Recomendados

| Herramienta | Obligatoria | Descripción |
| :--- | :---: | :--- |
| [Node.js](https://nodejs.org/) | si | Necesario para ejecutar el proyecto Hydrogen |
| [Python 3.11](https://www.python.org/) | si | Requerido para instalar y ejecutar Wapiti |
| [Live Server (VS Code)](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) | no | Opcional | Extensión de VS Code para servir archivos estáticos fácilmente. Útil para visualizar reportes HTML generados por Wapiti |

  

---
##  Parte 1: Instalación de Wapiti (Escáner DAST)

  

Wapiti es un escáner de vulnerabilidades web basado en terminal que audita la seguridad de tus aplicaciones web buscando fallos como XSS, SQL/XPath Injections, File Inclusions, etc.

  

### 1. Instalar Python 3.11

Wapiti funciona mejor con Python 3.x. Usaremos `winget` para una instalación limpia.

  

```powershell

winget install --id Python.Python.3.11 --source winget

```

> [!NOTE]

>  **Reinicia tu terminal** después de la instalación para asegurar que las variables de entorno se actualicen.

  

### 2. Instalar Wapiti3

Usa `pip` para instalar la herramienta directamente desde el repositorio de Python.

  

```powershell

py -3.11 -m pip install wapiti3

```

  

### 3. Configuración del PATH (Opcional pero recomendado)

Para ejecutar `wapiti` desde cualquier lugar sin escribir la ruta completa:

  

1. Ejecuta: `py -3.11 -m pip show wapiti3`

2. Copia la ruta que aparece en `Location`.

3. El ejecutable está dentro de la carpeta `Scripts` en esa ruta.

*  *Ejemplo:*  `C:\Users\<Usuario>\AppData\Local\Programs\Python\Python311\Scripts`

4. Agrega esa ruta a tus Variables de Entorno del sistema o usa la ruta completa.

  

### 4. Verificación

```powershell

# Si está en el PATH:

wapiti --version

  

# Si NO está en el PATH (ejemplo):

C:\Ruta\A\Scripts\wapiti.exe --version

```

  

---

  

## Parte 2: Configuración del Objetivo (Shopify Hydrogen)

  

Desplegaremos una tienda de demostración basada en Hydrogen (React) para usarla como blanco de nuestras pruebas.

  

###  Repositorio Modificado (Recomendado)

El proyecto base de Shopify Hydrogen **no incluye vulnerabilidades intencionales** y puede no funcionar correctamente para las pruebas DAST de este laboratorio. Por ello, se preparó un repositorio con las **modificaciones necesarias** (middleware de vulnerabilidades, rutas expuestas, etc.) para que el escáner pueda detectar los fallos de seguridad.

> [!IMPORTANT]
> **Se recomienda clonar este repositorio en lugar de crear el proyecto desde cero.** Contiene todas las configuraciones y vulnerabilidades intencionales listas para ser escaneadas.

```powershell
git clone https://github.com/irvingaldahirangelesromero/hydrogen-storefront.git
cd hydrogen-storefront
npm install
npm run dev
```

**Éxito:** Tu tienda debería estar corriendo en `http://localhost:3000`.

  

---

### Opción Alternativa: Crear el Proyecto desde Cero

Si prefieres configurar el entorno manualmente, sigue los pasos a continuación. Ten en cuenta que necesitarás agregar las vulnerabilidades y el middleware por tu cuenta.

  

#### 1. Preparación del Directorio

```powershell

mkdir shopify-hydrogen-DAST-lab

cd shopify-hydrogen-DAST-lab

```

  

#### 2. Crear Proyecto Hydrogen

```powershell

npm create @shopify/hydrogen@latest

```

>  **Durante la instalación:**

>  * Nombre del proyecto: `hydrogen-storefront` (o el que gustes)

>  * Install dependencies? **Yes**

>  * Language: **TypeScript** (recomendado) o JavaScript

  

#### 3. Ajuste de Dependencias (Solución de errores comunes)

Algunas versiones recientes pueden tener conflictos. Aseguramos estabilidad instalando versiones específicas de `react-router`.

  

```powershell

cd hydrogen-storefront

npm install react-router@7.9.2 react-router-dom@7.9.2  @react-router/dev@7.9.2  @react-router/fs-routes@7.9.2 --force

```

  

#### 4. Lanzar el Servidor

```powershell

npm run dev

```

**Éxito:** Tu tienda debería estar corriendo en `http://localhost:3000`.

  

---

  

## Parte 3: Configuración de Burp Suite (Proxy Manual)

  

Burp Suite Community Edition nos permitirá interceptar las peticiones entre Wapiti (o el navegador) y la tienda Hydrogen para entender qué está pasando "bajo el capó".

  

1.  **Descargar e Instalar**: [PortSwigger Community Downloads](https://portswigger.net/burp/communitydownload)

2.  **Iniciar Burp**: Abre la aplicación y selecciona "Temporary Project" -> "Next" -> "Start Burp".

3.  **Verificar Proxy**: Ve a la pestaña `Proxy` > `Proxy Settings`. Confirma que hay un listener activo en `127.0.0.1:8080`.

  

---

  

## Parte 4: Ejecución del Laboratorio (Pentesting)

  

Ahora conectaremos todo. Usaremos Wapiti para atacar la tienda Hydrogen, pasando el tráfico a través de Burp Suite para inspección.

  

### Comando de Ataque Básico

Este comando escanea la tienda local y envía el tráfico a Burp.

  

```powershell

wapiti -u http://localhost:3000 -p http://127.0.0.1:8080 --flush-session -v 2

```

  

### Desglose de Argumentos Útiles

  

| Argumento | Descripción corta | Ejemplo |
| :--- | :--- | :--- |
| `-u, --url` | URL base del objetivo (obligatorio). Define el punto de partida del crawler. | `wapiti -u http://localhost:3000` |
| `--scope` | Define el alcance del escaneo. | `--scope folder` |
| `-m, --module` | Lista de módulos de ataque (ej. sql,xss,xxe). | `-m xss,sql,blindsql` |
| `-f, --format` | Formato de reporte (html, json, xml, txt, csv). | `-f html` |
| `-o, --output` | Ruta/nombre de archivo para escribir el reporte. | `-o reporte_final.html` |
| `-v, --verbose` | Nivel de detalle en la salida por consola. | `-v 2` |
| `--proxy, -p` | URL de proxy para enrutar tráfico (Burp Suite). | `-p http://127.0.0.1:8080` |
| `--form-url` | URL de login si se requiere autenticación. | `--form-url http://sitio.com/login` |
| `--form-data` | Datos para formulario de autenticación (POST). | `--form-data "user=admin&pass=123"` |
| `--cookie-from-browser` | Importar cookies desde navegador. | `--cookie-from-browser firefox` |
| `--depth` | Profundidad máxima del crawler. | `--depth 5` |
| `--max-links-per-page` | Máximo número de links por página a seguir. | `--max-links-per-page 20` |
| `--flush-session` | Fuerza nuevo escaneo ignorando previos. | `--flush-session` |
| `--update` | Actualiza módulos/firmas y sale. | `--update` |
| `--no-bugreport` | Evita envío automático de informes de fallo. | `--no-bugreport` |
| `-h, --help` | Muestra ayuda y lista de opciones. | `-h` |

  

### Análisis de Resultados

1.  **En la Terminal:** Wapiti mostrará en tiempo real las vulnerabilidades encontradas (marcadas usualmente en rojo).

2.  **En Burp Suite:** Ve a la pestaña `HTTP history`. Verás miles de peticiones generadas por Wapiti. Puedes inspeccionar los *payloads* (cargas útiles) que Wapiti inyectó para intentar romper la aplicación.

3.  **Reporte HTML:** Si usaste `-o carpeta`, abre el archivo `index.html` generado para ver un reporte gráfico profesional.
---

*Laboratorio creado para fines educativos. No ejecutes escaneos en sitios sin autorización explícita.*