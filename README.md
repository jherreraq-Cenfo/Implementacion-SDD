# Implementación SDD

## 1) Objetivo Principal  
**Automatización de procesos**  
El objetivo principal de esta implementación es automatizar procesos utilizando Zia y Zoho Analytics, logrando un flujo de trabajo eficiente que minimice la intervención manual y optimice los tiempos de respuesta.

## 2) Implementación de Zia  
**Análisis de Sentimiento**  
Zia permite realizar análisis de sentimiento sobre los datos, brindando información útil para la toma de decisiones.  
**Predicción de Campos**  
A través de algoritmos de machine learning, Zia puede predecir campos relevantes en la base de datos que influyen en el análisis de los resultados.  
**Zia Insights**  
Ofrece percepciones profundas sobre los datos mediante el uso de tendencias y patrones detectados.

## 3) Refinamiento con Zoho Analytics  
Utilizando Zoho Analytics se puede refinar la información obtenida, creando reportes personalizados que proporcionen una visión clara y detallada de los KPIs.

**- Volumen de Tickets:** Predecir cuántos casos recibirán el próximo mes para ajustar turnos del personal.
**- Detección de Anomalías:** La IA les avisará si hay un pico inusual de tickets en una categoría específica antes de que se convierta en una crisis.

## 4) Hoja de Ruta  

| Fase | Accion |  Resultado Esperado
|------|-------------|----------------|  
| Limpieza| Usar IA para reclasificar tickets mal categorizados |  Datos maestros precisos.
| Enriquecimiento | Integrar sentimientos y tiempos de respuesta predictivos.|  Reportes con contexto emocional y preventivo.
| CVisualización| Crear Dashboards de "Causa Raíz" asistidos por IA.|  Identificar el origen real de los problemas recurrentes.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Spec-Driven Development  
Esta metodología permite que el desarrollo se centre en las especificaciones del sistema, garantizando que todas las funcionalidades requeridas sean implementadas y probadas adecuadamente.

El primer paso en SDD es crear un documento (usualmente en OpenAPI/Swagger o AsyncAPI) que describa cómo la IA interactuará con los datos de ServiceDesk.
¿Qué debe incluir tu Spec?
**- Data Models:** Define exactamente qué campos de un ticket (ID, descripción, tiempo de cierre, categoría) necesita la IA.
**- AI Enrichment Schema:** Define cómo se ve el dato "refinado". Por ejemplo, un objeto JSON que incluya sentiment_score, predicted_root_cause y urgency_adjustment.
**- Endpoints de Integración:** Si usas un servicio externo (como Azure OpenAI o una instancia propia), define los contratos de entrada y salida.


## 1) Flujo de Trabajo SDD  
**Fase A: El Contrato del Reporte**
Antes de generar el reporte en Zoho Analytics, diseñas el esquema del reporte final.


Ejemplo de Spec (JSON Schema):
JSON
{
  "report_name": "Analisis_IA_Semanal",
  "fields": {
    "ticket_id": "integer",
    "ai_summary": "string",
    "anomaly_detected": "boolean"
  }
}


**Fase B: Mocking y Desarrollo en Paralelo**
Con la especificación lista:
  **- Equipo de Datos:** Prepara los conectores de Zoho SDP para extraer los datos según el esquema.
  **- Equipo de IA:** Desarrolla los prompts o modelos basándose únicamente en la Spec. Pueden usar "datos ficticios" (mocks) que cumplan con el contrato mientras los datos reales se preparan.

**Fase C: Validación Automatizada**
Cualquier reporte generado por la IA debe validarse contra la Spec. Si la IA devuelve un formato de análisis que no coincide con lo definido (por ejemplo, un texto en lugar de un booleano en anomaly_detected), el proceso falla automáticamente antes de llegar al dashboard gerencial.



## 2) Implementación Técnica  
**Tabla de 4 capas**  
| Capa | Herramiemta |  Rol en SDD
|------|-------------|------------|  
| Data Source | Zoho SDP API | Proveedor de datos crudos (Inputs). |  
| Spec Layer | Stoplight / Swagger | Donde reside la definición del reporte refinado. |
| Logic (The Refiner) | Zoho Deluge / Python | El código que consume la Spec y llama a la IA para transformar el dato. |
| Consumer | Zoho Analytics | Renderiza el resultado final validado contra la Spec.|


## 3) Ventajas de SDD  
**- Consistencia:** No importa si cambian la versión de ServiceDesk; mientras la API cumpla con la Spec, tus reportes de IA no se romperán.
**- Desacoplamiento:** Puedes cambiar de motor de IA (pasar de Zia a GPT-4, por ejemplo) sin rediseñar el reporte, porque el "contrato" de salida sigue siendo el mismo.
**- Calidad:** Obligas a que los reportes tengan una estructura lógica, evitando que la IA alucine o entregue datos irrelevantes.

## Posible Ejemplo 

## 1) La Especificación Actualizada (Contrato JSON)  
Este esquema es el que "obliga" a la IA y a ServiceDesk Plus a hablar el mismo idioma. Si la IA detecta que un ticket "Normal" tiene palabras clave de "Urgente", el reporte lo marcará como una anomalía de cumplimiento.

- **Urgente**: 120min  
- **Alto**: 480min  
- **Medio**: 1440min  
- **Normal**: 2880min  
Los niveles de prioridad permiten clasificar las tareas de acuerdo a su urgencia y relevancia.

```json
{
  "sla_definitions": {
    "Urgente": { "resolution_time_min": 120, "impact": "Crítico" },
    "Alto": { "resolution_time_min": 480, "impact": "Alto" },
    "Medio": { "resolution_time_min": 1440, "impact": "Moderado" },
    "Normal": { "resolution_time_min": 2880, "impact": "Bajo" }
  },
  "ai_refinement_rules": [
    "sentiment_analysis",
    "bottleneck_prediction",
    "category_misalignment_check"
  ]
}
```
## 2) Ejemplo de Reporte Refinado  
**Tickets #501-#504**  
El reporte se genera a partir de los tickets especificados, proporcionando una visualización clara del estado de cada elemento.

## 3) Implementación en Zoho  

**1. Definir la Spec:** Crear el archivo YAML/JSON con las reglas de los 4 niveles (2h, 8h, 24h, 48h).

**2. Mocking:** Antes de conectar la IA real, genera reportes con datos falsos para ver si los widgets de Zoho Analytics muestran lo que necesitas.

**3. Integración de IA (The Refiner):** Configurar un script (en Deluge o Python) que tome el ticket, lo compare con la Spec y le pregunte a la IA: "¿Este ticket de 24h va a cumplir el contrato según este historial?".

**4. Validación:** El reporte solo se actualiza si la respuesta de la IA cumple con el formato del paso 1.

## 4) Beneficio Inmediato  
El beneficio inmediato de este proceso es una mejora en la capacidad de respuesta y en la calidad de información disponible para la toma de decisiones.

```json
{
  "id": "1028",
  "priority": "Alto",
  "subject": "El sistema de ventas no carga",
  "created_time": "2023-10-27 10:00:00",
  "current_time": "2023-10-27 13:30:00",
  "status": "Abierto"
}


{
  "ticket_id": "1028",
  "sla_status": "ADVERTENCIA",
  "risk_score": "75",
  "prediction": "El ticket lleva 3.5 horas de un SLA de 8h. Al ritmo actual y por historial de este tipo de fallas, se incumplirá en 5 horas.",
  "anomaly_detected": true,
  "suggested_priority": "Urgente",
  "recommended_action": "Escalar a Nivel 2 inmediatamente; el sistema de ventas afecta ingresos y debería ser Urgente (2h) no Alto (8h)."
}
```
