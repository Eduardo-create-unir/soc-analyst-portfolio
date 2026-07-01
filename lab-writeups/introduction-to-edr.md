# Introduction to EDR — TryHackMe (SOC Level 1)

**Room:** Introduction to EDR
**Ruta del certificado:** SOC Analyst Level 1
**Dificultad:** Fácil
**Duración estimada:** 60 min

## Resumen

Este room introduce el concepto de **Endpoint Detection and Response (EDR)**, una solución de seguridad diseñada para monitorizar, detectar y responder a amenazas avanzadas a nivel de endpoint. A diferencia de un antivirus tradicional, un EDR no se limita a comparar archivos contra firmas conocidas: analiza el comportamiento de procesos, conexiones de red, cambios en el registro y actividad de usuario para detectar amenazas que un antivirus normalmente pasaría por alto.

A lo largo de la sala se explican los fundamentos, se compara el EDR con un antivirus clásico, se detalla su arquitectura (agentes + consola central) y se practica una investigación real sobre alertas generadas en un EDR simulado (CrowdStrike Falcon como referencia visual).

## Objetivos de aprendizaje

- Entender los fundamentos de un **EDR** y cómo funciona.
- Diferenciar un **EDR** de las soluciones de **Antivirus (AV)** tradicionales.
- Examinar la arquitectura de una solución **EDR**.
- Analizar los tipos de **telemetría** que recolecta de los endpoints.
- Comprender las capacidades de **detección** y **respuesta** de un EDR.
- Investigar una alerta realista dentro del EDR.

## Prerrequisitos

- Conocimientos básicos de distintos endpoints (Windows, Linux, Mac) y los ataques comunes sobre ellos.
- Conocer el rol de un equipo **SOC** (Security Operations Center).

---

## Task 2 — ¿Qué es un EDR?

El aumento del uso de dispositivos digitales, sumado a la adopción del trabajo remoto, ha dejado muchos endpoints fuera del perímetro tradicional de la red. Un **EDR (Endpoint Detection and Response)** ofrece protección a nivel profundo del endpoint, sin importar dónde se encuentre, asegurando monitorización constante y detección de amenazas.

Algunas soluciones EDR presentes en el mercado:

- CrowdStrike Falcon
- SentinelOne ActiveEDR
- Microsoft Defender for Endpoint
- OpenEDR
- Symantec EDR

### Los tres pilares de un EDR

Un EDR se sostiene sobre tres capacidades principales:

**Visibilidad → Detección → Respuesta**

#### Visibilidad

Es una de las características que más diferencia al EDR de otras soluciones de seguridad de endpoint. Recolecta datos muy detallados como:

- Modificaciones de procesos
- Modificaciones de registro
- Modificaciones de archivos y carpetas
- Acciones de usuario
- Y mucho más

Toda esta información se presenta en un formato estructurado: el analista puede ver el **árbol completo de procesos** con una línea temporal de acciones, además de acceder a datos históricos de cualquier endpoint para threat hunting.

#### Detección

La capacidad de detección de un EDR supera a la de las soluciones tradicionales porque combina:

- Detecciones basadas en **firmas** (signature-based)
- Detecciones basadas en **comportamiento** (behavior-based), como actividad de usuario inesperada
- **Machine Learning** para identificar desviaciones de la línea base de comportamiento
- Detección de **malware fileless** (que reside solo en memoria)
- Posibilidad de alimentar **IOCs personalizados** para detecciones a medida

Cada detección se muestra con severidad, hora, archivo desencadenante, hostname, usuario, etc., y se mapea contra el framework **MITRE ATT&CK** mediante el campo "Tactic via Technique".

#### Respuesta

El EDR permite actuar directamente sobre la amenaza desde la consola central: aislar un endpoint completo, terminar un proceso, poner en cuarentena archivos o conectarse remotamente al host y ejecutar acciones (por ejemplo, mediante *Real Time Response*).

---

## Task 3 — Más allá del Antivirus

### ¿Por qué necesitamos un EDR si ya tenemos Antivirus?

Ambas soluciones buscan proteger el endpoint, pero difieren en el **nivel** de protección.

**Analogía del aeropuerto:**

- El **Antivirus (AV)** sería el control de inmigración: revisa el pasaporte de cada persona y lo compara contra una base de datos de criminales conocidos. Si hay coincidencia, bloquea la entrada.
- El problema: si alguien nunca ha sido identificado como criminal (aunque en realidad sea una amenaza entrenada para evadir seguridad básica), el control de inmigración lo dejará pasar sin más.
- El **EDR** sería el personal de seguridad dentro del aeropuerto: monitoriza constantemente cámaras y sensores de movimiento. Aunque alguien evada el control de inmigración, seguirá siendo observado — ¿se acerca a zonas restringidas?, ¿su comportamiento es sospechoso?, ¿deja bolsas abandonadas?

En resumen: el **AV** detecta amenazas conocidas por firma; el **EDR** detecta comportamiento anómalo, incluso de amenazas nunca vistas antes.

### Escenario práctico

1. Un usuario recibe un **phishing** con un documento Word que contiene una macro maliciosa (VBA).
2. El usuario descarga y abre el documento.
3. La macro se ejecuta silenciosamente y genera un proceso **PowerShell**.
4. PowerShell ejecuta un comando ofuscado que descarga un payload de segunda etapa.
5. El payload se inyecta en un proceso legítimo, `svchost.exe`.
6. El atacante obtiene acceso remoto al sistema.

| Paso del ataque | Respuesta del AV | Respuesta del EDR |
|---|---|---|
| 1. Descarga del archivo | No hace nada si no hay firma previa en su base de datos | Registra y monitoriza la descarga |
| 2. Apertura del documento | No hace nada porque `winword.exe` es legítimo | Registra la ejecución de `winword.exe` y sigue monitorizando |
| 3. Ejecución de la macro | No hace nada si la macro no tiene firma previa | Detecta la relación padre-hijo inusual entre `winword.exe` y `powershell.exe` |
| 4. Descarga del payload vía PowerShell | Normalmente no detecta scripts PowerShell ofuscados | Marca la ejecución del script ofuscado |
| 5. Inyección en `svchost.exe` | No lo marca porque no monitoriza inyecciones de memoria | Detecta la inyección de proceso en `svchost.exe` |
| 6. Conexión remota | Carece de visibilidad a nivel de red | Marca el comportamiento inesperado de `svchost.exe` al hacer una conexión saliente |
| **Resultado final** | Puede quedar marcado como "limpio" | Genera una alerta con la **cadena de ataque completa** y permite actuar desde la consola |

> **Nota:** algunos AV modernos tienen mayor visibilidad y detección, pero el EDR sigue estando por delante en cuanto a profundidad de detección y respuesta en el endpoint.

---

## Task 4 — ¿Cómo funciona un EDR?

Un EDR se compone principalmente de dos piezas:

### Agentes

Se despliegan en cada endpoint y también se conocen como **sensores**. Son los "ojos y oídos" del EDR: se sitúan en el endpoint y monitorizan toda la actividad, enviando esa información en tiempo real a la consola central. Los agentes pueden realizar detecciones básicas (por firma y por comportamiento) por sí mismos antes de enviarlas a la consola.

### Consola EDR

Toda la información enviada por los agentes se correlaciona y analiza mediante lógica compleja y algoritmos de **machine learning**, cruzándola con inteligencia de amenazas. La consola es literalmente el "cerebro" que conecta todos los puntos: cuando esos puntos se conectan entre sí, se genera una **detección** (alerta).

El dashboard de la consola ofrece una vista global del estado de las detecciones en todos los endpoints: CrowdScore, nuevas detecciones, detecciones basadas en SHA, malware prevenido por host, etc.

---

## Task 5 — Telemetría del EDR

### ¿Qué es la telemetría?

Es toda la información que los agentes EDR recolectan del endpoint y envían a la consola. Se describe como la **"caja negra"** del endpoint: contiene todo lo necesario para detección e investigación.

### Tipos de telemetría recolectada

- **Ejecución y terminación de procesos:** permite identificar relaciones padre-hijo sospechosas, ejecutables inusuales que inician procesos, payloads de malware, etc.
- **Conexiones de red:** ayuda a identificar conexiones a servidores C2, uso de puertos inusuales, exfiltración de datos o movimiento lateral.
- **Actividad de línea de comandos:** captura los comandos ejecutados en CMD, PowerShell, etc., ayudando a identificar ejecuciones maliciosas o scripts PowerShell ofuscados que un antivirus tradicional suele pasar por alto.
- **Modificaciones de archivos y carpetas:** los actores de amenaza suelen modificar archivos durante el data staging, ejecuciones de ransomware o dropping de archivos maliciosos. El EDR lo rastrea.
- **Modificaciones de registro:** el registro es una mina de información sobre la configuración de un sistema Windows; muchos cambios maliciosos se reflejan ahí y el EDR los monitoriza.

Individualmente, muchas de estas actividades pueden parecer inofensivas, pero al observarlas en conjunto y con telemetría detallada, cuentan una historia distinta. Esto no solo ayuda al EDR a detectar amenazas avanzadas, sino que también facilita enormemente el trabajo del analista durante una investigación: entender la cadena completa de eventos, identificar la causa raíz y reconstruir la línea temporal del ataque.

---

## Task 6 — Capacidades de detección y respuesta

En esta tarea se profundiza en las técnicas de detección avanzada y los mecanismos de respuesta del EDR, incluyendo el uso de **IOC Matching** (coincidencia de indicadores de compromiso) para identificar amenazas basadas en comportamientos maliciosos ya conocidos.

---

## Task 7 — Investigar una alerta en el EDR (práctica)

### Escenario

Como analista SOC en **TECH THM**, con acceso a la consola EDR, se presentan múltiples detecciones de severidad media y alta. El objetivo es realizar **triage** sobre cada detección usando la información disponible en el EDR (proceso, línea de comandos, red, threat intel, etc.).

> Nota del room: el reconocimiento y las acciones de remediación sobre las detecciones quedan fuera del alcance; el foco está puesto en entender la **visibilidad** que ofrece el EDR.

### Hallazgos de la investigación

| Pregunta | Respuesta |
|---|---|
| Herramienta lanzada por `CMD.exe` para descargar el payload en `DESKTOP-HR01` | `CURL.exe` |
| Ruta absoluta del malware descargado en `DESKTOP-HR01` | `C:\Users\Public\install.exe` |
| Ruta absoluta del `syncsvc.exe` sospechoso en `WIN-ENG-LAPTOP03` | `C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe` |
| URL del intento de exfiltración en `WIN-ENG-LAPTOP03` | `https://files-wetransfer.com/upload/session/ab12cd34ef56/dump_2025.dmp` |
| Etiqueta de Threat Intel para `UpdateAgent.exe` en `DESKTOP-DEV01` | Known internal IT utility tool |

### Aprendizaje clave de esta tarea

Esta práctica demuestra cómo, gracias a la telemetría detallada (procesos, línea de comandos, rutas de archivos, conexiones de red y contexto de threat intelligence), un analista puede reconstruir rápidamente la cadena de eventos de un ataque, identificar el binario y la ruta exactos utilizados, y correlacionar esa información con inteligencia de amenazas para determinar si un proceso es legítimo o malicioso.

---

## Task 8 — Conclusión

Con este room se cierra el aprendizaje de una de las herramientas esenciales en un **Security Operations Center (SOC)**: el **EDR**. Como analista SOC, ahora entiendo:

- La arquitectura básica de un EDR (agentes + consola).
- Sus capacidades más allá de un Antivirus tradicional.
- El tipo de telemetría detallada que provee.
- Sus capacidades de detección y respuesta.
- Cómo investigar detecciones reales dentro de la consola.

Esta sala sienta una base sólida para seguir explorando otras soluciones de seguridad que un analista SOC utiliza en su día a día.

---

## Notas personales

Este room forma parte de mi progreso en la ruta **SOC Analyst Level 1** de TryHackMe. Lo más valioso para mí fue entender de forma práctica *por qué* un EDR aporta una capa de detección que un antivirus tradicional simplemente no puede ofrecer, y cómo la telemetría (procesos, red, línea de comandos, archivos y registro) es la pieza clave que permite a un analista reconstruir un ataque completo desde la consola.

**Habilidades practicadas:** análisis de telemetría de endpoint, triage de alertas, correlación de eventos, lectura de árboles de procesos, y comprensión de la diferencia práctica entre detección basada en firmas vs. detección basada en comportamiento.
