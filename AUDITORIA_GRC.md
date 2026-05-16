# INFORME DE AUDITORÍA GRC — ANÁLISIS CRUZADO (v3)

**Documento auditado:** Tarea_GRC_Semana5_TechGuard (versión actualizada 2026-05-15)  
**Referencia oficial:** tarea.pdf  
**Evidencias técnicas:** Repositorio UMG-GRC-SOC-Lab  
**Fecha de auditoría:** 2026-05-15  
**Contexto:** Equipo de 4 aceptado | Runtime verificado | IP desestimada

---

## SECCIÓN 1 — INCONSISTENCIAS PERSISTENTES

### 1.1 ISO 27001:2022 — Corregido en Documento, Incorrecto en Artefactos de Runtime

El documento v3 ahora referencia correctamente controles ISO 27001:2022 en el texto. Sin embargo, el archivo `siem_data/alertas_siem.json` — generado por `siem_simulado.py` y verificable por cualquier evaluador — **continúa con nomenclatura 2013**:

```json
// alertas_siem.json — sin actualizar
{ "rule_id": "GRC-1001", "iso": "A.9.4.2"  }  // debe ser A.8.5
{ "rule_id": "GRC-1002", "iso": "A.12.4.1" }  // debe ser A.8.15
{ "rule_id": "GRC-1003", "iso": "A.12.4.1" }  // debe ser A.8.15
{ "rule_id": "GRC-1004", "iso": "A.13.1.1" }  // debe ser A.8.20
{ "rule_id": "GRC-1099", "iso": "A.16.1.5" }  // debe ser A.5.26
```

El documento dice 2022; el laboratorio ejecutable produce 2013. La corrección debe hacerse en [scripts/siem_simulado.py](scripts/siem_simulado.py) y regenerar el JSON.

### 1.2 MTTR — JSON de Runtime Contradice Documento

El documento (sección 5.3 y el ejemplo de `/api/status` en sección 4.6) declara **MTTR promedio = 15.5 minutos**. El archivo `siem_data/dashboard_metricas.json` exporta:

```json
"MTTR (Mean Time To Respond)": { "valor": 14, "meta": 15, "estado": "OK" }
```

El JSON real del repositorio reporta **14**, no 15.5. El evaluador que ejecute `cat siem_data/dashboard_metricas.json` obtendrá un valor diferente al del documento. La corrección requiere actualizar `dashboard_grc.py` para calcular el promedio de los 4 incidentes resueltos: (14+8+22+18)/4 = 15.5, y regenerar el JSON.

### 1.3 Log Consolidado — Duplicación de Eventos no Resuelta

`logs/consolidated_20260515.log` contiene 16 líneas con los mismos eventos registrados dos veces en formatos distintos:
- Líneas 1-7: formato `auth.log` (`FAILED`/`SUCCESS`, timestamps `HH:MM:SS`)
- Líneas 8-16: formato `soc_logs.txt` (`LOGIN_FAIL`/`LOGIN_SUCCESS`, timestamps `HH:MM`)

La concatenación `cat auth.log soc_logs.txt` fusiona dos fuentes que documentan el mismo ataque del 2026-05-01. El conteo real de fallos es 4; el log consolidado contiene 7 entradas de fallo. No fue abordado en la actualización del documento.

### 1.4 Cadena de Custodia SHA256 — Hash no Almacenado

El repositorio no contiene ningún archivo `.sha256` o similar que preserve el hash del log forense. `siem_data/` y `logs/` fueron verificados — solo existe el archivo origen `forense_ir_0502_20260515_1847.log`. El documento (secciones 3.5 y 6.1) afirma que la cadena de custodia con SHA256 es "un diferenciador crítico para la admisibilidad forense de evidencia digital", pero el hash no fue capturado ni almacenado.

Ejecutar y almacenar:
```bash
sha256sum logs/forense_ir_0502_20260515_1847.log > siem_data/cadena_custodia_ir0502.sha256
```

---

## SECCIÓN 2 — CORRECCIONES PENDIENTES (orden de impacto)

| Prioridad | Corrección | Archivo a modificar | Acción |
|---|---|---|---|
| 1 | Portada "Tarea 3" → "Tarea 5" | `Tarea_GRC_Semana5_TechGuard.docx` | Cambiar número en carátula |
| 2 | ISO 2022 en código | `scripts/siem_simulado.py` | Actualizar 5 valores `"iso"` + regenerar `alertas_siem.json` |
| 3 | MTTR promedio en JSON | `scripts/dashboard_grc.py` | Calcular (14+8+22+18)/4 + regenerar `dashboard_metricas.json` |
| 4 | SHA256 cadena de custodia | Ejecutar comando | Generar `siem_data/cadena_custodia_ir0502.sha256` |

---

## TABLA DE ESTADO FINAL (v3)

| # | Hallazgo | Severidad | Estado |
|---|---|---|---|
| 1 | Laboratorio funcional sin evidencia | CRÍTICA | ✅ SUBSANADO (v2) |
| 2 | PDCA — Do/Act ausentes | MEDIA | ✅ SUBSANADO (v3) |
| 3 | MTTD hardcodeado sin disclaimer | ALTA | ✅ SUBSANADO (v3) |
| 4 | IR-0503/0505 sin justificación | ALTA | ✅ SUBSANADO (v3) |
| 5 | ISO 27001:2022 en documento | ALTA | ✅ SUBSANADO (v3) |
| 6 | MTTR ejemplo `/api/status` | MEDIA | ✅ SUBSANADO (v3) |
| 7 | ISO 2013 en `alertas_siem.json` | ALTA | ❌ PERSISTE (código) |
| 8 | MTTR=14 en `dashboard_metricas.json` | ALTA | ❌ PERSISTE (código) |
| 9 | Log consolidado — eventos duplicados | MEDIA | ❌ PERSISTE |
| 10 | SHA256 no almacenado | MEDIA | ❌ PERSISTE |
| 11 | Equipo de 4 vs. 5 | — | ✅ DESESTIMADO |
| 12 | Discrepancia IP docker-compose | — | ✅ DESESTIMADO |

---

## DICTAMEN v3

**Progreso significativo:** 6 de 9 hallazgos activos de v2 fueron subsanados. El documento tiene solidez estructural y el laboratorio está correctamente ejecutado y evidenciado.

**Bloqueante de entrega:** El ítem 7 (portada "Tarea 3") es un error visible en la carátula — el primer dato que lee el evaluador. Los ítems 8 y 9 son verificables ejecutando el laboratorio y producen salidas que contradicen el texto.

**Orden mínimo antes de entregar:** Corregir portada → actualizar `siem_simulado.py` y `dashboard_grc.py` → regenerar JSONs → commit.
