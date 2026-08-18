# 01 · Plan Maestro

**Proyecto:** Digi Starlink-Primary Resilient Gateway Fleet — Reference Lab + Embedded Site Sensor
**Contexto:** capstone de internship en Digi International.
**Duración objetivo:** sprint de 2 semanas (ampliable, camino de expansión definido).
**Documento:** plan combinado a partir de `plan1` (los 4 proyectos) y `plan2` (el laboratorio de referencia de 3 sitios).

> Este es un documento de planificación. No contiene scripts ni configuraciones listas para ejecutar. Los nombres de rutas, campos CLI/API, SKUs y comandos se incluyen porque forman parte de la prueba técnica que se solicita demostrar.

---

## 1. Decisión ejecutiva (una frase)

> Construyo un laboratorio de referencia repetible donde **routers industriales Digi IX40-05 entregan servicios dual-stack y rutas BGP sobre un underlay Starlink-primary**, con **5G in-band como respaldo**, **gestión central como código por Digi Remote Manager**, **telemetría física XBee**, un **sensor de sitio embebido en ConnectCore MP255 con Yocto, gestionado por el mismo Remote Manager**, y una **comparación cuantitativa entre failover nativo (SureLink) y WAN Bonding** — todo verificado con inyección de fallos medida. El plano **out-of-band Opengear/Lighthouse** se integra como fase 2 cuando llegue el hardware.

Esta formulación satisface dos audiencias a la vez:

- **Digi (el manager):** un activo de interoperabilidad y enablement vendible — arquitectura de referencia "Starlink primary + Digi 5G + Opengear OOB" reutilizable por preventa, soporte y documentación, con hallazgos concretos para PM.
- **Starlink (tu carrera):** evidencia técnica de BGP/IPv6, separación underlay/overlay, convergencia medida, operación de flotas a escala, OOB independiente y software Linux embebido de calidad — exactamente los dos perfiles que Starlink contrata en tierra (Reliability/Ground Network Specialist **y** Software Engineer, Ground Stations).

El laboratorio **no** emula la red propietaria de Starlink ni afirma acceso a sus protocolos internos. Simula patrones de una flota de gateways y **mide** su comportamiento sobre un servicio Starlink comercial.

---

## 2. De dónde viene esto: fusión de plan1 y plan2

| Fuente | Qué aportaba | Qué tomo |
|---|---|---|
| **plan1** | 4 proyectos independientes: O1 (gateway autónomo con auto-remediación + SIEM), O2 (fleet provisioning por API), D1 (edge resiliente Starlink-WAN como IaC), D2 (sensor embebido ConnectCore/Yocto). | La **historia de carrera Starlink**, el **lazo cerrado detección→remediación→verificación** (de O1, como stretch), la **automatización de flota** (de O2, como el mecanismo de clonado) y, sobre todo, el **track embebido D2 elevado a entregable central**. |
| **plan2** | Un único laboratorio de referencia de 3 sitios, muy maduro: selección de producto justificada, contratos de API DRM/Lighthouse verificados, config real de DAL OS, registro de incertidumbres G1–G12, matriz de fallos, criterios de aceptación, BOM con SKUs y plan de 10 semanas. | La **arquitectura completa**, el **rigor de verificación**, la **BOM**, la **matriz de fallos** y los **criterios de aceptación**. Es el esqueleto del plan. |

**Reconciliación:** plan2 es el cuerpo; plan1 aporta el músculo embebido (D2 → central), la automatización de flota (O2 → mecanismo de clonado) y el lazo de auto-remediación (O1 → stretch). El resultado es **un solo laboratorio con dos narrativas que se refuerzan**: resiliencia de red medida + edge embebido de calidad de software, ambos bajo el mismo plano de gestión (Digi Remote Manager).

---

## 3. Objetivos

### 3.1 Para Digi (audiencia principal)

Producir una arquitectura de referencia repetible para clientes que usan Starlink como WAN primaria con Digi 5G, SureLink, DRM, Containers, WAN Bonding y Opengear OOB; generar datos de interoperabilidad y hallazgos documentales (G1–G12) útiles para ingeniería, preventa, soporte y enablement; y demostrar **uso significativo, no decorativo**, de IX40, DRM, Containers, XBee Hive, ConnectCore/Yocto, WAN Bonding y (fase 2) Opengear/Lighthouse.

### 3.2 Para Starlink (mapeo de perfiles)

| Perfil Starlink | Qué del lab lo evidencia |
|---|---|
| Ground Network Specialist / Reliability Engineer | Diagnóstico y reparación de hardware de tierra, convergencia de failover medida, OOB independiente, telemetría operacional. |
| Software Engineer, Ground Stations | Imagen Linux custom con **Yocto** sobre **ARM** (ConnectCore MP255), secure boot, sistema tolerante a fallos con mantenimiento mínimo, envío resiliente y OTA. |
| SecOps / seguridad de infraestructura distribuida | Acceso privilegiado auditado, segmentación management/servicio/OOB, detección en sitios desatendidos, respuesta cuando el enlace principal cae. |

---

## 4. Alcance: modelo de anillos concéntricos

plan2 es de 10 semanas con campañas de 30 repeticiones. **Eso no cabe entero en 2 semanas**, y fingir que sí reprobaría la exigencia de "verificar los pasos". El alcance se organiza en tres anillos; el sprint compromete el Anillo 0 y deja los anillos 1 y 2 como expansión mapeada.

### Anillo 0 — Sprint de 2 semanas (el compromiso del capstone)

Entregables **garantizados** al final de la semana 2:

- **1 sitio Digi completo end-to-end:** IX40-05 + Starlink Performance Kit (WAN primaria) + 5G IX40 (WAN backup) + `SFP/ETH5` a fabric de servicio + overlay IKEv2/IPsec + **eBGP** al core AS65000 con **IPv4 e IPv6** y filtros + **SureLink** multicapa afinado + failover **medido**.
- **DRM-as-code:** enrolamiento por API, golden config (Configuration Manager), settings por sitio, scan/compliance, alarmas y **Push Monitor → SIEM**.
- **Track embebido a estado funcional:** imagen **DEY (Yocto)** que arranca en ConnectCore MP255, **receta propia** con el **agente** que recolecta y envía al SIEM con **buffer offline + reenvío**. *(Secure boot y OTA quedan explícitamente para fase posterior — ver Anillo 2.)*
- **Telemetría física:** un sensor **XBee Zigbee** vía **XBee Hive** → DataPoints DRM; **Digi Container** ligero de métricas en el IX40 con límites de recursos.
- **Automatización de clonado:** plantillas DRM + Ansible que levantan un sitio nuevo con un comando (`make deploy`), idempotente.
- **WAN Bonding A/B:** Tier 3 en ese sitio; comparación native vs bonding con el **mismo** impairment trace.
- **Campaña de pruebas representativa:** ≈10 repeticiones en los fallos principales (desconexión Starlink, blackhole aguas arriba, DNS-only), con p50/p95 y trazas correlacionadas.
- **Demo grabada** de 12 min + versión de 90 s, one-pager ejecutivo y README sanitizado.

### Anillo 1 — Si el hardware llega a tiempo (dentro del sprint o inicio de fase 2)

- **Sitios 2 y 3 físicos**, levantados por la automatización ya probada (aquí "3 sitios" se vuelve real: sitio 2 y 3 son `make deploy`, no reconstrucción manual).
- **Opengear OOB/Lighthouse** integrado: OM1304 por LSP en Lighthouse Enhance, consola serial del IX40 en modo Login, RBAC, failover OOB. **Este es el motivo del enfoque "Digi primero, Opengear fase 2":** el retraso de Opengear no bloquea nada del camino crítico.

### Anillo 2 — Programa completo (fase 2, mapea a las semanas 3–10 de plan2)

- **Rotación de perfiles A/B/C** entre 3 unidades (elimina sesgo de una unidad/terminal/celda).
- **Campaña de 30 repeticiones** por fallo con p50/p95/p99 y publicación de datos brutos.
- **OTA completo** del agente embebido (`.swu` vía ConnectCore Cloud Services / firmware repo de DRM).
- **TrustFence secure boot** en el ConnectCore: firma de U-Boot/kernel, `SRK_efuses.bin`, `trustfence close` (irreversible), cifrado de rootfs. Se saca del sprint por ser irreversible y de tiempo impredecible.
- **Carril OEM** con ConnectCore MP255 como controlador simulado (TSN/NPU).
- **Matriz de interoperabilidad completa** por firmware, carrier, IPv4/IPv6, MTU y hardware Starlink.
- **Lazo de auto-remediación** (de plan1 O1): remediación decidida por árbol según tipo de fallo, verificada, con rollback.
- Cierre de **G1, G2, G11** (dependientes de Opengear) y backlog de hallazgos para PM.

> La demo del final del sprint muestra 1 sitio físico funcionando + la automatización que pararía la flota. Si el hardware de sitios 2-3 llegó, se muestran físicos.

---

## 5. Qué NO es (límites y honestidad)

- No es una réplica de la red de Starlink ni una afirmación de eBGP de cliente con Starlink. BGP corre **dentro del overlay** del laboratorio; Starlink es el **underlay IP** (ver G5).
- No es un plan que garantice completar las 10 semanas de plan2 en 2. El sprint entrega un **subconjunto coherente y demoable**, no la campaña estadística completa.
- No inventa contratos de API ni comandos: donde la documentación pública no confirma algo, se marca en el registro G1–G12 y **se valida contra el tenant/firmware real antes de depender de ello**.
- No comparte energía ni carrier entre el plano in-band y el OOB; si lo hiciera, el supuesto de independencia sería falso.

---

## 6. Componentes clave y por qué ganan (resumen)

El detalle de selección, alternativas y tradeoffs está en [`02-requisitos.md`](02-requisitos.md) y [`03-arquitectura.md`](03-arquitectura.md). Resumen:

| Rol | Selección | Por qué gana en una línea |
|---|---|---|
| Edge router | **IX40-05** | Único ajuste actual con 5G industrial + BGP/IPv6 + dos SFP 1 Gb/s + serial/E-S + edge compute + SureLink + DRM + Containers + Bonding. |
| WAN primaria | **Starlink Performance Kit (Gen 3)** | Orientación comercial fija, fuente AC/DC avanzada, vida de diseño para despliegues exigentes. |
| WAN backup | **5G del IX40** | Segundo underlay independiente con SIM/APN propio. |
| Gestión Digi | **DRM Premier** | Configuration Manager, API, alerts, monitors, data streams, Containers y VAS en el mismo plano — y **también** el OTA del ConnectCore vía CCCS. |
| Telemetría RF/edge | **XBee Hive `XH-24P-TW1-101`** + XBee 3 Zigbee | ConnectCore + XBee + LTE/Wi-Fi/Ethernet + edge + DRM sin diseñar carrier board. |
| Sensor embebido (central) | **ConnectCore MP255 `CC-WMP255-KIT`** | Carril de software embebido moderno: Yocto/DEY, TrustFence, TSN/NPU, OTA por CCCS. |
| Resiliencia de datos | **Failover nativo + experimento WAN Bonding** | Compara dos mecanismos distintos con el mismo hardware y los mismos fallos. |
| OOB (fase 2) | **OM1304-4E-C-LSP** (provisional, ver G1) | Consola + 5G independiente + switch de gestión + SFP + LSP compacto; fallback `CM8004-C-LSP`. |
| Gestión OOB (fase 2) | **Lighthouse Enhance** | Fabric, routed IP, observabilidad y automatización de flota, no solo portal de consolas. |

---

## 7. Mapa de documentos del paquete

| # | Documento | Para qué |
|---|---|---|
| 00 | [`00-README.md`](00-README.md) | Índice y resumen ejecutivo de una página. |
| 01 | **este documento** | El plan combinado, objetivos y alcance. |
| 02 | [`02-requisitos.md`](02-requisitos.md) | Todo lo que necesito pedir/autorizar: BOM, licencias, SIMs, planes, infra, cuentas. |
| 03 | [`03-arquitectura.md`](03-arquitectura.md) | Cómo queda: topología, cableado, direccionamiento, diagramas. |
| 04 | [`04-pasos-montaje.md`](04-pasos-montaje.md) | Cómo montarlo: runbook día a día del sprint + fase 2. |
| 05 | [`05-embebido-connectcore.md`](05-embebido-connectcore.md) | El track embebido central en detalle (Yocto, agente, secure boot, OTA). |
| 06 | [`06-pruebas-y-demo.md`](06-pruebas-y-demo.md) | Matriz de fallos, criterios de aceptación, guion de demo, artefactos. |
| 07 | [`07-riesgos-y-verificaciones.md`](07-riesgos-y-verificaciones.md) | Registro G1–G12, riesgos, honestidad de la compresión, gates. |
| 08 | [`08-executive-pitch-EN.md`](08-executive-pitch-EN.md) | Pitch de una página al manager (en inglés). |

---

## 8. Criterios de éxito del sprint (nivel plan)

El sprint es un éxito si, al final de la semana 2:

1. Un sitio Digi completo entrega servicio dual-stack con **failover Starlink→5G medido** (objetivo de trabajo p95 ≤ 30 s tras afinar SureLink; si no se logra, se publica la medición y la causa).
2. El **collector AS65000** recibe exactamente los prefijos IPv4/IPv6 esperados, cero route leaks.
3. La **imagen embebida (Yocto/DEY) arranca en el MP255**, el agente envía al SIEM y **no pierde datos** ante un corte de red (buffer + replay). *(Secure boot: fase posterior.)*
4. Un **segundo despliegue** (sitio 2 físico o simulado) se levanta con la automatización, idempotente.
5. La comparación **native vs WAN Bonding** existe con el mismo impairment trace.
6. Hay una **demo grabada** correlacionable con IDs de fallo y un one-pager para el manager.

Los criterios técnicos detallados (p50/p95/p99, 30 reps, etc.) del programa completo están en [`06-pruebas-y-demo.md`](06-pruebas-y-demo.md) §Criterios de aceptación.
