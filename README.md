
#   HOMELAB/SOC  *Analisis de Casos de Estudio*

HOMELAB/SOC  es un proyecto de ciberseguridad basado en la construcción y documentación de un laboratorio práctico (HOMELAB), diseñado para simular el funcionamiento de un Centro de Operaciones de Seguridad (SOC).

El proyecto reúne diferentes casos de estudio que recrean escenarios reales de ciberseguridad, permitiendo observar cómo se ejecutan, detectan, analizan, investigan y mitigan distintos eventos e incidentes de seguridad dentro de un entorno controlado.

Cada caso documenta el proceso completo, desde la ejecución de un ataque y la utilización de diferentes tecnologías hasta su detección, análisis y respuesta desde la perspectiva de un Operador SOC Nivel 1. De esta manera, esta infraestructura HOMELAB/SOC funciona tanto como un entorno de práctica técnica como un repositorio de documentación orientado al aprendizaje y al desarrollo de habilidades en ciberseguridad.

# Objetivo 

Demostrar conocimientos y habilidades prácticas en ciberseguridad mediante la construcción y operación de una infraestructura HOMELAB/SOC, documentando casos de estudio que simulan escenarios de seguridad informática.

Se busca evidencias 

-  Diseño e implementación de una infraestructura HOMELAB/SOC
-  Simulación y documentación de escenarios de ataque y defensa.
-  Detección, análisis, investigación y respuesta ante eventos e incidentes de seguridad.
-  Utilización de herramientas y tecnologías empleadas en entornos profesionales
-  Aplicación de metodologías y buenas prácticas de Blue Team y Centros de Operaciones de Seguridad (SOC).
- Desarrollo de estrategias de mitigación, prevención y mejora continua frente a amenazas de ciberseguridad

# Arquitectura 


























# Stack Stenico
 
### 🖥️ Sistemas Operativos
![Windows Server](https://img.shields.io/badge/Windows_Server-2019-0078D4?style=flat-square&logo=windows)
![Ubuntu](https://img.shields.io/badge/Ubuntu-Server-E95420?style=flat-square&logo=ubuntu)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-2023-557C94?style=flat-square&logo=kalilinux)
![Windows 11](https://img.shields.io/badge/Windows-11-0078D4?style=flat-square&logo=windows)

### 🛡️ SIEM & Monitoreo
![Wazuh](https://img.shields.io/badge/Wazuh-Manager-005EB8?style=flat-square&logo=wazuh)
![Sysmon](https://img.shields.io/badge/Sysmon-Monitoring-00A4EF?style=flat-square)

### 🌐 Redes
![TCP/IP](https://img.shields.io/badge/TCP%2FIP-Networking-FF6B6B?style=flat-square)
![DNS](https://img.shields.io/badge/DNS-Resolution-4285F4?style=flat-square)
![DHCP](https://img.shields.io/badge/DHCP-Protocol-00AA00?style=flat-square)
![VPN](https://img.shields.io/badge/VPN-Secure-FF00FF?style=flat-square)
![Firewall](https://img.shields.io/badge/Firewall-Protection-FFA500?style=flat-square)
![IDS/IPS](https://img.shields.io/badge/IDS%2FIPS-Detection-DC143C?style=flat-square)

### 🔍 Análisis de Seguridad 
![Nmap](https://img.shields.io/badge/Nmap-Scanning-0D5B05?style=flat-square)
![VirusTotal](https://img.shields.io/badge/VirusTotal-Analysis-390EFF?style=flat-square)

### ⚔️ Pentesting
![Metasploit](https://img.shields.io/badge/Metasploit-Framework-2D2D2D?style=flat-square&logo=metasploit)
![Hydra](https://img.shields.io/badge/Hydra-Brute_Force-FF6B6B?style=flat-square)
![Burp Suite](https://img.shields.io/badge/Burp_Suite-Testing-FF6633?style=flat-square)

### 💻 Desarrollo
![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python)
![Bash](https://img.shields.io/badge/Bash-5.0-4EAA25?style=flat-square&logo=gnubash)
![PowerShell](https://img.shields.io/badge/PowerShell-7.0-5391FE?style=flat-square&logo=powershell)
![Node.js](https://img.shields.io/badge/Node.js-18.x-339933?style=flat-square&logo=node.js)

### ☁️ Infraestructura
![Docker](https://img.shields.io/badge/Docker-Latest-2496ED?style=flat-square&logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-K8s-326CE5?style=flat-square&logo=kubernetes)
![VirtualBox](https://img.shields.io/badge/VirtualBox-Hypervisor-183A61?style=flat-square&logo=virtualbox)

### 📚 Frameworks & Buenas Prácticas
![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-Framework-EE0000?style=flat-square)
![ISO 27001](https://img.shields.io/badge/ISO_27001-Compliance-0066CC?style=flat-square)



# 📋 Casos de Estudio

| Caso | Descripción |
|------|------------|
| [🎯 Orchestrated Attack Framework: Multi-Stage Brute Force & SQLi with Real-Time SIEM Detection (Wazuh)](./casos/caso-1) | Ataque orquestado |
| [🌐 DNS Tunneling](./casos/dns-tunneling) | Túnel DNS exfiltrando datos confidenciales. Detección con Machine Learning (Isolation Forest). Análisis de patrones anómalos de tráfico DNS y correlación de eventos. |
| [🔄 Lateral Movement](./casos/lateral-movement) | Movimiento lateral en red Windows. Escalada de privilegios con técnicas de post-explotación. Monitoreo con Sysmon y Wazuh SIEM. |
| [📊 Log Analysis & Incident Response](./casos/log-analysis) | Análisis profundo de eventos de seguridad. Correlación de alertas multi-fuente. Respuesta a incidentes y investigación forense. |


# Resultados 



# Conclusiones 
