# LUMI Guardián AI: resumen del proyecto

Este resumen se basa en la instantánea final del repositorio (7 de septiembre de 2026) y se escribió en noviembre de 2026 como retrospectiva. Las cifras vienen de los registros del equipo y del README, y nadie las ha auditado de forma independiente.

## Qué es

LUMI Guardián AI monitorea la seguridad de servidores VPS que alojan tiendas en línea con WordPress. Un agente en Python corre en el servidor y vigila las métricas del sistema, los registros de autenticación SSH y los registros de acceso del servidor web, y envía lo que observa a un backend central. Cuando el backend se lo pide, el agente bloquea una dirección IP con el firewall UFW.

## Problema que buscaba resolver

Los dueños de tiendas pequeñas suelen operar su negocio en un VPS sin nadie de seguridad. Esos servidores reciben ataques automatizados todo el tiempo, como fuerza bruta por SSH o sondeo de rutas de WordPress, y el dueño casi siempre se entera tarde. LUMI quería detectar esos ataques y explicarlos en lenguaje sencillo, con tres niveles de autonomía: solo alertar, sugerir una acción o bloquear automáticamente.

## Decisiones de alcance

La detección funciona con reglas y no usa aprendizaje automático. El diseño original incluía una capa de detección de anomalías con ML, pero salió del alcance porque nadie en el equipo pudo desarrollarla a tiempo, como consta en la especificación técnica. El producto conserva "AI" en el nombre.

El MVP apunta a servidores VPS con Ubuntu y Apache/WordPress, y su primera fase cubre telemetría y detección.

En privacidad, el agente solo procesa métricas de rendimiento, eventos de autenticación SSH y metadatos de las peticiones HTTP. No accede al código de la aplicación, a las bases de datos de los clientes, al correo ni al cuerpo de las peticiones.

## Contexto

Un equipo multidisciplinario desarrolló el proyecto para el concurso de innovación WomenCISO / MenCISO (IA y ciberseguridad). Empezó con siete roles definidos y terminó con cuatro miembros activos. No fue seleccionado para las semifinales.

El resultado es un prototipo funcional. Se verificó de punta a punta en un laboratorio local y corrió unas 20 horas como servicio en un servidor real expuesto a Internet. Sus limitaciones están documentadas.
