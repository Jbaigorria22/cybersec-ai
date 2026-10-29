# CyberSec AI — Project Brief

> **Versión:** 1.1 · **Fecha:** 2026-10-29 · **Estado del proyecto:** Fase 1 — Diseño (Etapa 1 de 7)

## 1. Visión del proyecto

**CyberSec AI** es un proyecto de ciberseguridad que busca combinar detección determinística, inteligencia artificial y herramientas de investigación para ayudar a los analistas de seguridad a identificar, comprender e investigar actividades sospechosas en entornos cloud.

El objetivo es evolucionar desde un motor de detección basado en evidencia hasta una plataforma de investigación asistida por IA, con una interfaz visual profesional, explicaciones fundamentadas y flujos de investigación controlados.

No se trata simplemente de conectar un LLM a registros de seguridad. El valor del proyecto estará en combinar detecciones verificables, evidencia trazable, evaluación cuantitativa y asistencia inteligente.

### Objetivos principales

- Detectar comportamientos sospechosos a partir de eventos de seguridad.
- Generar alertas reproducibles y respaldadas por evidencia.
- Utilizar IA para explicar hallazgos, distinguir hechos de hipótesis y proponer pasos de investigación.
- Incorporar progresivamente RAG, agentes y herramientas de seguridad con permisos limitados.
- Construir una aplicación cloud-native, segura, escalable y desplegable mediante infraestructura como código.
- Desarrollar un portfolio técnico que demuestre competencias en Cloud Engineering, AI Engineering y Cybersecurity.

### Posicionamiento frente a proyectos previos

| Proyecto | Enfoque | Relación con CyberSec AI |
|---|---|---|
| **Guard-RAG** | *Security of AI*: proteger una aplicación de IA | Se reutilizan sus prácticas de defensa contra prompt injection y de evaluación |
| **ThreatScribe** | Fundamentos de detección sobre logs | Suspendido; sus aprendizajes se integran en la Fase 1 |
| **CyberSec AI** | *AI for Security*: usar IA para investigar incidentes en AWS | Proyecto actual |

## 2. Usuario objetivo

La primera versión estará orientada a **analistas de seguridad y profesionales de ciberseguridad que necesitan investigar eventos y comprender alertas con mayor rapidez**.

La plataforma debería permitirles responder preguntas como:

- ¿Qué ocurrió?
- ¿Qué identidad realizó la acción?
- ¿Cuándo y desde dónde ocurrió?
- ¿Por qué se generó esta alerta?
- ¿Qué evidencia respalda la detección?
- ¿Qué debería investigar a continuación?
- ¿Qué tan confiable es la conclusión y qué información falta?

A futuro, el proyecto podría incorporar una vista ejecutiva para responsables de pequeñas y medianas empresas, con resúmenes de riesgos y recomendaciones comprensibles sin conocimientos técnicos avanzados.

## 3. Referencia visual y experiencia de usuario

La imagen de referencia proporcionada para el proyecto muestra un dashboard de ciberseguridad (diseño de terceros). Se utilizará como **inspiración visual y funcional**, no como una interfaz que deba copiarse literalmente. La imagen se conserva solo como referencia privada y **no se incluye en el repositorio público**.

La dirección visual deseada es una aplicación moderna de seguridad, con estética oscura, acentos violetas, tarjetas bien organizadas, gráficos legibles y una jerarquía visual clara.

### Adaptación de la referencia a CloudTrail

La referencia está pensada para aplicaciones SaaS (Slack, Jira, GitHub). Cada componente se adapta a lo que CloudTrail realmente puede informar:

| Componente de la referencia | Equivalente en CyberSec AI | Fuente de datos | Disponible desde |
|---|---|---|---|
| Violations Summary | Hallazgos por severidad | Motor de detección | Fase 1 (UI inicial) |
| Event Severity (barras por hora) | Hallazgos por severidad en el tiempo | Hallazgos + `event_time` | Fase 1 (UI inicial) |
| Incident Timeline | Línea de tiempo de eventos de un hallazgo | Evidencia del hallazgo | Fase 1 (UI inicial) |
| Risk by Application | Hallazgos por servicio AWS (`event_source`) o por identidad | Hallazgos | Fase 1 (UI inicial) |
| Event Types | Hallazgos por regla o categoría (acceso, evasión, privilegios) | Catálogo de reglas | Fase 1 (UI inicial) |
| Live Threat Map | Actividad por región AWS / IP de origen | `aws_region`, `source_ip` | Posterior (requiere ingesta continua) |
| Detection Confidence | Precisión y recall **por regla**, medidos sobre datos etiquetados | Módulo de evaluación | Fase 1 (como reporte de evaluación, no como métrica en vivo) |
| Security Query | Consulta conversacional sobre hallazgos | Capa de IA | Fase de asistencia con IA |
| Users / Endpoints / Apps monitored | Identidades y cuentas observadas | Eventos normalizados | Posterior |

> **Regla de honestidad visual:** las métricas de calidad (falsos positivos, verdaderos positivos) solo existen cuando hay datos etiquetados. En producción no se conoce la verdad de referencia, por lo que el dashboard nunca mostrará esas métricas como si fueran mediciones en vivo.

### Elementos visuales deseados

- **Overview dashboard:** resumen del estado de seguridad y las alertas.
- **Violations Summary:** cantidad de hallazgos por severidad: crítica, alta, media y baja.
- **Threat Monitoring:** visualización de actividad sospechosa y evolución temporal de los eventos.
- **Detection Metrics:** métricas de calidad de las detecciones cuando existan datos etiquetados suficientes.
- **Incident Timeline:** secuencia cronológica de los eventos relacionados con una investigación.
- **Risk by Resource:** distribución de hallazgos por servicio, identidad o recurso cloud.
- **Event Types:** clasificación de hallazgos por categoría.
- **Security Query:** una interfaz conversacional para consultar hallazgos y solicitar asistencia durante una investigación.
- **Incident Investigation:** una vista detallada con evidencia, explicación, hipótesis, nivel de confianza y acciones recomendadas.

### Principios de diseño

1. Priorizar la legibilidad y la utilidad analítica sobre los efectos visuales.
2. Utilizar colores para comunicar severidad y estado, no solamente como decoración.
3. Mantener una interfaz consistente, con navegación clara y componentes reutilizables.
4. Evitar gráficos, porcentajes o indicadores ficticios que parezcan métricas reales.
5. Mostrar siempre la evidencia que respalda una alerta y permitir rastrear su origen.
6. Diseñar una interfaz que pueda crecer sin necesidad de reconstruir toda la aplicación.
7. No depender solo del color para comunicar severidad: acompañarlo siempre con texto o íconos (accesibilidad).

La referencia visual representa la dirección deseada del producto a largo plazo. El MVP no necesita implementar todos estos componentes de inmediato. La interfaz debe crecer junto con las capacidades reales del sistema.

## 4. Alcance inicial: MVP (Fase 1)

La primera versión se centrará en AWS CloudTrail y utilizará eventos sintéticos etiquetados para desarrollar y evaluar un motor de detección reproducible.

### Funcionalidades iniciales

**1. Ingesta de eventos**

Leer registros de CloudTrail en formato JSON (estructura `{"Records": [...]}`, como los entrega CloudTrail en S3). Inicialmente se utilizarán conjuntos de datos sintéticos, sin conectar una cuenta AWS real. Los registros inválidos se rechazan y se reportan; no detienen el procesamiento del resto.

**2. Normalización**

Convertir cada registro a un esquema común `NormalizedEvent`, preservando la trazabilidad y evitando que las reglas dependan directamente de la estructura del JSON original.

**3. Detección determinística**

Implementar reglas explícitas para identificar patrones sospechosos. Las primeras candidatas son:

- Uso de la cuenta root, con evaluación adicional del uso de MFA cuando la información esté disponible.
- Múltiples intentos fallidos de inicio de sesión en la consola desde una misma IP, con escalamiento si luego hay un inicio de sesión exitoso.
- Intentos de detener, eliminar o modificar la configuración de CloudTrail que puedan afectar la monitorización.

Cada regla deberá tener una lógica documentada, condiciones de activación, severidad, falsos positivos conocidos y pruebas automatizadas. Las reglas se basan en los controles de monitoreo del CIS AWS Foundations Benchmark.

**4. Evidencia y hallazgos**

Cada detección debe producir un hallazgo estructurado que incluya, como mínimo:

- Identificador de la regla.
- Severidad.
- Motivo de la detección.
- Eventos relacionados (identificadores).
- Evidencia verificable.
- Marca temporal.
- Identificador de la investigación (opcional; se definirá cuando exista agrupación de hallazgos).

**5. Evaluación**

Crear pruebas y conjuntos de datos etiquetados para medir la calidad del detector. Evaluar precisión, recall y falsos positivos por regla. Las etiquetas (verdad de referencia) se almacenan **separadas** de los eventos, para que el detector nunca pueda acceder a ellas.

**6. Interfaz inicial**

Una vez que el motor produzca resultados fiables, construir una interfaz web para consultar las detecciones, filtrar hallazgos y explorar su evidencia. El diseño debe seguir la dirección visual establecida, pero solo mostrará métricas y funcionalidades realmente implementadas.

### Fuera de alcance de la Fase 1

- Conexión a cuentas AWS reales.
- Procesamiento en tiempo real.
- Componentes de IA (LLM, RAG, agentes).
- Soporte multi-cliente y roles de usuario.
- Respuesta automática a incidentes.

### Definición de terminado (Fase 1)

La Fase 1 se considera terminada cuando:

1. Las 3 reglas iniciales están implementadas, documentadas y probadas (casos positivos y negativos).
2. Existe un dataset sintético etiquetado y reproducible.
3. Se reportan precisión y recall por regla.
4. Cada hallazgo puede rastrearse hasta sus eventos de origen.
5. El repositorio incluye README, ADRs y limitaciones conocidas.

Todo lo posterior a este punto se considera una extensión del proyecto, no un requisito para darlo por terminado.

## 5. Arquitectura conceptual

La arquitectura deberá separar las responsabilidades de procesamiento, detección, explicación y presentación. Cada etapa recibe y entrega datos con un contrato definido.

```
CloudTrail JSON
      │
Ingestion & Validation
      │
Normalization Layer ──────► NormalizedEvent
      │
Detection Engine (reglas determinísticas)
      │
Findings & Evidence ──────► Evaluation ◄── Etiquetas (verdad de referencia)
      │
AI Analysis Layer (futuro: explica, nunca detecta)
      │
Application API
      │
Security Dashboard
```

El flujo inicial se implementará de forma local y controlada. La arquitectura podrá evolucionar hacia servicios administrados de AWS cuando exista una necesidad concreta.

La elección de frameworks, servicios y patrones de arquitectura deberá justificarse por requisitos reales, no por la cantidad de tecnologías que sea posible incorporar.

## 6. Rol de la inteligencia artificial

La IA debe complementar el motor de detección, no reemplazarlo. **La IA nunca decide si hubo una detección**: solo trabaja sobre hallazgos ya producidos por el motor.

### Responsabilidades de la IA

- Explicar por qué un hallazgo merece atención.
- Resumir una secuencia de eventos relacionados.
- Diferenciar hechos observados de hipótesis.
- Identificar información faltante.
- Recomendar próximos pasos de investigación.
- Consultar documentación y procedimientos de seguridad en fases posteriores.
- Utilizar herramientas de investigación autorizadas cuando el sistema incorpore capacidades de agente.

### Límites y garantías

- El LLM no debe inventar evidencia ni presentar hipótesis como hechos confirmados.
- Las conclusiones deben estar vinculadas a eventos y hallazgos verificables.
- Las respuestas estructuradas deben validarse antes de ser utilizadas por la aplicación.
- Los registros y documentos recuperados deben tratarse como datos no confiables, nunca como instrucciones privilegiadas. Campos como `userAgent` o `requestParameters` pueden ser controlados por un atacante.
- Las acciones sensibles deben requerir autorización humana.
- Las herramientas de un futuro agente deben tener permisos mínimos y alcance explícito.
- Cualquier componente de IA debe demostrar, con métricas, una mejora frente a la línea base sin IA.

## 7. Evolución tecnológica prevista

Las tecnologías se incorporarán progresivamente cuando resuelvan una necesidad identificada.

### Fase inicial: motor de detección

- Python.
- Pydantic para validación y normalización.
- Pytest para pruebas automatizadas.
- Ruff para linting y formato del código.
- JSON y conjuntos de datos sintéticos.
- Reglas determinísticas y métricas de evaluación.

### Fase de asistencia con IA

- Un LLM para generar explicaciones basadas en evidencia.
- Salidas estructuradas y validación de respuestas.
- Evaluaciones de calidad, fidelidad y afirmaciones no respaldadas (juez de una familia de modelos distinta a la del generador).

### Fase de conocimiento y agentes

- RAG sobre documentación y procedimientos de seguridad.
- Herramientas de investigación de solo lectura.
- LangChain, LangGraph, MCP y LangSmith cuando su incorporación esté justificada por necesidades de integración, orquestación, interoperabilidad o evaluación.

Estas tecnologías no forman parte de los requisitos obligatorios del MVP.

### Fase cloud

- Servicios administrados de AWS, seleccionados según los requisitos de la aplicación.
- IAM y principio de mínimo privilegio.
- Cifrado, gestión de secretos, logging y monitorización.
- Infraestructura como código con Terraform.
- Despliegue reproducible, destrucción de infraestructura cuando no se usa y control de costos.

## 8. Seguridad, privacidad y confiabilidad

La seguridad debe ser una propiedad del sistema desde el comienzo.

- Utilizar datos sintéticos durante el desarrollo inicial.
- No almacenar credenciales ni secretos en el repositorio.
- Validar los registros antes de procesarlos.
- Limitar el acceso a los eventos originales.
- Evitar enviar registros completos al LLM cuando no sean necesarios.
- Mantener trazabilidad entre hallazgos y evidencia original.
- Aplicar permisos mínimos a los servicios y herramientas.
- Operar en modo de solo lectura: no ejecutar acciones destructivas ni modificar recursos de forma autónoma.
- Documentar limitaciones, supuestos y falsos positivos conocidos.

## 9. Riesgos conocidos

| Riesgo | Mitigación |
|---|---|
| Dataset sintético demasiado "fácil", con métricas engañosamente altas | Incluir casos ambiguos y falsos positivos realistas (por ejemplo, un administrador que se equivoca de contraseña) |
| Crecimiento descontrolado del alcance | Definición de terminado explícita para la Fase 1 |
| Prompt injection a través de campos de logs | Tratar los logs como datos no confiables; enviar hallazgos agregados al LLM |
| Una interfaz atractiva que sugiere capacidades inexistentes | Regla de honestidad visual |
| Costos de AWS inesperados | Desarrollo local primero; infraestructura efímera con Terraform |

## 10. Criterios de éxito

El proyecto no se considerará exitoso únicamente porque produzca alertas o presente un dashboard atractivo.

Su calidad se evaluará mediante:

- Correcta normalización de los eventos contemplados.
- Reglas determinísticas reproducibles.
- Pruebas automatizadas que cubran casos positivos y negativos.
- Precisión y recall medidos sobre conjuntos de datos etiquetados.
- Trazabilidad completa de las alertas hasta su evidencia.
- Explicaciones de IA fieles a los hallazgos y sin afirmaciones no respaldadas.
- Controles de seguridad apropiados para las capacidades implementadas.
- Interfaz coherente con los datos reales que el sistema puede proporcionar.
- Documentación clara de la arquitectura, decisiones técnicas y limitaciones.

## 11. Forma de trabajo

El desarrollo será incremental y guiado por decisiones técnicas explícitas.

Antes de implementar una nueva etapa:

1. Explicar el problema que se busca resolver.
2. Definir el resultado esperado.
3. Comparar las alternativas relevantes.
4. Elegir la solución y justificarla.
5. Registrar la decisión en un ADR (`docs/adr/`).
6. Implementar una unidad pequeña y comprensible.
7. Probarla y documentar sus limitaciones.
8. Continuar únicamente cuando la etapa actual esté suficientemente validada.

Los asistentes de IA deben actuar como colaboradores técnicos y mentores, explicando el propósito del código generado y las decisiones que lo justifican. No deben producir grandes cantidades de código o agregar frameworks sin que exista un objetivo claro.

### Etapas de diseño de la Fase 1

| # | Etapa | Estado |
|---|---|---|
| 1 | Límites y arquitectura general | ADR-0001 propuesto |
| 2 | Modelo de datos (`NormalizedEvent`) | Pendiente (existe un borrador previo, a revisar) |
| 3 | Dataset sintético y verdad de referencia | Pendiente |
| 4 | Motor de reglas | Pendiente |
| 5 | Hallazgos y evidencia | Pendiente |
| 6 | Evaluación | Pendiente |
| 7 | Dónde entra la IA | Pendiente |

## 12. Principio rector

**Build a reliable security engine first. Then turn it into an AI-assisted investigation platform.**

Primero, detecciones confiables y medibles. Después, explicaciones fundamentadas. Finalmente, agentes, integraciones cloud y una experiencia de usuario profesional.

CyberSec AI debe crecer a partir de evidencia y resultados verificables, no de complejidad técnica innecesaria.
