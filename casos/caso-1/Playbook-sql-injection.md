# 📖 PLAYBOOK SOC L1 — ESCENARIO SQL INJECTION
## Bypass de Autenticación por Inyección SQL

| ID | Nivel | Severidad Base | MITRE ATT&CK | NIST 800-61 |
|---|---|---|---|---|
| PB-WEB-003-SQLI | SOC L1 | Alta | T1190 | Detección y Análisis |

---

## 1. Trigger

Se activa cuando se detecta un **acceso exitoso (200)** en el que el campo usuario contiene **sintaxis SQL de bypass** (comillas, operadores lógicos, comentarios SQL), sin necesidad de intentos fallidos previos con credenciales limpias.

- Regla Wazuh: `100405` (Successful login detected), con el campo usuario mostrando patrones como `' or '1'='1`, `admin' --`, `admin' or 1=1 --`.
- Si además se detectan intentos fallidos previos con credencial limpia (fuerza bruta) inmediatamente antes → usar el playbook de Orquestador, no este.

---

## 2. Contexto de la Alerta

| Campo | Dato a verificar |
|---|---|
| Hostname / agente | `Wazuh-windoServ-PC2-Lab` |
| IP origen | `192.168.3.163` |
| IP / endpoint objetivo | `/api/login` (192.168.3.10:3000) |
| Payload final exitoso | `admin' or 1=1 --` |
| `rule.firedtimes` | 10 |
| Timestamp del evento exitoso | 21/07/2026 05:45:58.890Z |
| Código HTTP | 200 |
| SLA de triage | < 15 min desde la detección del acceso exitoso |

---

## 3. IOCs y Enriquecimiento

- **Payloads observados (variantes iteradas):** `admin' --`, `' or '1'='1'`, `admin' or 1=1 --`.
- **Reputación externa:** No aplica — `192.168.3.163` es IP privada (RFC1918), sin visibilidad en VT/AbuseIPDB.
- **Contexto interno del origen:** `192.168.3.163` corresponde a la máquina atacante (Kali Linux) dentro de la topología del lab — no pertenece a ningún segmento autorizado de acceso a la Bank App.
- **Correlación temporal:** El alto `firedtimes` (10) indica iteración de múltiples variantes del payload hasta encontrar una funcional — no fue un intento único ni accidental.
- **Contexto del activo objetivo:** `192.168.3.10:3000` — Bank App, activo de criticidad alta (simula aplicación financiera con datos sensibles).
- **Alcance de la vulnerabilidad:** Pendiente confirmar si la inyección funciona en otros endpoints además de `/api/login`.

---

## 4. Triage (Árbol de Decisión)

**1.** ¿El campo usuario del evento exitoso contiene sintaxis SQL (comillas, `or 1=1`, `--`, comentarios)? → **NO:** este no es un caso de SQLi — evaluar como fuerza bruta u otro vector. **SÍ:** seguir.

**2.** ¿`rule.firedtimes` muestra múltiples disparos, consistente con iteración de variantes del payload? → **NO:** podría ser un intento manual único y aislado — evaluar impacto igual, pero bajar prioridad relativa. **SÍ:** seguir.

**3.** ¿Hubo intentos de fuerza bruta con credencial limpia inmediatamente antes, en la misma ventana? → **SÍ:** redirigir al playbook de Orquestador. **NO:** seguir.

**4.** ¿El origen no pertenece a ningún segmento autorizado? → **NO:** posible FP (testing de seguridad interno autorizado), verificar con el equipo. **SÍ:** **Verdadero Positivo — SQL Injection confirmada.**

---

## 5. Clasificación Final

- [ ] Falso Positivo — justificación:
- [x] Verdadero Positivo — Severidad: **Alta**
- **Justificación técnica:** El payload `admin' or 1=1 --` manipula la lógica de la consulta SQL para que la condición de autenticación siempre se evalúe como verdadera, sin necesidad de conocer una contraseña real. Los 10 disparos de la regla confirman iteración de variantes hasta lograr éxito, no un evento accidental.

---

## 6. Acciones del Analista L1

1. Documentar el payload exacto y todas las variantes observadas en la ventana del evento.
2. Extraer IOCs: IP origen, IP destino, payload final exitoso, timestamps.
3. Verificar si la inyección compromete otros endpoints además de `/api/login` (solo consultar, no explotar activamente).
4. Notificar al dueño del activo si la política lo requiere.
5. Recomendar acción de contención (bloqueo de IP + revisión de sanitización de inputs) en el ticket — **no ejecutarla** salvo automatización pre-aprobada.

---

## 7. Escalación

Escalar siempre que el resultado del triage sea Verdadero Positivo.

🚨 ALERTA ESCALADA: SQL INJECTION — BYPASS DE AUTENTICACIÓN

-ID Incidente: INC-2026-0721-003

-IP origen → IP destino / endpoint: 192.168.3.163 → 192.168.3.10:3000 (/api/login)
-Payload exitoso: admin' or 1=1 --

-Timestamp del acceso exitoso: 21/07/2026 05:45:58.890Z

-rule.firedtimes: 10

-Técnica MITRE: T1190

-Recomendación de contención: Bloqueo de IP 192.168.3.163 en firewall/WAF; revisión urgente de sanitización de inputs / uso de prepared statements en /api/login

-Evidencia adjunta: capturas de Discover (histograma + full_log del evento + variantes de payload)


---

## 8. Evidencia a Documentar

- [ ] Histograma de eventos (Discover)
- [ ] Detalle del evento exitoso (`full_log`)
- [ ] Listado de variantes de payload observadas
- [ ] Ticket de escalación

---

## 9. Lecciones Aprendidas

- **Gap detectado:** La aplicación no sanitiza el campo usuario ni utiliza consultas parametrizadas — permite que un payload de bypass se ejecute directamente contra la base de datos.
- **Mejora propuesta:** Implementar prepared statements en el backend, sumar reglas de WAF que detecten patrones SQLi comunes (`or '1'='1`, `--`, `/**/`) y bloqueen la petición antes de que llegue a la aplicación.
