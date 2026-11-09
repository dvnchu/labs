# Labs — Dante Culini

Notas y writeups de laboratorios propios y de entornos de práctica autorizados
(Hack The Box, TryHackMe, rangos locales).

Todo el contenido corresponde a entornos que administro o a plataformas que
autorizan explícitamente la práctica. Las IPs de ejemplos públicos usan rangos de
documentación (RFC 5737).

## Contenido

### Homelab
- **[Proxmox + pfSense: virtualización, segmentación y acceso remoto](homelab/proxmox-pfsense-lab.md)** —
  diseño del laboratorio, plan de direccionamiento, VLANs, firewall, VPN y los
  problemas reales que aparecieron en el camino.

### Blue team
- **[CASE-001 — PowerShell ofuscado desde un macro malicioso](blue-team/CASE-001-encoded-powershell.md)** —
  informe de práctica de respuesta a incidentes: triage, análisis técnico,
  evidencia, IOCs, recomendaciones y mapeo a MITRE ATT&CK.

### Offensive
- _En preparación_ (HTB / CPTS).

## Stack

Proxmox VE · KVM · LXC · Docker · pfSense · VLANs · OpenVPN · Wireshark/tcpdump ·
Splunk / ELK · Snort · Windows Server / Active Directory (en despliegue)
