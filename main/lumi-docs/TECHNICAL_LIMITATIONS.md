# Limitaciones técnicas y problemas conocidos

Cada problema lleva una etiqueta. CONFIRMADO significa que se observó en el código final o en los resultados de pruebas. DEDUCIDO significa que se infirió del código y no se probó. SIN VERIFICAR significa que un documento lo menciona sin pruebas que lo respalden.

## Entrega y fiabilidad

| Problema | Impacto | Etiqueta |
|---|---|---|
| La entrega corre dentro del hilo del consumidor y las respuestas de error HTTP se reintentan sin pausa, incluidos los errores 4xx | Un backend lento o caído bloquea el procesamiento y el *heartbeat*, así que el VPS puede parecer desconectado en pleno ataque | CONFIRMADO |
| SQLite no tiene un indicador de «enviado» | Los eventos que no logran enviarse nunca se reintentan | CONFIRMADO |
| El vaciado del búfer depende solo de su tamaño y los hilos no están configurados como *daemon* | Las alertas pueden quedar retenidas durante minutos, y un hilo bloqueado impide que el proceso termine | CONFIRMADO |
| La corrección del apagado quedó a medias: el emisor acepta `stop_event` pero nunca lo usa | Los reintentos pueden continuar después de que el sistema se apagó | CONFIRMADO |
| El conjunto de idempotencia se vacía por completo al llegar a 1000 entradas, y no hay reintento tras la primera conexión WebSocket | Los comandos antiguos pueden volver a ejecutarse (*replay*), y si el backend no está disponible al arrancar, el canal de control queda inactivo | CONFIRMADO |

## Calidad de la detección

| Problema | Impacto | Etiqueta |
|---|---|---|
| El umbral de ráfagas se calcula por ciclo de lectura y no distingue los archivos estáticos | La carga normal de una página puede disparar una alerta `ip_flood` | CONFIRMADO |
| El estado HTTP se captura pero no se usa | Las redirecciones 301 (benignas) generan alertas, igual que las respuestas 200 (posiblemente expuestas) | CONFIRMADO |
| Los inicios de sesión fallidos se clasifican como `IP_MALICIOSA`, que además es el tipo por defecto | La etiqueta sugiere más gravedad de la que respaldan las pruebas | CONFIRMADO |
| No se envían la técnica MITRE, el país de origen ni la reputación | Los campos de cumplimiento (*compliance*) del backend quedan vacíos | CONFIRMADO |
| `solicitudes por minuto` siempre vale 0, y `procesos activos` solo cuenta los procesos filtrados | Dos métricas del panel son engañosas | CONFIRMADO |
| El umbral SSH es 3 en la instantánea, pero en el entorno local se probó con 5. Los dos patrones podrían coincidir con el mismo intento | Posible sobreconteo | CONFIRMADO / DEDUCIDO |

## Seguridad

| Problema | Impacto | Etiqueta |
|---|---|---|
| El agente llama a `ufw` sin `sudo`, y el instalador lo ejecuta con un usuario sin privilegios de root | Es probable que los bloqueos fallen cuando el servicio esté instalado. Nunca se probó en producción | DEDUCIDO |
| No hay límites de CPU ni RAM (ni restricciones de `systemd` ni medidas de endurecimiento). El consumo de memoria (RSS) medido fue de unos 51 MB, frente a los 25 a 30 MB documentados | Las afirmaciones sobre limitación de recursos no tienen respaldo | CONFIRMADO |
| `permitir_privadas` vale `True` por defecto en `main.py` y la lista blanca viene vacía | La protección contra bloqueos accidentales (*lock-out*) es más débil de lo previsto | CONFIRMADO |
| La configuración valida que los parámetros existan, pero no sus valores (por ejemplo, `"undefined"`) | Una salida defectuosa del instalador puede hacer que el agente arranque en mal estado | CONFIRMADO |
