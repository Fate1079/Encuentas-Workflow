# Proyecto: Automatización Inteligente de Encuestas (`CaveCoders`)

Sistema integrado para la recolección, calificación automatizada y gestión de retroalimentación en encuestas académicas utilizando **n8n**, **Google Forms**, **Google Apps Script** y **Supabase**.

---

## 📂 Estructura del Repositorio
- `workflows/`
  - `Principal_Encuesta.json` — Flujo principal de recepción, validación y control.
  - `Subworkflow_Feedback.json` — Subworkflow modular encargado de procesar la retroalimentación pedagógica y actualizar la base de datos.
- `Bitacora_Unificada_Proyecto.md` — Documentación técnica detallada del estado actual del proyecto.

---

## 🌐 Enlace de Prueba (Google Forms)
Puedes probar el funcionamiento en vivo de la interfaz de usuario accediendo al siguiente formulario público:
👉 **[Acceder al Google Form de la Encuesta](https://forms.gle/ye4EJb5R69Q3b6TY7)**

---

## ⚙️ Notas de Infraestructura y Configuración Local
Debido a que este desarrollo se encuentra en un entorno académico local, la arquitectura depende de los siguientes componentes configurados en la máquina del autor:

1. **Google Forms y Apps Script**: 
   - El formulario está enlazado a un script de Google Apps Script vinculado a la cuenta institucional del autor.
   - El script envía automáticamente el payload en formato JSON mediante un Webhook cada vez que un usuario responde el formulario.

2. **Túnel de Cloudflare (Cloudflare Tunnels)**:
   - Para recibir peticiones en el entorno local (`localhost:5678`), se utiliza un túnel temporal de Cloudflare ejecutado desde la máquina local (`cloudflared tunnel --url http://localhost:5678`).
   - *Nota importante*: Como la dirección del túnel es dinámica y depende de la ejecución en la máquina del autor, debe iniciarse el servicio localmente y actualizarse la URL del Webhook en Google Apps Script si se requiere realizar pruebas de extremo a extremo en vivo.

3. **Base de Datos (Supabase)**:
   - El almacenamiento relacional se gestiona mediante tablas configuradas en una instancia de Supabase vinculada al entorno de desarrollo del equipo (`CaveCoders`). 
   - Para ejecutar los flujos de manera local, es necesario configurar las credenciales de acceso (`supabaseApi`) en los nodos correspondientes dentro de n8n.

---

## 🚀 Guía rápida de puesta en marcha
1. **Clonar el repositorio** en tu entorno local.
2. **Importar los flujos** ubicados en la carpeta `workflows/` dentro de tu instancia de **n8n**.
3. **Configurar las credenciales** de Supabase en los nodos correspondientes de ambos workflows.
4. **Iniciar n8n localmente** y levantar el túnel de Cloudflare para conectar el Webhook con el Google Form.
5. **Activar los workflows** en tu panel de n8n.

---
📄 **Documentación Completa**: Consulta todos los detalles, diagramas y pruebas en la [Bitácora Unificada del Proyecto](./Bitacora_Unificada_Proyecto.md).
"# Encuentas-Workflow" 
"# Encuentas-Workflow" 
