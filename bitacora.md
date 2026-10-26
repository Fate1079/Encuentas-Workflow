# Bitácora Unificada del Proyecto: Automatización Inteligente de Encuestas y Retroalimentación con n8n y Supabase

## 1. Identificación y Alcance
* **Nombre del Proyecto**: Sistema Automatizado de Gestión y Calificación de Encuestas Académicas (`CaveCoders`).
* **Institución**: Institución Universitaria EAM.
* **Curso**: Automatización inteligente de procesos con n8n.
* **Periodo**: 2026-II.
* **Equipo de Trabajo**: 
  * Erick Sebastián Bolívar Arboleda (Owner / Desarrollador Principal)
  * Andrés Felipe García González (Code Analyst)
  * Diego Alejandro Muñoz Buitrago (Tester)
  * Sebastian Muñoz (Designer)
  * Johan Sebastián Peña (Designer)
* **Problema que resuelve**: La recolección manual de encuestas sobre preferencias y calificaciones de lenguajes de programación genera cuellos de botella operativos, retrasos en la clasificación de casos críticos (NPS bajo o respuestas incorrectas) y falta de retroalimentación inmediata estructurada para los estudiantes o participantes.
* **Alcance de la versión entregada**: 
  1. Captura dinámica de datos mediante formularios estructurados (`Form Trigger`).
  2. Procesamiento de variables, limpieza de datos (`Edit Fields`) y enrutamiento lógico mediante condicionales (`If`) y clasificadores (`Switch`).
  3. Calificación automatizada de respuestas basadas en palabras clave (*keywords*).
  4. Integración modular mediante un **Subworkflow de Retroalimentación Pedagógica** encargado de calcular mensajes personalizados.
  5. Almacenamiento persistente y actualización de registros en base de datos relacional (**Supabase**).
* **Fuera de alcance**: Pasarelas de pago, despliegue en clústeres de servidores cloud comerciales 24/7 (se ejecuta localmente con túneles de desarrollo) y envío masivo de correos electrónicos transaccionales.

---

## 2. Proceso Manual vs. Proceso Automatizado
* **Proceso Manual (Antes)**: 
  * Los participantes respondían cuestionarios en formatos físicos o digitales aislados.
  * Un operador debía revisar manualmente cada respuesta, contrastarla con las claves de calificación y calcular el puntaje de aciertos uno a uno.
  * La captura de los datos finales hacia las tablas de control se hacía mediante transcripción manual, propensa a errores humanos y demoras significativas.
* **Proceso Automatizado (Con n8n y Supabase)**:
  * El usuario ingresa sus datos y respuestas a través de la interfaz web nativa.
  * El flujo en n8n recibe el payload de manera instantánea mediante el disparador (`Form Trigger`).
  * El sistema limpia los datos, ejecuta el enrutamiento y la calificación por *keywords* de forma totalmente desatendida.
  * Se invoca un subworkflow modular que inyecta un mensaje de retroalimentación pedagógica y actualiza de inmediato los registros en la base de datos de Supabase.

---

## 3. Funcionamiento de la Solución
* **Activación y Entrada**: El proceso inicia mediante un disparador de formulario (`Form Trigger`) que captura campos obligatorios como el nombre del usuario, el lenguaje de programación seleccionado, el puntaje NPS y las respuestas técnicas.
* **Transformación y Reglas de Negocio**:
  * El nodo *Edit Fields* estandariza las cadenas de texto (convirtiendo lenguajes a minúsculas, por ejemplo).
  * El nodo *Switch* clasifica las solicitudes según la categoría del problema o lenguaje.
  * El nodo *If* separa las valoraciones críticas (detractores con NPS ≤ 6) para priorizar su atención.
* **Arquitectura de Workflows (Principal y Subworkflow)**:
  * **Flujo Principal (`Principal_Encuesta.json`)**: Recibe la data, procesa la calificación principal de los lenguajes y guarda el registro inicial en Supabase. Al finalizar su ejecución normal, invoca mediante un nodo *Execute Workflow* al subworkflow.
  * **Subworkflow (`Subworkflow_Feedback.json`)**: Recibe los datos procesados, evalúa el total de respuestas correctas (`total_correctas`), genera de forma dinámica un texto de recomendación pedagógica (`retroalimentacion_ia`) y realiza una operación de actualización (*Update*) en la tabla `respuestas_encuesta` de Supabase utilizando el ID correspondiente.

---

## 4. Integraciones y Configuración
* **Servicios Externos Utilizados**:
  * **n8n**: Orquestador principal de la lógica de negocio, nodos condicionales, bloques de código JavaScript y llamadas modulares.
  * **Supabase**: Base de datos como servicio (BaaS) que aloja las tablas de usuarios y el registro histórico de respuestas evaluadas.
* **Credenciales y Configuración Requerida**:
  * Credencial de conexión API de Supabase (`supabaseApi`) configurada con permisos de lectura y escritura sobre la tabla `respuestas_encuesta`.
  * URL pública activa para la recepción de los triggers de formulario.

---

## 5. Manejo de Errores y Recuperación
* **Errores del Negocio (Validaciones)**: Si el usuario omite campos obligatorios o introduce un formato no válido en las respuestas, el flujo de n8n frena la ejecución en las etapas condicionales iniciales, evitando registros corruptos en la base de datos.
* **Fallos Técnicos Controlados**: En caso de interrupciones temporales de conexión con el servicio externo de Supabase, las excepciones se registran en el historial de ejecuciones de n8n. Para evitar duplicidad de registros al reintentar una operación, la actualización en el subworkflow utiliza filtros estrictos basados en identificadores únicos de fila (`id`).

---

## 6. Pruebas y Evidencias
* **Caso 1: Envío Normal con Éxito (5/5 aciertos)**
  * *Datos de entrada*: Nombre: "Estudiante Prueba", Lenguaje: "Python", Puntaje NPS: 9, Respuestas correctas.
  * *Resultado esperado*: Autenticación correcta, calificación máxima, generación de mensaje de felicitación por el subworkflow y almacenamiento exitoso de la fila en Supabase.
  * *Resultado observado*: Ejecución en verde en todos los nodos y verificación visual directa de la columna `retroalimentacion_ia` poblada en la tabla de Supabase.
* **Caso 2: Entrada Crítica / Detractor (NPS ≤ 6)**
  * *Datos de entrada*: Calificación baja y comentarios con observaciones de mejora.
  * *Resultado esperado*: Enrutamiento a través de las rutas condicionales del nodo *If* y etiquetado prioritario.
  * *Resultado observado*: Clasificación correcta de la alerta en el flujo sin interrupciones técnicas.

---

## 7. Evolución y Estado Final
* **Evolución desde el Corte 1**: Se pasó de un flujo lineal básico de control local a una arquitectura modular profesional que incorpora separación de responsabilidades mediante subworkflows, manejo de variables en código JavaScript y persistencia robusta en base de datos relacional.
* **Aportes del Equipo (`CaveCoders`)**:
  * *Erick Sebastián Bolívar Arboleda*: Orquestación general de la arquitectura en n8n, diseño de lógica y estructuración de subworkflows.
  * *Andrés Felipe García González*: Análisis de código, validación de sintaxis y expresiones en nodos de transformación.
  * *Diego Alejandro Muñoz Buitrago*: Ejecución de pruebas unitarias de integración y validación de datos en Supabase.
  * *Sebastian Muñoz y Johan Sebastián Peña*: Diseño conceptual de interfaces de captura y optimización de flujos visuales.
* **Estado Final**: 
  * *Implementado*: Flujo principal, subworkflow de retroalimentación, conexión con Supabase y lógica de evaluación.
  * *Simulado / Propuesto*: Despliegue permanente en clúster cloud 24/7 y pasarela de notificaciones automáticas por mensajería instantánea en tiempo real.
