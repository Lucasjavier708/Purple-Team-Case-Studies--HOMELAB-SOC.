# 📖 PLAYBOOK SOC L1 — ESCENARIO ORQUESTADOR
## Ataque Multi-Vector: Fuerza Bruta + SQL Injection Correlacionados

| ID | Nivel | Severidad Base | MITRE ATT&CK | NIST 800-61 |
|---|---|---|---|---|
| PB-WEB-001-ORQ | SOC L1 | Alta | T1110.001 + T1190 | Detección y Análisis |

---

## 1. Trigger

Se activa cuando se detectan **2 o más técnicas distintas de acceso** (fuerza bruta y SQLi) desde el **mismo origen**, en una ventana **≤ 5 minutos**.

- Regla Wazuh: `100405` (Successful login) + `2501` (fallos previos de auth)
- No usar este playbook si solo hay un vector detectado → usar el playbook de Fuerza Bruta o de SQLi según corresponda.

---

## 2. Contexto de la Alerta

| Campo | Dato a verificar |
|---|---|
| Hostname / agente | `Wazuh-windoServ-PC2-Lab` |
| IP origen | `192.168.3.163` |
| IP / endpoint objetivo | `/api/login` (192.168.3.10:3000) |
| `rule.firedtimes` por evento | 5 (fase SQLi inicial) → 6 (fase fuerza bruta) → 10 (fase SQLi final) |
| Ventana temporal del patrón | 21/07/2026 05:42:29Z — 05:45:58Z (≈ 3 minutos y medio) |
| Código HTTP de cada evento | 200 (éxito) en los tres eventos |
| SLA de triage | < 15 min desde detección del patrón |

---

## 3. IOCs y Enriquecimiento

- **Usuario(s)/payload(s) usados en cada evento:**
  1. `05:42:29Z` → `admin' or 1=1 --` (SQLi)
  2. `05:44:36Z` → `admin` (credencial limpia, fuerza bruta exitosa)
  3. `05:45:58Z` → `admin' or 1=1 --` (SQLi, reutilizado)
- **Reputación externa:** No aplica — `192.168.3.163` es IP privada (RFC1918), sin visibilidad en VT/AbuseIPDB.
- **Contexto interno del origen:** `192.168.3.163` corresponde a la máquina atacante (Kali Linux) dentro de la topología del lab — no pertenece a ningún segmento autorizado de acceso a la Bank App.
- **Correlación temporal:** Se confirmaron 3 accesos exitosos (HTTP 200) desde el mismo origen en 3m29s: SQLi → credencial válida → SQLi nuevamente. El patrón descarta coincidencia — evidencia de cambio de táctica automatizado.
- **Contexto del activo objetivo:** `192.168.3.10:3000` — Bank App, activo de criticidad alta (simula aplicación financiera con datos sensibles).

---

## 4. Triage (Árbol de Decisión)

**1.** ¿2+ técnicas distintas, mismo origen, ≤5 min? → **NO:** redirigir a playbook específico (FB o SQLi). **SÍ:** seguir.

**2.** ¿`firedtimes` progresa en cada fase, mostrando automatización? → **NO:** posible evento manual aislado, bajar prioridad. **SÍ:** seguir.

**3.** ¿El cambio de técnica ocurre sin pausa significativa? → **NO:** evaluar como eventos separados. **SÍ:** seguir.

**4.** ¿El origen no pertenece a ningún segmento autorizado? → **NO:** posible FP (testing interno), verificar con el equipo. **SÍ:** **Verdadero Positivo — Orquestador confirmado.**

---

## 5. Clasificación Final

- [ ] Falso Positivo — justificación:
- [x] Verdadero Positivo — Severidad: **Alta**
- **Justificación técnica:** El HTTP 200 inicial no corresponde a un login real — el campo usuario contiene el payload `admin' or 1=1 --`, típico de bypass por SQLi. El timing es la clave: apenas terminan los intentos de fuerza bruta contra "admin", arrancan las inyecciones SQLi casi de inmediato, sin pausa — consistente con un orquestador que cambió de táctica automáticamente al no lograr entrar primero. El origen `192.168.3.163` no pertenece a ningún segmento válido de la red.

---

## 6. Acciones del Analista L1

1. Documentar la correlación completa (los eventos relacionados, no solo uno aislado)
2. Extraer IOCs: IP origen, IP destino, usuarios/payloads, timestamps
3. Verificar si la cuenta afectada tiene privilegios elevados (solo consultar, no modificar)
4. Notificar al dueño del activo si la política lo requiere
5. Recomendar acción de contención (bloqueo de IP) en el ticket — **no ejecutarla** salvo automatización pre-aprobada

---

## 7. Escalación

Escalar siempre que el resultado del triage sea Verdadero Positivo.

---

## 8. Evidencia a Documentar

- [ ] Histograma de eventos (Discover)
- [ ] Detalle de cada evento (`full_log`)
- [ ] Tabla de correlación de los eventos
- [ ] Ticket de escalación

---

## 9. Lecciones Aprendidas

- **Gap detectado:** [ej. Wazuh no distingue el vector usado dentro del mismo rule.id — requiere inspección manual del campo usuario]
- **Mejora propuesta:** [ej. regla de correlación dedicada para el patrón fallo repetido + éxito con sintaxis SQL]
