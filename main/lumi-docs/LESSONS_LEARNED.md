# Lecciones aprendidas

1. Un error puede funcionar como técnica de evasión. Un desplazamiento en los grupos de una expresión regular para SSH hizo que los nombres de usuario terminaran en el campo de la dirección IP. Al rotar los nombres de usuario, los conteos se fragmentaban y la detección se podía eludir. Los registros antiguos de la base local todavía muestran este comportamiento.

2. Nunca compares direcciones IP como texto. `"1.2.3.4" in "1.2.3.40"` devuelve verdadero. Al procesar las direcciones con objetos de red desaparecieron los falsos positivos de elementos "ya bloqueados".

3. La defensa en profundidad funcionó. La capa del firewall rechazó los valores malformados que emitía el sensor y contuvo el impacto.

4. Los fallos se propagan en cascada. Un bloqueo en la entrega de datos dentro de un hilo interrumpió el *heartbeat*. Además, un proceso de apagado que ignoró la señal de detención cerró la base de datos mientras todavía había escrituras en curso.

5. Di qué falta por terminar. Documentar las limitaciones resultó más útil que pulir la presentación, y es lo que da credibilidad al resto de la propuesta.
