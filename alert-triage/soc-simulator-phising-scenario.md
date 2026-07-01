# SOC Simulator — Phishing Scenario (TryHackMe)

**Plataforma:** TryHackMe — SOC Simulator
**Rol:** Analista SOC L1 (Alert Triage)
**Resultado:** ✅ Victory — Security breach prevented
**Fecha:** Julio 2026

---

## 📋 Resumen del escenario

Escenario de triage de alertas centrado en un ataque de **phishing** con enlace externo sospechoso, combinado con una alerta de firewall por acceso a URL en lista negra. El objetivo era clasificar correctamente cada alerta como verdadero positivo o falso positivo, priorizando por severidad y tiempo de respuesta.

## 📊 Métricas obtenidas

| Métrica | Resultado |
|---|---|
| Alertas cerradas | 4 |
| Tiempo medio de resolución (MTTR) | 3 minutos |
| Dwell time medio | 5 minutos |
| Clasificaciones correctas | 3 / 4 |
| Resultado final | Victory (breach prevented) |

## 🔍 Alertas investigadas

| ID | Regla de alerta | Severidad | Tipo | Tiempo resolución | Clasificación |
|---|---|---|---|---|---|
| 8815 | Inbound Email Containing Suspicious External Link | Medium | Phishing | 3.52 min | ✅ Correct |
| 8814 | Inbound Email Containing Suspicious External Link | Medium | Phishing | 3.23 min | ❌ Incorrect |
| 8817 | Inbound Email Containing Suspicious External Link | Medium | Phishing | 2.90 min | ✅ Correct |
| 8816 | Access to Blacklisted External URL Blocked by Firewall | High | Firewall | 2.55 min | ✅ Correct |

### Metodología aplicada en las alertas correctas (8815, 8816, 8817)

- Revisión de cabeceras del correo y del dominio del enlace externo para identificar indicadores de phishing (dominio recién registrado, discrepancia entre remitente y dominio del enlace, ausencia de SPF/DKIM/DMARC válidos).
- Correlación con la alerta de firewall (ID 8816), que confirmó el bloqueo del acceso a la URL maliciosa, reforzando la clasificación como verdadero positivo.
- Priorización de la alerta de severidad **High** (8816) sobre las de severidad **Medium**, alineado con el proceso estándar de triage en un SOC.

### Lección aprendida

Una de las cuatro alertas (ID 8814) fue clasificada incorrectamente. Sirvió para reforzar la importancia de no generalizar el patrón de las alertas repetidas (mismas reglas de detección no siempre implican el mismo veredicto) y de validar cada indicador de forma individual antes de aplicar una conclusión ya usada en una alerta similar.

## 🎯 Progresión

Escenario superado con desbloqueo del siguiente módulo:

**Execution: T1204** — Investigación de la técnica *T1204.002: User Execution: Malicious File*, determinando si la actividad observada es benigna o maliciosa.

---

## 🧠 Habilidades demostradas

- Triage y priorización de alertas por severidad (SOC L1)
- Análisis de indicadores de phishing (email + URL)
- Correlación de eventos entre distintas fuentes (email gateway + firewall)
- Gestión de tiempos de resolución (MTTR) y dwell time
- Aprendizaje iterativo a partir de errores de clasificación

---

*Parte de mi proceso de preparación hacia CompTIA Security+ y TryHackMe SAL1, documentado como parte de mi portfolio público de analista SOC.*
