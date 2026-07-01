# Laboratorio 3 — Spike of Domain Discovery Commands

**Fecha:** Junio 2026 | **Severidad:** High | **Herramienta:** TryHackMe SIEM Simulator
**Veredicto:** True Positive

## Resumen

Salta una alerta por un pico de comandos de reconocimiento de dominio (`whoami`, `net user`, `Get-ADUser`...) en un servidor Exchange que está en la DMZ. En cuanto miro la cadena de procesos, queda claro que esto **no es actividad de un admin**, es un compromiso real.

## Detalles de la alerta

| Campo | Valor |
|---|---|
| Descripción de la regla | Detecta picos de comandos de descubrimiento de dominio (`whoami`, `net user`, `Get-ADUser`) potencialmente indicativos de compromiso |
| Origen | Host `DMZ-MSEXCHANGE-2013` (Windows Server 2012 R2), usuario `NT AUTHORITY\SYSTEM` |
| Destino | N/A (actividad local del host) |
| Datos adicionales | Comandos: `dir`, `hostname`, `whoami /priv`, `net group "Domain Admins" /domain`, `nltest /dclist:tryhackme.thm` · Proceso: `cmd.exe` ← `revshell.exe` (`C:\Users\Public\`) ← `w3wp.exe` |

## Investigación

Lo primero que hago es tirar del **árbol de procesos**, y ahí está la clave de todo: `w3wp.exe` (que es el proceso de IIS/Exchange) tiene como hijo un binario que se llama, literalmente, **`revshell.exe`**, guardado en `C:\Users\Public\`. Una carpeta pública donde cualquiera puede escribir. Eso ya de entrada huele fatal.

Luego miro con qué privilegios corre todo esto: **`NT AUTHORITY\SYSTEM`**. El nivel más alto que hay en Windows. O sea, quien haya entrado, entró con el control total del servidor desde el primer segundo.

Y los comandos que se ejecutaron no son casualidad: `whoami /priv` para ver qué privilegios tiene, `net group "Domain Admins" /domain` para localizar las cuentas más jugosas, y `nltest /dclist` para listar los controladores de dominio. Esto es **reconocimiento de manual**, la fase previa a moverse lateralmente por la red.

Y el servidor está en la **DMZ** — o sea, expuesto a internet. Si lo comprometen ahí, tienen un puente directo hacia la red interna.

## Razonamiento

No hay forma humana de justificar que el proceso de trabajo de IIS de un Exchange lance una reverse shell desde una carpeta pública y encima se ponga a buscar Domain Admins. Con solo ver esa cadena de procesos ya tengo evidencia de sobra — no hace falta cruzar más fuentes. El patrón encaja con explotación de vulnerabilidades conocidas de Exchange (tipo ProxyShell/ProxyLogon).

## Conclusión y acción tomada

**Veredicto final:** True Positive — esto ya no es "sospechoso", es un **compromiso confirmado**
**Acción:** Lo escalo a L2/IR con prioridad máxima. Aquí no hay margen: **hace falta investigación forense**, **hay que aislar el host ya** y quitar el binario, y como el atacante iba a por Domain Admins, el riesgo de que esto escale a todo el dominio es real. Esto se sale claramente de lo que puedo cerrar yo solo como L1.

## Lecciones aprendidas

La cadena de procesos (padre → hijo → abuelo) es de las cosas más reveladoras que hay: un binario con nombre sospechoso colgando de un proceso legítimo como `w3wp.exe` ya es motivo de alarma antes incluso de mirar qué comandos se ejecutaron después. Y aprendo también que la severidad automática (High) se quedó corta — este caso, en la práctica, era mucho más grave de lo que sugería la etiqueta.

---
*Caso documentado como parte de mi preparación para SOC Analyst L1*
