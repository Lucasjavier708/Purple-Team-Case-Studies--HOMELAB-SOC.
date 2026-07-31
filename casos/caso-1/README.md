

## Orchestrated Attack Framework: Multi-Stage Brute Force & SQLi with Real-Time SIEM Detection (Wazuh)

## Objetivo 

generar eventos de ataque realistas contra una aplicación web vulnerable para que el SIEM (Wazuh) pueda detectarlos, registrarlos y generar alertas. Básicamente es crear un escenario de incidente de seguridad controlado donde puedas estudiar cómo se ve un ataque desde la perspectiva del SOC. , se vana usar scripts automatizados en donde van a ser explicados paso a paso

## Arquitectura


<img width="808" height="608" alt="Caso Diagrama" src="https://github.com/user-attachments/assets/a322f426-e69c-405c-aad4-f4a41cf4d31a" />

<img width="828" height="622" alt="Diagrama sin título" src="https://github.com/user-attachments/assets/78d88b79-ce65-4dcd-ac8c-d17b6c527b0f" />

## Herramientas 

![Kali Linux](https://img.shields.io/badge/Kali%20Linux-Penetration%20Testing-557C94?style=flat-square&logo=linux&logoColor=white)
![Hydra](https://img.shields.io/badge/Hydra-Brute%20Force-FF6B6B?style=flat-square&logo=python&logoColor=white)
![Burpsuit](https://img.shields.io/badge/Burpsuit-Web%20Proxy-FF9900?style=flat-square&logo=firefox&logoColor=white)
![Nmap](https://img.shields.io/badge/Nmap-Port%20Scanner-4169E1?style=flat-square&logo=network&logoColor=white)


![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Bank%20App-339933?style=flat-square&logo=node.js&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2019-CC2927?style=flat-square&logo=microsoft-sql-server&logoColor=white)


![Wazuh](https://img.shields.io/badge/Wazuh-4.x%20SIEM-0078D4?style=flat-square&logo=security&logoColor=white)






 ## Paso a Paso 


APP

Este es el servidor objetivo. La Bank App corre sin protecciones (vulnerable intencionalmente) para que podamos simular ataques reales: brute force contra el endpoint /api/login y SQL injection en los campos de usuario y contraseña.

<img width="1024" height="768" alt="Captura de pantalla (1)" src="https://github.com/user-attachments/assets/04153a85-29da-4ff3-a77b-666835ba8450" />

<img width="1024" height="768" alt="Captura de pantalla (5)" src="https://github.com/user-attachments/assets/8d4ad243-ec3e-47e6-94d5-fbc6a9ebfe26" />


Nmap

Antes de atacar, necesitamos saber qué puertos están abiertos en el Windows Server y qué servicios corren. Este escaneo es la fase de reconocimiento (footprinting) donde identificamos la superficie de ataque disponible.

<img width="1024" height="768" alt="nmap1" src="https://github.com/user-attachments/assets/656bac1d-1819-4b5d-bd79-1a806503f6ec" />



BurpSuit

Configuramos Burpsuit en modo proxy listening en 127.0.0.1:8080, activamos Intercept, y capturamos las peticiones legítimas a la Bank App.Al activar el módulo Intercept, capturamos cada petición POST al endpoint /api/login, permitiéndonos examinar los parámetros de usuario y contraseña, los headers HTTP, y las respuestas del servidor. Este análisis manual es crucial antes de automatizar ataques, ya que identificamos patrones de error, tiempos de respuesta, y validaciones que luego configuraremos en los scripts de Hydra y SQLi.



<img width="1024" height="768" alt="Screenshot_2026-07-21_10_06_11" src="https://github.com/user-attachments/assets/47f00d77-82c5-4a69-9136-e85a1bb110f1" />

<img width="1024" height="768" alt="Screenshot_2026-07-21_10_06_57" src="https://github.com/user-attachments/assets/55021553-bda4-400b-a6fe-3544b3ad55ed" />

<img width="1024" height="768" alt="Screenshot_2026-07-21_10_07_07" src="https://github.com/user-attachments/assets/7e9f346d-24ab-45fe-b4ca-0185e1b9a032" />

<img width="1024" height="768" alt="Screenshot_2026-07-21_10_07_32" src="https://github.com/user-attachments/assets/13d3cb61-3594-47ed-9574-4082aaca2d7b" />





scripts automatizados

En /redlab-attacks se encuentran los tres pilares de la automatización: orchestrator.sh que coordina todo el ataque, attack_brute_force.sh que ejecuta Hydra contra /api/login, y attack_sqli.sh que prueba inyecciones SQL. Los diccionarios y wordlists (midiccionaries.txt) alimentan los ataques, mientras que los archivos de output registran cada intento exitoso y fallido.

<img width="1024" height="768" alt="redlab" src="https://github.com/user-attachments/assets/b9d2a711-bb33-47e2-aecc-54c0ae003d6b" />


**El orchestrator.sh** es el script maestro que coordina todos los ataques. Al ejecutarse, inicia automáticamente el ataque de brute force contra /api/login usando Hydra, espera a que termine, luego lanza el ataque de SQL injection. Cada fase se marca con [+] al comenzar y [✓] al completarse. Al finalizar ambos ataques, genera un resumen de sesión con todos los eventos registrados.

<img width="1024" height="768" alt="Orquestador 1" src="https://github.com/user-attachments/assets/54dc44d1-d99c-4271-b6dd-0f458fcc0c85" />

<img width="1024" height="768" alt="Orquestador 2" src="https://github.com/user-attachments/assets/4b09f48c-878d-4534-a35c-03db4dc980c1" />


**Attack_brute_force***.sh automatiza el ataque que hicimos manualmente en Burpsuit. Hydra toma la estructura POST que vimos (/api/login:username=USER&password=PASS) y la replica miles de veces con diferentes contraseñas del diccionario. El comando hydra -l admin -P midiccionary.txt -s 3000 192.168.3.10 http-post-form "/api/login:username=USER&password=PASS:i:F=Usuario o contraseña inválido" automatiza exactamente lo que interceptamos en Burpsuit: envíos POST al /api/login con parámetro username y password. La cadena i:F=Usuario o contraseña inválido es la respuesta de error que vimos en Burpsuit, permitiendo a Hydra diferenciar intentos fallidos de exitosos. Cuando Hydra encuentra una contraseña válida, Wazuh registra ese acceso exitoso como evento de seguridad.

<img width="1020" height="710" alt="fuezabruta" src="https://github.com/user-attachments/assets/e7e93869-3fc5-4ff0-a6ae-d6a70b311bde" />

<img width="1007" height="592" alt="Captura de pantalla_2026-07-21_11-07-22" src="https://github.com/user-attachments/assets/75bef765-d04c-4e82-b604-1ce547c9abca" />



## Hydra Code- 

hydra -l admin -P midiccionary.txt -s 3000 192.168.3.10 http-post-form "/api/login:username=USER&password=PASS:i:F=Usuario o contraseña inválido"

| Parámetro | Descripción |
|-----------|-------------|
| `hydra` | Herramienta de fuerza bruta para ataques de login |
| `-l admin` | Define usuario específico a atacar (login fijo) |
| `-P midiccionary.txt` | Carga diccionario de contraseñas a probar |
| `-s 3000` | Puerto específico donde corre la Bank App |
| `192.168.3.10` | IP del Windows Server objetivo |
| `http-post-form` | Módulo de ataque vía formulario POST |
| `/api/login` | Endpoint/ruta del servidor a atacar |
| `username=USER&password=PASS` | Estructura POST (USER y PASS se reemplazan en cada intento) |
| `i:F=` | Flag: `i` (case-insensitive) + `F` (fail string) |
| `Usuario o contraseña inválido` | Mensaje de error exacto que detecta intentos fallidos |




**Attack_sqli.sh** automatiza inyecciones SQL contra /api/login. Envía payloads maliciosos como admin' or '1'='1 -- en los campos de usuario y contraseña para bypassear la validación. El servidor intenta ejecutar estas consultas y responde con HTTP 200 si la inyección funciona. Wazuh captura cada intento como evento de ataque SQLi.


0<img width="1024" height="768" alt="Sqli" src="https://github.com/user-attachments/assets/f0e1400a-fe3a-4ef7-a430-3be0a289f059" />


------------------------------------- 
<img width="2554" height="974" alt="Captura de pantalla 2026-07-20 210513" src="https://github.com/user-attachments/assets/ea22a075-1d0d-4de7-a5a3-d6914dcb3958" />


 ## Monitoreo

 Aca es la ventana hacia lo que sucedió en el servidor objetivo. A través del dashboard de Wazuh, visualizamos la línea temporal completa del incidente: los intentos fallidos del brute force, los payloads SQLi variantes, y finalmente los accesos exitosos. Como analistas, extraemos IOCs (Indicadores de Compromiso), identificamos patrones anómalos, y documentamos la cadena de eventos para el reporte de seguridad.


<img width="2544" height="907" alt="Captura de pantalla 2026-07-21 172021" src="https://github.com/user-attachments/assets/9c5bd579-6e05-403a-9d2e-854f082f3c38" />



### ORQUESTADOR 

El script orquestador generó eventos híbridos: intentos fallidos de brute force seguidos de inyecciones SQL variantes. Los IOCs extraídos son: IP 192.168.3.163, payloads admin' or 1=1 --, timestamps 05:42:22 a 05:42:30 UTC, y HTTP 200 exitoso con Rule 100405 disparada. Evidencia: coordinación inteligente entre técnicas.

<img width="2557" height="903" alt="Evento Orq" src="https://github.com/user-attachments/assets/bf87d84b-5f03-4d5a-af9a-5b3bd7b379a4" />

<img width="2557" height="902" alt="LOG 1 Orquestador" src="https://github.com/user-attachments/assets/69ef60d6-1085-4c3a-b8cb-984deda89fc8" />




### FUERZA BRUTA 

El script de brute force generó múltiples intentos fallidos (401) contra /api/login entre 05:44:33 y 05:44:36 UTC. IOCs documentados: IP 192.168.3.163, usuario "admin" consistente, 9+ intentos fallidos, culminando en HTTP 200 exitoso. Evidencia: patrón temporal claro de bot atacando credencial específica.

<img width="2552" height="902" alt="Fuerza bruta evento" src="https://github.com/user-attachments/assets/060bb68d-780b-464b-9e66-8e2f666f99ed" />

<img width="2557" height="903" alt="Captura de pantalla 2026-07-21 111627" src="https://github.com/user-attachments/assets/5b3f00e0-279a-4e66-a8e1-598a9b4c9aab" />



### SQL INJECTION 

El script SQLi inyectó payloads variantes: admin' --, ' or '1'='1', admin' or 1=1 --. Timestamps 05:45:53 a 05:45:59 UTC. IOCs: IP 192.168.3.163, payloads exactos, HTTP 200 exitoso con misma Rule 100405. Evidencia: cambio de técnica de ataque después de brute force, bypassing autenticación.

<img width="2557" height="897" alt="evento sqli" src="https://github.com/user-attachments/assets/7873b163-dc9e-4d54-bea1-508a56a3b84e" />

<img width="2554" height="906" alt="Captura de pantalla 2026-07-21 121441" src="https://github.com/user-attachments/assets/df1514f2-59ba-4e25-9cb3-4e8bcf876fd8" />



# 3. PATRONES Y ANOMALÍAS

### CAPTURA 1: Timeline de Eventos (Franja Horaria)
La vista temporal muestra eventos concentrados entre 10:10-10:14 UTC. Los picos verdes indican densidad anómala: 9 intentos en 30 segundos es comportamiento de bot, no usuario legítimo. Primera indicación de brute force automatizado.


<img width="2554" height="498" alt="Captura de pantalla 2026-07-21 182441" src="https://github.com/user-attachments/assets/05ce1168-87dc-4573-ac02-5d42af226773" />

<img width="2541" height="781" alt="Captura de pantalla 2026-07-21 182524" src="https://github.com/user-attachments/assets/6395df00-8422-446f-8bfb-0ffa40a1d369" />


### CAPTURA 2: Búsqueda de Payloads SQLI
Al filtrar por inyecciones SQL, vemos variantes de payloads (`admin' --`, `admin' or '1'='1'`, etc.). La progresión fallido → exitoso demuestra que el atacante **iteró y refinó** la técnica hasta encontrar una funcional. Ataque inteligente, no accidental.


<img width="2550" height="806" alt="payload sqli" src="https://github.com/user-attachments/assets/56815818-beff-4d70-b2f8-164590c4c9a8" />


<img width="2548" height="907" alt="payload sqli jason" src="https://github.com/user-attachments/assets/d59c41b8-826c-4805-8619-21d1008fa26b" />


### CAPTURA 3: Detalles JSON del Evento
El drill-down muestra: timestamp exacto, IP origen, payload exacto, HTTP 200 (exitoso), Rule 100405 disparada. Confirma acceso exitoso real y documenta evidencia forense completa para análisis.



---

## 4. CORRELACIÓN DE EVENTOS

Durante el análisis se identificaron **3 accesos exitosos (HTTP 200)** desde la IP atacante 192.168.3.163 en un intervalo de 3 minutos. El primer acceso (05:42:29Z) utilizó el payload SQLi `admin' or 1=1 --`, confirmando que la inyección SQL es efectiva contra la aplicación. El segundo acceso (05:44:36Z) empleó credencial limpia `admin`, evidenciando que el brute force tuvo éxito en obtener la contraseña válida. El tercer acceso (05:45:58Z) reutilizó el mismo payload SQLi, demostrando que el atacante dispone de múltiples vectores de explotación funcionales. La relevancia crítica radica en que el atacante no solo accedió una vez: accedió 3 veces usando técnicas diferentes, lo que indica reconocimiento previo de la vulnerabilidad y reconfirmación de acceso post-infiltración.

---

## 5. MITRE ATT&CK - MAPEO DEL INCIDENTE

### Reconocimiento:
- **T1046 (Network Service Discovery)** - Nmap scan identificó puertos abiertos (3000, 1433)

### Acceso Inicial:
- **T1190 (Exploit Public-Facing Application)** - SQLi contra endpoint `/api/login`
- **T1110.001 (Brute Force - Credential Guessing)** - Hydra atacó username "admin" con diccionario

### Ejecución/Persistencia:
- **T1021.004 (Remote Service Session Hijacking)** - Post-acceso exitoso, el atacante mantiene sesión HTTP 200

**Por script:**
- Orquestador: T1190 (SQLi exitosa)
- Brute Force: T1110.001 (Credencial adivinada)
- SQLi: T1190 (Inyección SQL variantes)

---

## 6. MITIGACIÓN

### Inmediatas (24 horas):

1. **Bloqueo de IP:** Implementar regla en firewall/WAF bloqueando 192.168.3.163 en todos los puertos
2. **Cambio de credencial:** Resetear credencial "admin" inmediatamente; forzar cambio de contraseña en todos los usuarios
3. **Revoke de sesiones:** Terminar todas las sesiones activas de la cuenta admin desde el último acceso legítimo conocido

### Corto plazo (1 semana):

4. **Input Validation:** Implementar sanitización en `/api/login` - validar caracteres especiales (comillas, dashes, comentarios SQL) en campos username/password
5. **Parameterized Queries:** Refactorizar queries SQL para usar prepared statements en lugar de concatenación de strings
6. **Rate Limiting:** Configurar máximo 5 intentos fallidos por usuario/IP en 5 minutos, luego account lockout por 15 minutos
7. **MFA:** Implementar autenticación multifactor (TOTP/SMS) en cuenta admin

### Largo plazo (30 días):

8. **WAF Rules:** Crear reglas en Wazuh/WAF detectando payloads SQLi comunes (`or '1'='1`, `--`, `/**/`) y bloquearlos automáticamente
9. **Security Testing:** Realizar penetration testing formal contra la Bank App para identificar otras vulnerabilidades
10. **Logging mejorado:** Aumentar verbosidad de logs para capturar User-Agent, session IDs, y source IP en todos los intentos  
