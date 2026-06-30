# Entrega Final — Ecosistema de Automatización IA
## Clasificación de Leads VIP — OVO Market

Sistema de automatización que captura leads, los clasifica con IA según
intención de compra, y reserva el contacto directo por WhatsApp solo para
leads VIP aprobados por un humano (Human-in-the-Loop).

---

## 1. Caso de uso
OVO Market (reseller de Apple y Samsung) recibe consultas de potenciales
clientes. El sistema interpreta en lenguaje natural cada consulta, clasifica
el lead como VIP o Estándar según presupuesto e intención, y deja el contacto
final supervisado por un humano para evitar el "Efecto Metralleta".

## 2. El Cerebro — Base de datos (Airtable)
🔗 **Link en modo lectura:** https://airtable.com/app6xDi1tdyjUjv0W/shr1iAyt3pMb7vSm6

La base "OVO Market - CRM Leads" tiene dos tablas relacionadas:
- **Leads:** campos de estado (`Pendiente` → `Procesado por IA` →
  `Aprobado por Humano` → `Cerrado`), `Clasificación IA` (VIP / Estándar),
  `Presupuesto USD` (tipo número) y `Producto de Interes` (campo vinculado).
- **Productos:** catálogo de OVO, relacionado con Leads mediante un campo de
  enlace bidireccional, evitando datos aislados.

## 3. El Corazón — Orquestación (Make)
📄 **Blueprint:** `/blueprint/escenario-leads.json`

- **Trigger inteligente:** Google Sheets "Watch New Rows" alimentado por un
  Google Form, configurado para no consumir operaciones innecesarias.
- **Motor de IA:** módulo OpenAI con prompt dinámico que usa variables del
  sistema (datos del lead), mapeo de la respuesta y `max_tokens` limitado a 5
  para optimizar costos.
- **Router:** divide el flujo en "Prioridad Alta" y "Prioridad Baja".
- **Filtros:** validan la palabra exacta devuelta por la IA con
  `trim(upper(Result))` para comparar tipos correctos (texto vs texto).
- **Gestión de errores:** módulo Retry (Break) con 3 reintentos automáticos.
  Si la API de IA falla, guarda la ejecución como incompleta en lugar de
  romper el flujo.

## 4. Human-in-the-Loop
Antes de contactar al cliente, el flujo se detiene en el estado
`Aprobado por Humano`: el sistema espera la validación manual (checkbox /
aprobación vía Slack) antes de avanzar a la salida por WhatsApp.

## 5. La Voz — Salida WhatsApp
El contacto VIP se realiza por WhatsApp API usando una plantilla (Template)
aprobada y el número en formato internacional con prefijo `+`.

---

## Check de Seguridad
1. **Anti-bucle infinito:** el trigger "From now on" y los filtros por estado
   evitan reprocesar la misma fila.
2. **Comparación de tipos correctos:** los filtros comparan texto con texto
   (Estado, Clasificación) y número con número (Presupuesto USD).
3. **Prompt dinámico:** el prompt usa variables del sistema, no texto fijo.

## Evidencias (carpeta /evidencias)
- `01-flujo-completo.png` — escenario completo en Make (todos los módulos).
- `02-ejecucion.png` — ejecución del flujo (trigger leyendo una fila).
- `03-prompt-ia.png` — módulo OpenAI con el prompt dinámico configurado.
- `04-retry-3-reintentos.png` — error handler Retry con 3 reintentos.
- `05-error-handling-incomplete.png` — el sistema captura un fallo de la API
  (error 429) y guarda la ejecución como incompleta: resiliencia demostrada
  (test del "camino infeliz").
- `06-airtable-leads.png` — tabla Leads con los campos de estado y clasificación.
- `07-airtable-productos-relacion.png` — tabla Productos mostrando la relación
  bidireccional con Leads.

## Stack técnico
Google Forms · Google Sheets · Airtable · Make · OpenAI (GPT-4o-mini) ·
WhatsApp API
