# 📖 PLAYBOOK SOC L1 — ESCENARIO FUERZA BRUTA
## Compromiso de Credencial por Ataque de Diccionario

| ID | Nivel | Severidad Base | MITRE ATT&CK | NIST 800-61 |
|---|---|---|---|---|
| PB-WEB-002-FB | SOC L1 | Alta | T1110.001 | Detección y Análisis |

---

## 1. Trigger

Se activa cuando se detecta un **patrón de múltiples intentos fallidos de autenticación (401)** contra el mismo usuario, seguido de un **acceso exitoso (200)** desde el mismo origen.

- Regla Wazuh: `2501` (syslog: User authentication failure) precediendo a `100405` (Successful login detected)
- El valor del campo usuario en el evento exitoso es una **credencial limpia** (sin sintaxis SQL ni caracteres de bypass) — esto diferencia este escenario del de SQL Injection.
- Si el evento exitoso contiene sintaxis SQL en el usuario → usar el playbook de SQL Injection. Si además hay cambio de técnica en la misma ventana → usar el playbook de Orquestador.

---

## 2. Contexto de la Alerta

| Campo | Dato a verificar |
|---|---|
| Hostname / agente | `Wazuh-windoServ-PC2-Lab` |
| IP origen | `192.168.3.163` |
| IP / endpoint objetivo | `/api/login` (192.168.3.10:3000) |
| Usuario atacado | `admin` |
| `rule.firedtimes` | 6 |
| Timestamp del evento exitoso | 21/07/2026 05:44:36.967Z |
| Código HTTP | 401 (intentos previos) → 200 (evento final) |
| SLA de triage | < 15 min desde la detección del acceso exitoso |

---

## 3. IOCs y Enriquecimiento

- **Usuario comprometido:** `admin` — credencial válida obtenida por diccionario, sin uso de payload SQLi.
- **Reputación externa:** No aplica — `192.168.3.163` es IP privada (RFC1918), sin visibilidad en VT/AbuseIPDB.
- **Contexto interno del origen:** `192.168.3.163` corresponde a la máquina atacante (Kali Linux) dentro de la topología del lab — no pertenece a ningún segmento autorizado de acceso a la Bank App.
- **Correlación temporal:** Verificar cuántos intentos fallidos (401) precedieron al éxito y en qué ventana de tiempo — un volumen alto en poco tiempo confirma comportamiento de bot, no error humano.
- **Contexto del activo objetivo:** `192.168.3.10:3000` — Bank App, activo de criticidad alta (simula aplicación financiera con datos sensibles).
- **Privilegios de la cuenta comprometida:** Confirmar si `admin` tiene privilegios elevados dentro de la aplicación — esto define el impacto real del compromiso.

---

## 4. Triage (Árbol de Decisión)

**1.** ¿Hubo múltiples intentos fallidos (401) contra el mismo usuario antes del éxito? → **NO:** revisar si el acceso fue con primer intento (posible credencial filtrada, no fuerza bruta) — evaluar como caso aparte. **SÍ:** seguir.

**2.** ¿`rule.firedtimes` muestra un volumen alto en una ventana corta (ej. 5+ disparos en pocos minutos)? → **NO:** posible error de tipeo legítimo del usuario real, bajar prioridad y confirmar con el dueño de la cuenta. **SÍ:** seguir.

**3.** ¿El usuario final exitoso es una credencial limpia (sin sintaxis SQL)? → **NO:** redirigir al playbook de SQL Injection. **SÍ:** seguir.

**4.** ¿El origen no pertenece a ningún segmento autorizado? → **NO:** posible FP (testing interno o usuario legítimo con varios intentos), verificar con el equipo. **SÍ:** **Verdadero Positivo — Fuerza Bruta confirmada.**

---

## 5. Clasificación Final

- [ ] Falso Positivo — justificación:
- [x] Verdadero Positivo — Severidad: **Alta**
- **Justificación técnica:** Múltiples intentos fallidos (401) contra la cuenta `admin` culminaron en un acceso exitoso (200). La regla llegó a 6 disparos, lo que descarta un evento aislado o un error de tipeo. Una fuerza bruta que termina en acceso sobre una cuenta con privilegios es exactamente el escenario de riesgo que se busca detectar.

---

## 6. Acciones del Analista L1

1. Documentar el patrón completo: cantidad de intentos fallidos, timestamp del éxito, `firedtimes`.
2. Extraer IOCs: IP origen, IP destino, usuario comprometido, timestamps.
3. Verificar si la cuenta `admin` tiene privilegios elevados dentro de la aplicación (solo consultar, no modificar).
4. Notificar al dueño del activo si la política lo requiere.
5. Recomendar acción de contención (bloqueo de IP + reseteo de credencial) en el ticket — **no ejecutarla** salvo automatización pre-aprobada.

---

## 7. Escalación

Escalar siempre que el resultado del triage sea Verdadero Positivo.

🚨 ALERTA ESCALADA: FUERZA BRUTA — COMPROMISO DE CREDENCIAL

-ID Incidente: INC-2026-0721-002

-IP origen → IP destino / endpoint: 192.168.3.163 → 192.168.3.10:3000 (/api/login)

-Usuario comprometido: admin

-Timestamp del acceso exitoso: 21/07/2026 05:44:36.967Z

-rule.firedtimes: 6

-Técnica MITRE: T1110.001

-Recomendación de contención: Bloqueo de IP 192.168.3.163 en firewall/WAF; reseteo inmediato de credencial "admin"; revisión de sesiones activas

-Evidencia adjunta: capturas de Discover (histograma + full_log del evento)


---

## 8. Evidencia a Documentar

- [ ] Histograma de eventos (Discover)
- [ ] Detalle del evento exitoso (`full_log`)
- [ ] Conteo de intentos fallidos previos (401) en la misma ventana
- [ ] Ticket de escalación

---

## 9. Lecciones Aprendidas

- **Gap detectado:** No hay bloqueo automático (account lockout) tras N intentos fallidos — el ataque pudo completarse sin fricción del sistema.
- **Mejora propuesta:** Implementar rate limiting (ej. máx. 5 intentos fallidos por usuario/IP en 5 minutos) con lockout temporal, y una regla Wazuh dedicada que dispare alerta crítica al superar ese umbral, antes de que ocurra el acceso exitoso.
