# 09 · Vacantes SpaceX/Starlink — mapeo desde el capstone

**Fuente:** [spacex.com/careers/jobs](https://www.spacex.com/careers/jobs) (las vacantes viven en Greenhouse; los IDs y títulos cambian — verificar antes de aplicar).
**Fecha de revisión:** 2026-07-29.
**Contexto:** este documento cruza las vacantes reales de Starlink (segmento tierra) con la evidencia que produce el laboratorio del capstone (ver [`01-plan-maestro.md`](01-plan-maestro.md) §3.2).

---

## 1. Ranking de posiciones objetivo

### 🥇 1. Software Engineer, Ground Stations (Starlink) — *Gateway Software Team*

> Vacantes vistas: [Software Engineer, Ground Stations (Starlink)](https://job-boards.greenhouse.io/spacex/jobs/8357115002) · [Software Engineer, Starlink Ground Stations](https://boards.greenhouse.io/spacex/jobs/7788972002)

**Qué piden:** operar y escalar una red global de ground stations; grado en CS/ingeniería/matemáticas; preferido C/C++ en producción, protocolos de red (TCP/UDP/IP), sistemas operativos, y **ownership completo de desarrollo → pruebas → despliegue en aplicaciones reales**.

**Por qué es la ideal:** es literalmente el tema del capstone — *gateways* de tierra resilientes. El track embebido (ConnectCore MP255, imagen Yocto propia, agente con buffer offline + replay, OTA) demuestra exactamente "software Linux en ARM para sitios desatendidos que no pierde datos cuando cae el enlace". El plan ya la identifica como perfil objetivo (§3.2).

**Qué mostrar del lab:**
- Imagen DEY/Yocto propia arrancando en MP255 + receta del agente (Anillo 0).
- Buffer/replay ante corte de red — tolerancia a fallos medida, no afirmada.
- (Fase 2) TrustFence secure boot y OTA por CCCS — súbelo de prioridad si apuntas aquí.

**Gap a cerrar:** C/C++ en producción. Si el agente embebido está en Python/shell, considera reescribir el núcleo en C++ o Rust y documentarlo.

---

### 🥈 2. Software Engineer (Starlink Ground Network) — *automatización de red*

> Vacante vista: [Software Engineer (Starlink Ground Network)](https://job-boards.greenhouse.io/spacex/jobs/8342599002)

**Qué piden:** escalar despliegue, aprovisionamiento y operación mediante **automatización de red robusta**.

**Por qué encaja:** el entregable "DRM-as-code + Ansible + `make deploy` idempotente" es un caso de fleet provisioning por API de libro. La comparación SureLink vs WAN Bonding con el mismo impairment trace demuestra pensamiento de ingeniería de red medible.

**Qué mostrar del lab:** enrolamiento por API, golden config, clonado de sitio con un comando, Push Monitor → SIEM, eBGP dual-stack con filtros y cero route leaks.

---

### 🥉 3. Ground Network Specialist (Starlink) — *campo/operaciones*

> Vacante vista: [Ground Network Specialist (Starlink)](https://spacecrew.com/space-jobs/lwhhdmj0-spacex-ground-network-specialist-starlink) · [detalle en Built In](https://builtin.com/job/ground-network-specialist-starlink/8229930)

**Qué piden:** desplegar, diagnosticar y reparar hardware de tierra de Starlink; **~75% de viaje** doméstico e internacional; perfil multidisciplinario, curioso, resolutivo bajo presión.

**Por qué encaja:** el lab entero es diagnóstico de gateway: inyección de fallos (desconexión Starlink, blackhole, DNS-only), failover medido p50/p95, telemetría operacional, OOB independiente (fase 2 Opengear). Es la ruta de entrada con menos barrera de titulación/experiencia — buena opción como primer pie dentro si el objetivo es entrar ya.

**A tener en cuenta:** es rol de campo (75% viaje), no de diseño. Úsala como plan B o puerta de entrada, no como destino.

---

### Otras relacionadas (monitorear)

| Vacante | Nota |
|---|---|
| [Network Engineer (Starlink)](https://job-boards.greenhouse.io/spacex/jobs/8213701002) | Más senior; el eBGP/IPv6 del lab es la evidencia, pero suelen pedir años de experiencia en redes de producción. |
| [Ground Network Field Technician (Starlink)](https://spacecrew.com/space-jobs/m6vunyhf-spacex-ground-network-field-technician-starlink) | Escalón por debajo del Specialist; remoto. Solo como última puerta de entrada. |
| Principal Software / Optical Network Engineer (Ground Network) | Fuera de alcance por seniority; útiles para leer hacia dónde escala el equipo. |

---

## 2. Requisitos transversales a tener en cuenta

1. **Titulación:** las vacantes de SWE piden Bachelor's en CS/ingeniería/matemáticas/ciencias. Ten el estado de tu grado claro en el CV.
2. **ITAR/EAR:** SpaceX exige ser ciudadano de EE. UU., residente permanente, o asilado/refugiado — **verificar tu elegibilidad antes de invertir en la aplicación**. Es el filtro duro número uno.
3. **C/C++ en producción:** aparece como preferido en casi todas las de software de tierra. Es el gap más rentable de cerrar desde el capstone.
4. **Ownership end-to-end:** piden haber sido dueño de desarrollo + pruebas + despliegue reales. El capstone lo cubre si lo cuentas así: *diseñé, verifiqué (G1–G12), medí y demostré*.
5. **Ritmo y disponibilidad:** cultura de horas extendidas; el Specialist exige ~75% de viaje.
6. **Evidencia medida > afirmaciones:** el diferenciador del capstone es que todo está *medido* (p95 de failover, cero route leaks, buffer sin pérdida). Lidera con números en CV y entrevista.

---

## 3. Cómo contar el capstone según la vacante

| Si aplicas a… | Titular del proyecto en el CV |
|---|---|
| SWE Ground Stations | "Custom Yocto Linux image + fault-tolerant telemetry agent (offline buffer/replay, OTA) on ARM for unattended gateway sites, fleet-managed" |
| SWE Ground Network | "API-driven fleet provisioning: idempotent one-command site deployment (golden config as code), dual-stack eBGP over a Starlink-primary underlay" |
| Ground Network Specialist | "Measured Starlink→5G failover (p50/p95) under injected faults, with independent OOB access and correlated telemetry" |

---

## 4. Acciones concretas

- [ ] Verificar elegibilidad ITAR antes de nada.
- [ ] Portar el núcleo del agente embebido a C++ (o documentar componentes C/C++ existentes) — gap #1 para la vacante ideal.
- [ ] Priorizar en fase 2: TrustFence secure boot + OTA (refuerzan SWE Ground Stations).
- [ ] Publicar el README sanitizado + demo de 90 s (ya en Anillo 0) como portafolio enlazable.
- [ ] Re-verificar las vacantes en [spacex.com/careers/jobs](https://www.spacex.com/careers/jobs) — los IDs de Greenhouse rotan; filtrar por "Starlink" + "ground".

---

**Fuentes:** [SpaceX Careers](https://www.spacex.com/careers/jobs) · [SWE Starlink Ground Stations (Greenhouse)](https://boards.greenhouse.io/spacex/jobs/7788972002) · [SWE Ground Stations (Greenhouse)](https://job-boards.greenhouse.io/spacex/jobs/8357115002) · [SWE Starlink Ground Network (Greenhouse)](https://job-boards.greenhouse.io/spacex/jobs/8342599002) · [Network Engineer Starlink (Greenhouse)](https://job-boards.greenhouse.io/spacex/jobs/8213701002) · [Ground Network Specialist (spacecrew)](https://spacecrew.com/space-jobs/lwhhdmj0-spacex-ground-network-specialist-starlink) · [Ground Network Specialist (Built In)](https://builtin.com/job/ground-network-specialist-starlink/8229930) · [Ground Network Field Technician (spacecrew)](https://spacecrew.com/space-jobs/m6vunyhf-spacex-ground-network-field-technician-starlink)
