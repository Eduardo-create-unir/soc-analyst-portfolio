# Laboratorio 2 — Email Marked as Phishing after Delivery

**Fecha:** Junio 2026 | **Severidad:** Critical | **Herramienta:** TryHackMe SIEM Simulator
**Veredicto:** True Positive

## Resumen

Me llegó una alerta **Critical** de un correo que el sistema ya había marcado como phishing tras entregarlo. El email **suplantaba a Microsoft** y usaba un mensaje de urgencia para que el usuario abriera un archivo adjunto.

## Detalles de la alerta

| Campo | Valor |
|---|---|
| Descripción de la regla | Email clasificado como phishing tras análisis automático post-entrega |
| Origen | `support@microsoft.com` (remitente suplantado) |
| Destino | Eddie Huffman, IT Manager (`e.huffman@tryhackme.thm`) |
| Datos adicionales | Asunto: "Important Update: Microsoft Teams Pricing Increase" · **SPF: Fail** · **DKIM: Fail** · Adjunto: `REPORT.rar` · Sin URLs |

## Investigación

Lo primero que miré fue la **autenticación del correo**, y ahí ya salta la primera señal gorda: **tanto SPF como DKIM fallan** sobre el dominio `microsoft.com`. Eso no es un fallo de configuración random, es que **el remitente está suplantado**.

Después me fijé en el cuerpo del mensaje: mete urgencia falsa ("600% price increase", "urgent notice") y te empuja a hacer algo ya ("download the report"). Es el típico gancho de ingeniería social.

Lo curioso es que **no hay ninguna URL**, sino un `.rar` adjunto. Eso también es una pista — muchos filtros de correo revisan mejor los enlaces que los archivos comprimidos, así que es una forma de colarse.

Y el destinatario no es cualquiera: es el **IT Manager**. Eso me hace pensar que no es spam masivo, sino un intento dirigido a alguien con más acceso del normal.

## Razonamiento

Para mí el punto que cierra el caso es el **fallo simultáneo de SPF y DKIM** sobre un dominio con tanta reputación como Microsoft — eso no pasa por error, es suplantación activa. Sumado a la urgencia falsa y al adjunto sin enlace visible, todo apunta a una campaña de malware dirigida, no a un correo mal filtrado.

## Conclusión y acción tomada

**Veredicto final:** True Positive
**Acción:** Lo escalé a L2. ¿Por qué? Porque **hay que hacer algo con esto** (poner en cuarentena, bloquear el remitente, mirar el `.rar` en sandbox), porque **igual el usuario ya lo abrió** y hay que confirmarlo, y porque el objetivo es una cuenta con privilegios — mejor que lo revise alguien más antes de cerrarlo.

## Lecciones aprendidas

Me quedo con que **no hace falta un link para que sea peligroso** — un archivo comprimido cuela igual o mejor. Y que **lo primero que hay que mirar en una alerta de suplantación es SPF/DKIM**, antes de perder tiempo analizando el resto del contenido.

---
*Caso documentado como parte de mi preparación para SOC Analyst L1*
