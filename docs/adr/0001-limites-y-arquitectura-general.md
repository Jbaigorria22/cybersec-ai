# ADR-0001: Límites del sistema y arquitectura general (Fase 1)

- **Estado:** Propuesto
- **Fecha:** 2026-10-29
- **Etapa de diseño:** 1 de 7

## Contexto

CyberSec AI busca ayudar a analizar eventos de seguridad en AWS. Antes de incorporar IA,
necesitamos un detector cuyos resultados se puedan **probar y medir**. Sin esa base,
no hay forma de saber si un componente de IA agrega valor o solo complejidad.

La fuente de datos inicial es **AWS CloudTrail**, porque registra la actividad del
plano de control de AWS (quién hizo qué, desde dónde y con qué resultado) y se alinea
con un perfil Cloud + Security.

## Decisiones

### D1. Procesamiento por lotes (archivos), no en tiempo real
El sistema procesa archivos JSON con el formato que CloudTrail entrega en S3.
- **Por qué:** con la misma entrada se obtiene siempre la misma salida, así que las pruebas son repetibles.
- **Alternativa descartada:** consumir eventos en vivo con EventBridge. Agrega infraestructura y dificulta las pruebas sin aportar aprendizaje en esta fase.

### D2. Pipeline de etapas desacopladas con contratos
`Ingesta → Normalización → Detección → Hallazgos → Evaluación`.
Cada etapa recibe y entrega datos con una forma definida y hace una sola cosa.
- **Por qué:** cada etapa se prueba por separado y puede reemplazarse sin afectar a las demás.
- **Alternativa descartada:** un script monolítico. Es más rápido de escribir, pero imposible de probar por partes y difícil de extender.

### D3. Sistema de solo lectura
El sistema observa e informa. No bloquea usuarios, no modifica recursos y no se conecta a cuentas reales en esta fase.
- **Por qué:** un detector que puede actuar es un objetivo valioso para un atacante. Esto cumple el requisito de aprobación humana para acciones consecuentes.
- **Alternativa descartada:** respuesta automática (por ejemplo, deshabilitar credenciales). Queda para una fase futura, siempre con aprobación humana.

### D4. Detección determinística
Las reglas no usan aleatoriedad, heurísticas probabilísticas ni modelos de IA.
- **Por qué:** las métricas (precisión, recall) solo tienen sentido si el resultado es reproducible. Esto establece la **línea base** contra la que se comparará cualquier componente de IA.
- **Alternativa descartada:** detección asistida por LLM. No es reproducible ni medible como punto de partida.

## Fuera de alcance (Fase 1)

Conexión a cuentas AWS reales, procesamiento en tiempo real, interfaz gráfica,
componentes de IA, soporte multi-cliente.

## Consecuencias

- **Positivas:** sistema testeable y medible; la IA podrá agregarse luego como una etapa más, después de "Hallazgos", sin modificar la detección.
- **Negativas / aceptadas:** no detecta en tiempo real; depende de datos sintéticos hasta fases posteriores, por lo que las métricas reflejan el dataset, no un entorno real.
- **Riesgo a vigilar:** que el dataset sintético sea demasiado "fácil" y las métricas den una falsa sensación de calidad. Se aborda en la Etapa 3.

## Dónde se conecta

- **Etapa 2 (Modelo de datos):** define el contrato entre Normalización y Detección.
- **Etapa 3 (Dataset):** alimenta la Ingesta y la Evaluación.
- **Etapa 7 (IA):** se ubica después de Hallazgos y nunca decide si hubo detección.
