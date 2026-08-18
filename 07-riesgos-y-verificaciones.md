# 07 · Riesgos y verificaciones

La mayor fortaleza de este plan es su honestidad: **no depende de nada que no se haya verificado**, y donde algo no está confirmado, lo dice y explica cómo cerrarlo antes de comprometerse. Este documento reúne (1) el registro de incertidumbres G1–G12, (2) los riesgos del track embebido, (3) la honestidad sobre comprimir 10 semanas a 2, y (4) los gates de aprobación de compra.

---

## 1. Registro de afirmaciones que requieren verificación interna (G1–G12)

Ninguna de estas se convierte en dependencia del laboratorio hasta cerrarse. Las marcadas **[bloquea sprint]** deben cerrarse en el Día 0; las **[fase 2]** pueden esperar porque dependen de Opengear o de la expansión.

| ID | Incertidumbre real | Cómo se cierra | Cuándo |
|---|---|---|---|
| **G1** | El selector/datasheet 2026 ofrecen **OM1304-4E-C** con 5G y opción `-LSP`, pero una página del manual OM 25.11.1 dice que OM1300 no soporta celular. | Confirmar con Opengear PM/ventas el SKU `OM1304-4E-C-LSP`, revisión de hardware, región y firmware. Si no se confirma, sustituir por **`CM8004-C-LSP`** + switch de gestión externo. | [fase 2] |
| **G2** | La API pública de Lighthouse no tiene llamada para "run ZTP" ni empujar un Resource Bundle. | Usar **LSP factory-onboarding** como prueba base. Preguntar a ingeniería de Lighthouse si existe API interna/actual para Secure Provisioning antes de incluirla como entregable. | [fase 2] |
| **G3** | La página estática de DRM confirma `POST /ws/v1/configs/scan_now` pero no publica su body. | Leer el resumen autodocumentado `GET /ws/v1/configs` y el API Explorer del tenant de laboratorio. **No adivinar el body.** | **[bloquea sprint]** (Día 4) |
| **G4** | El ejemplo público BGP no nombra el campo de **ASN remoto** del vecino. | Inspeccionar el esquema DAL del firmware exacto (`?` en `network route service bgp neighbour` o la API local `https://{ix40}/cgi-bin/config.cgi`); congelar versión y capturar el nombre real. | **[bloquea sprint]** (Día 3) |
| **G5** | No hay evidencia pública de que un terminal Starlink permita **eBGP de cliente**. | **No** afirmar ni intentar "BGP con Starlink". BGP corre dentro del overlay del lab; Starlink es el underlay IP. | Regla permanente |
| **G6** | Digi documenta sockets SFP de 1 Gb/s en IX40 pero no publica lista cerrada de ópticas validadas. | Prueba de compatibilidad con fabricante, longitud de onda, DOM y temperatura; registrar P/N aprobado. **No comprar SFP+.** | **[bloquea sprint]**† (Día 3) |
| **G7** | ConnectCore 95 aparece en el selector, pero contenido de familia sugiere disponibilidad futura. | No hacerlo dependencia. Usar **`CC-WMP255-KIT`** para el carril embebido/OEM. | Resuelto por diseño |
| **G8** | La asignación de DRM Premier, Push Monitor, Containers, WAN Bonding **y CCCS/firmware repo** depende de entitlements Digi 360 y del tenant. | **License audit** antes de la compra para evitar duplicados y confirmar licencias por dispositivo y que el tenant permite subir `.swu`. | **[bloquea sprint]** (Día 0) |
| **G9** | SKU/plan del Starlink Kit varían por región; IPv4 pública es opcional y no estática por defecto. | Cotización regional (Local/Global Priority); registrar hardware revision, plan, CGNAT/pública, IPv6-PD observado. | **[bloquea sprint]** (Día 0–1) |
| **G10** | Soporte de IPv6, 5G SA y APN cambia por operador celular. | Seleccionar Carrier A (y en fase 2, Carrier B independiente); validar IPv4, IPv6, MTU y NAT antes de la prueba de resiliencia. | **[bloquea sprint]** (Día 0–2) |
| **G11** | Secure Provisioning downstream tuvo asociación histórica con Lighthouse Automation/NetOps; su inclusión en Core/Enhance actual debe confirmarse. | Confirmación de licencia y versión con PM de Lighthouse. Mantenerlo como **extensión**, no criterio de éxito base. | [fase 2] |
| **G12** | El `type` y `vendor_id` de Configuration Manager cambian por modelo. | Leer los valores del **IX40-05 enrolado**; **no** reutilizar el ejemplo TX64 (`4261412874`). | **[bloquea sprint]** (Día 4) |

† **G6 con salida elegante:** si el SFP validado no llega a tiempo, se arranca el sitio por **cobre** en `WAN/ETH1` (Starlink) y `SFP/ETH5` se puebla cuando el óptico esté validado. No bloquea de forma dura el Día 1, pero sí el anuncio de la red de servicio por fibra (Día 3).

---

## 2. Riesgos del track embebido (R-E*)

| ID | Riesgo | Impacto | Mitigación |
|---|---|---|---|
| **R-E1** | Primer `bitbake` tarda horas; dependencias que se rompen. | Retrasa el track central. | Arrancar el **Día 1**; host con recursos (~100+ GB, buena CPU); fijar versiones de capa `meta-digi`. |
| **R-E2** | **`trustfence close` es irreversible** → posible brick. | Pérdida de una unidad. | **Se ejecuta en fase posterior (Anillo 2), fuera del sprint**; practicar el flujo de firma primero; imagen ya estable; **claves en secret store**; unidad sacrificable. |
| **R-E3** | Recetas/versiones DEY y comportamiento exacto varían por SOM/firmware. | Pasos no reproducibles. | Validar contra el kit real; congelar **DEY 4.0 / `ccmp25`** y revisiones. |
| **R-E4** | El `CC-WMP255-KIT` tarda en llegar. | Sin hardware embebido en el sprint. | Ruta **QEMU + Raspberry Pi** (ver [`05`](05-embebido-connectcore.md) §5); el aprendizaje de Yocto y el build no requieren el SOM. |
| **R-E5** | OTA/CCCS requiere entitlements del tenant. | OTA no demostrable. | Confirmar en **G8** que el tenant permite subir `.swu` al firmware repo de DRM. |

---

## 3. Honestidad sobre comprimir 10 semanas a 2

plan2 es un programa de **10 semanas** con campañas de **30 repeticiones**. Elegiste "3 sitios comprimidos" + "embebido central", y eres rápido — pero conviene ser explícito sobre qué **sí** y qué **no** entra en 2 semanas, porque presentar un plan que finge lo imposible te haría quedar mal ante el manager.

**Lo que 2 semanas SÍ entregan (Anillo 0):** 1 sitio Digi completo end-to-end, failover medido, DRM-as-code, telemetría XBee + Container, sensor embebido funcional (imagen Yocto + agente) con buffer/replay, automatización de clonado probada, WAN Bonding A/B, campaña **representativa** (~10 reps) y demo.

**Lo que 2 semanas NO entregan (queda para fase 2):**
- La **campaña estadística de 30 reps × toda la matriz × 3 sitios**: es intrínsecamente semanas de reloj, no de esfuerzo.
- **Opengear OOB/Lighthouse**: por decisión propia va en fase 2 (hardware retrasado).
- **Sitios 2 y 3 físicos** si el hardware no llegó: se demuestra el **método** de clonado, no necesariamente 3 racks.
- **Secure boot del ConnectCore (TrustFence)**: se saca del sprint por ser irreversible (`trustfence close`) y de tiempo impredecible; pasa a fase posterior (Anillo 2).
- **OTA robusto con rollback** y **auto-remediación** (plan1 O1): stretch de Anillo 2.

**Supuestos de los que depende el sprint (si fallan, se ajusta el alcance, no la honestidad):**
1. El hardware Digi del sprint (IX40, MP255, Hive) llega en la primera semana.
2. El tenant DRM tiene los entitlements (G8) desde el Día 0.
3. Starlink y el SIM 5G están activos con cobertura de banco (G9/G10).
4. El plan Starlink entrega IPv6 utilizable (si no, F4 se documenta como limitación del plan/región, no del diseño).

**Dependencias de reloj que no se aceleran por ser rápido:**
- Primer build Yocto (**horas**).
- Entrega/activación de Starlink y SIMs (días, según región).
- `trustfence close` requiere una imagen madura y es irreversible — por eso está en fase posterior, no en el sprint.
- Una campaña de N repeticiones toma N × (tiempo de fallo + recuperación + asentamiento).

---

## 4. Matriz de riesgos de proyecto (no técnicos del lab)

| Riesgo | Prob. | Impacto | Mitigación |
|---|---|---|---|
| Hardware Digi del sprint se retrasa | Media | Alto | Autorizar en el Día 0; ruta QEMU para el embebido; empezar por lo que haya (Starlink+IX40 primero). |
| Opengear se retrasa más de lo previsto | Alta | Bajo (por diseño) | Ya está en fase 2; no toca el camino crítico. |
| Entitlements del tenant incompletos | Media | Alto | License audit (G8) **antes** de comprar; sin Push Monitor no hay webhook. |
| Starlink sin IPv6/pública en la región | Media | Medio | Registrar lo observado (G9); F4 pasa a limitación documentada. |
| Alcance crece (scope creep) por "ser rápido" | Media | Medio | El modelo de anillos fija el compromiso en Anillo 0; lo demás es expansión explícita. |
| Secretos filtrados en artefactos | Baja | Alto | Secret store; sanitizar evidencia; revisión del Día 9; nada de credenciales en el repo. |

---

## 5. Gates de aprobación de compra

La compra completa se aprueba solo cuando se cumpla (adaptado de plan2 §6, con el desdoble sprint/fase 2):

**Para arrancar el sprint (Anillo 0):**
1. **G3** body de `scan_now` validado en el API Explorer del tenant.
2. **G4** nombre exacto del campo de ASN remoto capturado del esquema DAL.
3. **G8** licencias DRM/Containers/Push Monitor/Bonding/**CCCS** reconciliadas contra Digi 360.
4. **G9** plan/hardware Starlink y comportamiento IPv4/IPv6 registrado.
5. **G10** Carrier A disponible y con cobertura de banco.
6. **G6** SFP validado en IX40 — o aceptar arranque por cobre.
7. Firmware IX40/Hive/MP255 congelado.
8. Revisión de seguridad y reglas de publicación aprobadas.
9. El manager acepta objetivos y límites de divulgación.

**Para la fase 2 (Opengear + expansión):**
10. **G1** cerrado o fallback CM8004 aprobado.
11. **G2/G11** Secure Provisioning y su licencia confirmados con PM de Lighthouse (o se mantiene como extensión).
12. Segundo carrier (B) disponible e independiente.
13. Un ingeniero **distinto** acepta ejecutar el runbook (repetibilidad).

---

## 6. Regla de oro

> Ante cualquier campo de API, comando o SKU del que no haya evidencia: **no se adivina.** Se marca aquí, se valida contra el tenant/firmware real, y solo entonces entra al runbook. Esta disciplina es, en sí misma, uno de los entregables que más valora un PM de Digi.
