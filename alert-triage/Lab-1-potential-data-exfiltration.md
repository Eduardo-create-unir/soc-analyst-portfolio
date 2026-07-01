# Laboratorio 1 — Potential Data Exfiltration

**Fecha:** Junio 2026 | **Severidad:** Critical | **Herramienta:** TryHackMe SOC Simulator
**Veredicto:** False Positive

## Resumen

Alerta crítica disparada por una regla que detecta envíos de 5GB o más desde un único dispositivo a un mismo destino en 24h, lo cual puede indicar exfiltración de datos hacia un destino no confiable.

## Detalles de la alerta

| Campo | Valor |
|---|---|
| Descripción de la regla | Detecta ≥5GB enviados desde un dispositivo a un destino único en un día |
| Origen | 192.168.45.66 (Red: UK04/MEETINGROOM) |
| Destino | *.zoom.us |
| Datos adicionales | Enviados: 5.8 GB / Recibidos: 5.2 GB |

## Investigación

- El destino (`*.zoom.us`) es un dominio corporativo legítimo y ampliamente usado, no un dominio desconocido o de baja reputación.
- El origen es una red identificada como "MEETINGROOM", consistente con un dispositivo de videoconferencia.
- El tráfico es prácticamente simétrico (5.8GB enviados vs 5.2GB recibidos), un patrón típico de videollamada bidireccional en tiempo real, no de exfiltración (que normalmente muestra tráfico saliente predominante hacia un destino sin tráfico de retorno significativo).

## Razonamiento

La combinación de (a) destino legítimo y conocido, (b) origen consistente con uso esperado del dispositivo, y (c) patrón de tráfico simétrico, apunta a una videollamada larga o con múltiples participantes en lugar de una transferencia de datos no autorizada. Una exfiltración real normalmente implicaría un destino externo no corporativo y un patrón de tráfico marcadamente asimétrico (mucho más saliente que entrante).

## Conclusión y acción tomada

**Veredicto final:** False Positive
**Acción:** Cerrado tras documentar el razonamiento. No se requiere escalado a L2.

## Lecciones aprendidas

El volumen de datos por sí solo no es indicador suficiente de exfiltración; hay que contextualizar siempre con el destino y la simetría del tráfico. Las reglas basadas solo en umbral de GB transferidos generan muchos falsos positivos con herramientas de colaboración (videoconferencia, almacenamiento en la nube corporativo, etc.).

---
*Caso documentado como parte de mi preparación para SOC Analyst L1*
