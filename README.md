Smart Request IA

Automatización inteligente para la gestión de solicitudes de clientes

Trabajo final — Automatización con IA
Desarrollado por Evelin Bordón · BC_Proyectos · Julio 2026


Resumen

Smart Request IA automatiza la recepción, clasificación y gestión inicial de los requerimientos que los clientes envían por correo electrónico. Combina automatización de procesos, inteligencia artificial generativa y validación humana para reducir tiempos operativos, mejorar la trazabilidad y estandarizar la gestión de solicitudes.

Objetivo: automatizar la recepción y clasificación de solicitudes comerciales, permitiendo que cada requerimiento sea registrado, analizado y priorizado automáticamente antes de su revisión por el equipo responsable.

Problemática


Recepción manual de correos
Clasificación subjetiva y no estandarizada
Riesgo de pérdida de solicitudes
Falta de trazabilidad del estado de cada pedido
Demoras en la asignación al equipo correcto


Solución propuesta


Detecta nuevos correos automáticamente
Registra la solicitud en la base de datos
Analiza el contenido mediante IA (Gemini)
Clasifica prioridad, complejidad y equipo
Notifica al responsable para su aprobación
Ante un error, alerta al equipo sin perder visibilidad


Arquitectura

Dos flujos independientes en n8n, sincronizados a través de Airtable como memoria central.

Flujo 1 · Disparado por Gmail


Llega el mail (Gmail Trigger)
Se marca como leído — evita reprocesos
Se crea el registro en Airtable
Gemini clasifica la solicitud
Si hay éxito → espera aprobación humana en Airtable
Si hay error → alerta al equipo por mail


Flujo 2 · Disparado por Schedule (cada 1 min)


Schedule Trigger
Busca en Airtable solicitudes Aprobadas / Rechazadas
Responde al cliente en el mismo hilo de correo
Actualiza el estado a "Enviado"


Arquitectura de datos en Airtable

La base no es una tabla plana: está compuesta por dos tablas relacionadas, para evitar datos aislados y permitir escalar la gestión de equipos sin tocar el flujo.


Solicitudes — registro principal de cada requerimiento (datos del cliente, clasificación de la IA, estado, aprobación humana). Incluye un campo Link to another record que vincula cada solicitud con su equipo responsable.
Equipos_Responsables — catálogo de equipos disponibles (nombre, capacidad, canal de contacto), referenciado desde la tabla Solicitudes.


🔗 Base de datos (solo lectura): https://airtable.com/appCl3l8wMg04OFVh/shrIjfyurzqTK5RJI

Herramientas utilizadas

HerramientaFunciónn8nOrquestación del flujo end-to-endGmailEntrada de solicitudes y salida de respuestasAirtableBase de datos central (memoria del sistema)Google GeminiMotor de IA generativa para clasificaciónGitHubVersionado y documentación del proyecto

Inteligencia artificial

Google Gemini analiza el asunto del correo, la descripción de la solicitud, la urgencia declarada y la complejidad estimada, y devuelve un JSON estructurado con la clasificación completa del requerimiento, sin ambigüedad ni texto libre:

json{
  "tipo_solicitud": "Desarrollo",
  "prioridad": "Media",
  "complejidad": "Media",
  "equipo_responsable": "Datos",
  "comentarios_ia": "..."
}

Para optimizar costos, se limita el presupuesto de razonamiento interno (Thinking Budget = 0) y el máximo de tokens de salida, evitando respuestas truncadas o innecesariamente largas.

Human in the Loop

Aunque la IA clasifica automáticamente, la decisión final permanece en manos del equipo mediante el campo Aprobación Humana, garantizando control y supervisión antes de contactar al cliente.


Aprobada → se responde al cliente dentro del mismo hilo de correo confirmando el inicio del desarrollo, y el registro pasa a estado Enviado.
Rechazada → se responde al cliente en el mismo hilo informando que la solicitud no fue aprobada en esta oportunidad, y el registro también pasa a estado Enviado.


Resultados y resiliencia

Validado durante las pruebas:


Registro automático de cada solicitud
Clasificación automática consistente
Actualización correcta de Airtable
Envío de notificaciones internas y al cliente
Integración completa entre Gmail, IA y Airtable
Anti-bucle: sin reprocesos de correos ya leídos


Camino infeliz (test de estrés): el flujo se ejecutó repetidamente con datos incompletos y fallas simuladas de la API de IA. Cada error quedó registrado en Airtable con su detalle técnico (Logs_Errores) y generó una alerta automática al equipo, sin interrumpir el resto del proceso ni perder visibilidad sobre la solicitud.

Escalabilidad

La solución puede ampliarse incorporando nuevos canales y herramientas de gestión, sin modificar el núcleo del flujo: Jira, ClickUp, Slack, Microsoft Teams, WhatsApp, Power BI.

Contenido de este repositorio

Smart_Request_IA/
├── README.md
├── Smart_Request_IA_Final.pdf     → presentación completa del proyecto
├── workflow.json                   → export del flujo n8n
├── /evidencia                      → capturas del flujo en funcionamiento
│                                      (incluye la relación entre tablas Solicitudes ↔ Equipos_Responsables)
└── /video                          → demo del sistema en ejecución

Conclusión

Smart Request IA demuestra cómo la combinación de automatización, inteligencia artificial y supervisión humana permite optimizar procesos administrativos, reducir tiempos de respuesta y mejorar la calidad de la gestión de requerimientos. El proyecto constituye una base escalable para futuras soluciones desarrolladas por Evelin Bordón, en el marco de BC_Proyectos.
