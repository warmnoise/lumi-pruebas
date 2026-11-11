# LUMI Guardián AI: arquitectura del sistema

Estado final al 7 de septiembre de 2026. Este documento describe lo que hace el código, no lo que se planeó al inicio.

## Visión general

Un agente en Python corre en el VPS, vigila el sistema, SSH y el servidor web, y guarda los eventos en una base SQLite local antes de enviarlos a un backend NestJS. El backend ofrece la API REST, la autenticación y la persistencia en PostgreSQL, y muestra los resultados en un panel hecho en Angular. Cuando se ordena un bloqueo, el backend manda el comando por WebSocket y el agente lo ejecuta con UFW. Nginx y Docker Compose se encargan del despliegue.

## Componentes

| Componente | Responsabilidad | Tecnología |
|---|---|---|
| `agent/` | Monitoreo, búfer local, envío de datos y control del firewall | Python 3.11+, psutil, requests, python-socketio, SQLite |
| `server/` | API REST, autenticación, lógica autónoma, persistencia y WebSockets | NestJS, TypeScript, PostgreSQL, Prisma, Socket.IO |
| `client/` | Panel de control (alertas, bloqueos, VPS y configuración) | Angular |
| `deploy/` | Proxy inverso y configuración con Docker Compose | Docker, Nginx |
| `server/public/install.sh` | Instalador del agente con un solo comando, como servicio systemd | Bash |

## Flujo del agente

1. Tres monitores heredan de `BaseMonitor` y cada uno corre en su propio hilo, con un ciclo de 5 segundos por defecto.
2. Los eventos entran en una cola acotada de 1.000 elementos. Si la cola se llena, los eventos se descartan y se registra la saturación (*back-pressure*).
3. El consumidor agrupa los eventos en una base SQLite en modo WAL, con retención de 30 días y un límite de tamaño.
4. El emisor envía métricas y alertas al backend, con reintentos y retroceso exponencial, y manda un *heartbeat* cada 60 segundos.
5. Un cliente Socket.IO recibe los comandos `bloquear-ip` y `desbloquear-ip` y responde con un evento que indica el resultado. El backend tiene un tiempo de espera de 5 segundos.

## Reglas de detección (estado final)

| Origen | Regla | Evento |
|---|---|---|
| SSH | Cada línea de contraseña fallida o usuario no válido | `ssh_failed_login` |
| SSH | 3 intentos fallidos desde una misma IP en 5 minutos | `ssh_brute_force_burst` (crítico) |
| SSH | 7 intentos fallidos desde una misma IP en 15 minutos | `ssh_brute_force_persistent` |
| HTTP | La ruta de la petición empieza con una ruta sensible de WordPress (por ejemplo, `/wp-json/wp/v2/users` o `/wp-admin`) | `wp_sensitive_route` |
| HTTP | El *user-agent* contiene el nombre de un escáner conocido | `scanner_detected` |
| HTTP | Más de 20 peticiones desde una misma IP en un ciclo de lectura | `ip_flood` |
| Sistema | Muestreo en cada ciclo de CPU, RAM, disco, conexiones y procesos de alta carga | `system_sample` |

## Salvaguardas del firewall (`ufw_manager.py`)

- Valida IP y CIDR con el módulo `ipaddress`.
- Ejecuta los comandos como lista de argumentos, sin *shell*.
- Rechaza direcciones *loopback*, *link-local*, *multicast* y reservadas, prefijos demasiado amplios y, según la configuración, rangos privados.
- Comprueba si la dirección se solapa con la lista blanca.
- Es idempotente: consulta `ufw status` en tiempo real antes de actuar.
- Inserta las reglas en la posición 1.
- Permite 30 operaciones por minuto y un máximo de 500 reglas.
- Aplica un tiempo de espera propio a cada comando.

## No implementado

- Detección de anomalías con aprendizaje automático.
- Consulta de reputación de IP. VirusTotal solo aparece en la descripción del ataque.
- TLS.
- Caducidad de reglas.
- Límites de CPU y RAM por recurso.
