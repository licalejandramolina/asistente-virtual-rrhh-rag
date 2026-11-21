# Asistente Virtual de Recursos Humanos con IA y RAG

## 1. Descripción del proyecto

Este proyecto consiste en el desarrollo de un **Asistente Virtual de Recursos Humanos**, diseñado para automatizar la atención de consultas frecuentes de los colaboradores de una organización.

La solución utiliza Inteligencia Artificial Generativa y la arquitectura RAG (*Retrieval-Augmented Generation*) para recuperar información desde una base de conocimiento interna y generar respuestas contextualizadas, claras y consistentes.

El objetivo es reducir la carga operativa del área de Recursos Humanos, agilizar el acceso a la información y permitir que el equipo pueda concentrarse en tareas de mayor valor agregado.

## 2. Problema de negocio

En las áreas de Recursos Humanos se reciben diariamente consultas recurrentes relacionadas con:

- Documentación laboral.
- Licencias y ausencias.
- Obras sociales.
- Recibos de sueldo.
- Beneficios corporativos.

La atención manual de estas consultas requiere tiempo del equipo de RR. HH., genera interrupciones en las tareas habituales y puede ocasionar demoras en las respuestas.

Frente a esta problemática, se propone implementar un asistente virtual que permita resolver consultas frecuentes de manera automatizada, utilizando información previamente validada y disponible en una base de conocimiento.

## 3. Objetivos

- Automatizar la resolución de consultas frecuentes de Recursos Humanos.
- Centralizar la información en una base de conocimiento.
- Reducir los tiempos de respuesta.
- Disminuir las tareas repetitivas del equipo.
- Derivar a un responsable humano las consultas que requieran intervención.
- Mejorar la disponibilidad y accesibilidad de la información.

## 4. Arquitectura y funcionamiento

El proyecto fue desarrollado mediante n8n, utilizando un flujo de automatización que integra distintas fuentes de información, un sistema de recuperación documental y un agente de Inteligencia Artificial.

### Fuentes de información

La base de conocimiento se construye a partir de dos fuentes:

**1. Documento FAQ de Recursos Humanos**

Contiene preguntas frecuentes y sus respectivas respuestas, preparadas para la demostración del proyecto.

**2. Google Sheets**

Se utiliza como fuente externa de información, con una tabla estructurada que contiene las siguientes columnas:

- Categoría.
- Pregunta.
- Respuesta.
- Fuente.
- Vigencia.

Los registros se filtran para incorporar únicamente aquellos marcados como vigentes.

### Proceso de carga de documentos

1. Se inicia el proceso de carga mediante un disparador manual.
2. Se obtiene la información de Google Sheets.
3. Se filtran los registros vigentes.
4. Se transforman los registros al formato requerido.
5. Se unifican las dos fuentes de información mediante un nodo Merge.
6. Se normalizan los documentos.
7. Se generan embeddings mediante el modelo Nomic Embed Text.
8. Se incorporan los documentos a una base vectorial en memoria.

Para la demostración se utilizan seis documentos: un documento FAQ y cinco registros provenientes de Google Sheets.

### Proceso de atención de consultas

1. El colaborador envía una consulta mediante un Webhook.
2. El sistema valida que estén presentes el mensaje y el identificador del remitente.
3. Se recupera información relevante desde la base de conocimiento.
4. El AI Agent analiza la consulta utilizando el contexto recuperado.
5. Se genera una respuesta estructurada.
6. El sistema devuelve la respuesta mediante un nodo Respond to Webhook.

Cuando no existe información documental suficiente, el asistente reconoce la limitación y deriva la consulta al área de Recursos Humanos.

## 5. Tecnologías utilizadas

| Tecnología | Función |
|---|---|
| n8n | Orquestación y automatización del flujo |
| Ollama | Ejecución local de modelos de IA |
| Ministral 3B | Modelo de lenguaje utilizado por el AI Agent |
| Nomic Embed Text | Generación de embeddings |
| Simple Vector Store | Almacenamiento y recuperación vectorial en memoria |
| Google Sheets | Fuente externa de información |
| Webhook | Recepción de consultas |
| PowerShell | Pruebas de funcionamiento del asistente |

### Entorno de ejecución: n8n local

Para el desarrollo y ejecución del proyecto se utilizó **n8n instalado y ejecutado localmente**, en lugar de utilizar n8n Cloud.

Esta decisión se tomó principalmente por los siguientes motivos:

- **Aprendizaje y experimentación:** permite construir, configurar y probar los flujos de automatización en un entorno de desarrollo propio.
- **Control del entorno:** facilita administrar las configuraciones, credenciales, integraciones y ejecuciones del workflow.
- **Reducción de costos:** permite desarrollar el prototipo sin contratar una suscripción a n8n Cloud.
- **Integración con modelos locales:** facilita la conexión con Ollama para ejecutar el modelo de lenguaje Ministral 3B y el modelo de embeddings Nomic Embed Text sin depender de servicios externos de inferencia.

La instancia de n8n se ejecuta en el entorno local mediante `localhost:5678`, mientras que Ollama proporciona los modelos de Inteligencia Artificial desde el equipo.

Esta arquitectura resulta adecuada para el desarrollo, experimentación y demostración del prototipo. Para una implementación productiva con acceso de múltiples colaboradores sería necesario evaluar un despliegue en un servidor o infraestructura cloud, junto con mecanismos de autenticación, disponibilidad, seguridad y persistencia.

## 6. Justificación de las decisiones tecnológicas

### Modelo local vs. modelo en la nube

Se eligió Ollama para ejecutar los modelos de manera local.

Esta decisión permite desarrollar y probar la solución sin depender de servicios externos de inferencia ni de una API paga.

Además, facilita el control sobre los datos utilizados durante las pruebas y reduce los costos asociados al consumo de modelos en la nube.

Como contrapartida, la ejecución local requiere disponer de recursos de hardware suficientes y realizar tareas de mantenimiento.

### Elección de embeddings

Se utilizó Nomic Embed Text para transformar los documentos en representaciones vectoriales.

Esto permite recuperar información según la similitud semántica entre la consulta del usuario y los documentos disponibles, en lugar de depender exclusivamente de coincidencias exactas de palabras.

### Vector Store

Se utilizó Simple Vector Store de n8n en modo de almacenamiento en memoria para simplificar la implementación y las pruebas del prototipo.

Es una alternativa adecuada para una demostración funcional. Para un entorno productivo sería conveniente evaluar una base vectorial persistente y escalable.

### AI Agent + RAG vs. un nodo de IA simple

Se eligió una arquitectura basada en AI Agent y RAG porque permite que el modelo consulte una fuente de conocimiento antes de generar una respuesta.

Un nodo de IA simple podría responder utilizando su conocimiento general, con el riesgo de proporcionar información incorrecta o no alineada con las políticas de la organización.

La arquitectura RAG permite:

- Fundamentar las respuestas en documentación disponible.
- Reducir las respuestas inventadas.
- Actualizar la información sin tener que modificar el prompt del modelo.
- Reconocer cuándo no existe información suficiente.
- Derivar consultas que requieren intervención humana.

## 7. Prompt y criterios de comportamiento del agente

El AI Agent utiliza un prompt de sistema diseñado para actuar como asistente de Recursos Humanos.

Sus principales reglas son:

- Comunicarse en español rioplatense, con tono profesional y cordial.
- Responder exclusivamente con información respaldada por los documentos recuperados.
- No inventar políticas, procedimientos, plazos ni condiciones.
- Reconocer cuando no dispone de información suficiente.
- Derivar las consultas que requieren análisis individual al área de Recursos Humanos.
- No brindar asesoramiento legal.
- No validar liquidaciones individuales de recibos de sueldo.
- No solicitar ni exponer información personal sensible.
- Rechazar consultas que no estén relacionadas con el ámbito de Recursos Humanos.

El agente devuelve una respuesta estructurada con los siguientes campos:

- `success`
- `categoria`
- `respuesta`
- `requiere_intervencion_humana`

## 8. Validaciones y manejo de errores

Se incorporaron mecanismos de validación para evitar que el sistema procese solicitudes incompletas.

El nodo IF verifica la presencia de dos datos obligatorios:

- `message`: consulta del usuario.
- `sender`: identificador del remitente.

Si alguno de estos datos falta, el flujo deriva a una ruta alternativa que devuelve una respuesta de error estructurada con código HTTP 400.

También se contempla la derivación al equipo de Recursos Humanos cuando el asistente no encuentra información suficiente en la base de conocimiento.

### Casos de prueba realizados

| Escenario | Resultado esperado | Estado |
|---|---|---|
| Consulta sobre certificado laboral | Res
