# LUMI Guardián AI

Prototipo de monitoreo de seguridad para servidores VPS que alojan tiendas en línea con WordPress. Un agente en Python vigila el sistema, la autenticación SSH y el servidor web, envía los eventos a un backend central y, cuando el backend lo ordena, bloquea direcciones IP con el firewall UFW.

Proyecto colaborativo desarrollado para el concurso de innovación **WomenCISO / MenCISO** (IA y ciberseguridad).

> **Estado:** prototipo funcional, no un producto listo para producción. Se verificó de punta a punta en un laboratorio local y corrió unas 20 horas como servicio en un servidor real expuesto a Internet. El proyecto no fue seleccionado para las semifinales. Las limitaciones están documentadas en [`lumi-docs/TECHNICAL_LIMITATIONS.md`](lumi-docs/TECHNICAL_LIMITATIONS.md).

Este README describe el **snapshot técnico final (7 de septiembre de 2026)**, no solo la idea original. Las diferencias entre ambos se explican abajo.

---

## Propósito original vs. snapshot final

| Aspecto | Propuesta original | Snapshot final |
|---|---|---|
| Detección | Reglas más una capa de detección de anomalías con ML | **Solo reglas.** El ML salió del alcance porque nadie en el equipo pudo desarrollarlo a tiempo |
| Autonomía | Tres niveles: alertar, sugerir o bloquear automáticamente | Existe lógica autónoma en el backend; el alcance verificado en pruebas es el de las alertas y los bloqueos ordenados por WebSocket *(por confirmar el detalle de los tres niveles)* |
| Reputación de IPs | Contemplada en la descripción del ataque (VirusTotal) | **No implementada** |
| Comunicaciones | — | **Sin TLS** |
| Contención de recursos | Se documentaron cifras de consumo bajo | **Sin límites de CPU ni RAM.** El consumo medido (RSS) fue de ~51 MB, frente a los 25–30 MB documentados |
| Despliegue | Varios servidores | Consolidado en un solo servidor |

El nombre conserva "AI", pero el sistema final **no incluye aprendizaje automático**.

---

## Qué hace

- Monitorea métricas del sistema (CPU, RAM, disco, conexiones, procesos de alta carga).
- Detecta fuerza bruta por SSH, sondeo de rutas sensibles de WordPress, escáneres conocidos por *user-agent* y ráfagas de peticiones desde una misma IP.
- Guarda los eventos en una base SQLite local antes de enviarlos al backend.
- Muestra alertas, bloqueos y estado de los VPS en un panel web.
- Ejecuta bloqueos y desbloqueos de IP con UFW cuando el backend envía el comando.

## Flujo resumido

```
Linux VPS ──► Agente Python ──► Cola acotada ──► SQLite (WAL) ──► Backend NestJS ──► Dashboard Angular
(SSH, web,     (3 monitores,     (1.000 eventos)   (búfer local)    (API, Postgres,     (alertas, bloqueos,
 sistema)       hilos)                                               Socket.IO)           VPS)
                     ▲                                                    │
                     └────────── comando bloquear-ip / desbloquear-ip ────┘
                                  (WebSocket → UFW)
```

Detalle completo en [`lumi-docs/SYSTEM_ARCHITECTURE.md`](lumi-docs/SYSTEM_ARCHITECTURE.md).

## Tecnologías

| Capa | Tecnología |
|---|---|
| Agente | Python 3.11+, psutil, requests, python-socketio, SQLite |
| Backend | NestJS, TypeScript, PostgreSQL, Prisma, Socket.IO |
| Panel | Angular |
| Despliegue | Docker, Docker Compose, Nginx |
| Instalador del agente | Bash (`install.sh`), servicio systemd |
| Firewall | UFW |

## Alcance

**Incluye:** telemetría y detección basada en reglas en VPS Ubuntu con WordPress, alertas, y bloqueo de IP controlado por el backend.

**No incluye:** detección de anomalías con ML, consulta de reputación de IP, TLS, caducidad de reglas del firewall ni límites de recursos.

## Privacidad

El agente solo procesa métricas de rendimiento, eventos de autenticación SSH y metadatos de las peticiones HTTP. No accede al código de la aplicación, a las bases de datos de los clientes, al correo ni al cuerpo de las peticiones.

---

## Autoría y contribuciones

Este es un proyecto de equipo. Empezó con siete roles definidos y terminó con cuatro miembros activos.

| Área | Responsable |
|---|---|
| Pruebas de punta a punta, validación operativa, integración y análisis de logs | **[Tu nombre]** |
| Backend, panel de control e instalador | Compañero de backend e interfaz web |
| Documentación de ataques, *fail2ban* y simulación con Caldera | Compañero especializado en seguridad |
| Producto, UX y modelo de negocio | Otros miembros del equipo |

**Asistencia de IA:** la mayor parte del código del agente se generó en sesiones supervisadas con un asistente de IA. Mi aporte fue dirigir el trabajo, ejecutar y probar el código, analizar resultados, aplicar y depurar cambios y reportar los defectos encontrados. No se afirma que el agente haya sido desarrollado de forma individual.

Más detalle en [`lumi-docs/CONTRIBUTIONS.md`](lumi-docs/CONTRIBUTIONS.md).

---

## Documentación

| Documento | Contenido |
|---|---|
| [`PROJECT_OVERVIEW.md`](lumi-docs/PROJECT_OVERVIEW.md) | Contexto, problema y decisiones de alcance |
| [`SYSTEM_ARCHITECTURE.md`](lumi-docs/SYSTEM_ARCHITECTURE.md) | Componentes, flujo, reglas de detección y salvaguardas del firewall |
| [`CONTRIBUTIONS.md`](lumi-docs/CONTRIBUTIONS.md) | Roles, trabajo de otros miembros y uso de IA |
| [`TECHNICAL_DECISIONS.md`](lumi-docs/TECHNICAL_DECISIONS.md) | Elecciones técnicas y sus motivos |
| [`TECHNICAL_LIMITATIONS.md`](lumi-docs/TECHNICAL_LIMITATIONS.md) | Problemas conocidos, con etiqueta de nivel de evidencia |
| [`LESSONS_LEARNED.md`](lumi-docs/LESSONS_LEARNED.md) | Aprendizajes técnicos |
| [`EVIDENCE_INDEX.md`](lumi-docs/EVIDENCE_INDEX.md) | Qué afirmación se apoya en qué evidencia |

## Estructura del repositorio

```
lumi-guardian-ia/
├── README.md
├── agent/        # Agente Python (monitores, búfer, emisor, UFW)
├── server/       # Backend NestJS (API, auth, WebSockets, Prisma)
│   └── public/install.sh
├── client/       # Dashboard Angular
├── deploy/       # Docker Compose y Nginx
└── lumi-docs/    # Documentación técnica
```

## Sobre los datos y cifras

Las cifras provienen de los registros del equipo y del README original, y nadie las ha auditado de forma independiente. El snapshot del 7 de septiembre y el servidor desplegado son estados distintos del proyecto: algunas correcciones se aplicaron directamente en el servidor y no figuran en el repositorio.

## Licencia

*[Pendiente de definir por el equipo.]*
