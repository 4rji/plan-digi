# Digi Starlink-Primary Resilient Gateway Fleet — Reference Lab + Embedded Site Sensor

Paquete de planificación del capstone de internship. Combina `plan1` (los 4 proyectos) y `plan2` (el laboratorio de referencia de 3 sitios) en **un solo plan** ejecutable en un **sprint de 2 semanas** (Anillo 0), con un camino de expansión claro.

> Todo es planificación: rutas, campos de API, SKUs y comandos se citan porque forman parte de la prueba técnica, **no** como scripts listos para ejecutar.

---

## Resumen ejecutivo (1 página)

**Qué:** un laboratorio repetible donde routers **Digi IX40-05** entregan servicios dual-stack y rutas **BGP** sobre un underlay **Starlink-primary**, con **5G in-band** de respaldo, **gestión como código por Digi Remote Manager**, **telemetría física XBee**, un **sensor de sitio embebido en ConnectCore MP255 (Yocto, gestionado por el mismo DRM)**, y una **comparación medida entre failover nativo (SureLink) y WAN Bonding**. El **OOB Opengear/Lighthouse** entra en fase 2.

**Para quién:** principalmente el **manager de Digi** (activo de interop/enablement vendible); en segundo plano, tu **portafolio hacia Starlink** (BGP/IPv6, flotas a escala, OOB, software embebido en ARM).

**Cómo cabe en 2 semanas — modelo de anillos:**
- **Anillo 0 (sprint, comprometido):** 1 sitio Digi completo end-to-end + embebido funcional (imagen Yocto + agente, buffer/replay) + automatización de clonado + WAN Bonding A/B + campaña representativa (~10 reps) + demo.
- **Anillo 1 (si llega hardware):** sitios 2-3 físicos por automatización + Opengear OOB/Lighthouse.
- **Anillo 2 (programa completo):** rotación A/B/C de 3 sitios, 30 reps con p50/p95/p99, **secure boot (TrustFence)**, OTA robusto, carril OEM, matriz de interop completa.

**Decisiones que definieron el plan:** 3 sitios comprimidos · audiencia = manager Digi · **Digi primero / Opengear fase 2** (por posible retraso del hardware) · **track embebido ConnectCore/Yocto elevado a central**.

**Regla de honestidad:** nada que no esté verificado se convierte en dependencia. Las 12 incertidumbres (G1–G12) y los riesgos del embebido (R-E*) se cierran antes de comprometerse — ver [`07`](07-riesgos-y-verificaciones.md).

---

## Cómo navegar el paquete

| # | Documento | Idioma | Léelo si quieres… |
|---|---|---|---|
| 00 | **este README** | ES | El mapa y el resumen de una página. |
| 01 | [`01-plan-maestro.md`](01-plan-maestro.md) | ES | El plan combinado: objetivos, fusión plan1+plan2, modelo de anillos, alcance. |
| 02 | [`02-requisitos.md`](02-requisitos.md) | ES | **Lo que necesito pedir/autorizar**: BOM, licencias, SIMs, planes, infra, cuentas, checklist. |
| 03 | [`03-arquitectura.md`](03-arquitectura.md) | ES | **Cómo queda montado**: topología, cableado, direccionamiento, diagramas. |
| 04 | [`04-pasos-montaje.md`](04-pasos-montaje.md) | ES | **Cómo montarlo**: runbook día a día del sprint + fase 2. |
| 05 | [`05-embebido-connectcore.md`](05-embebido-connectcore.md) | ES | El **track embebido central**: Yocto/DEY, agente y buffer/replay (Ring 0); TrustFence secure boot + OTA por CCCS (fase posterior). |
| 06 | [`06-pruebas-y-demo.md`](06-pruebas-y-demo.md) | ES | Matriz de fallos, criterios de aceptación, guion de demo (12 min + 90 s), artefactos. |
| 07 | [`07-riesgos-y-verificaciones.md`](07-riesgos-y-verificaciones.md) | ES | Registro G1–G12, riesgos del embebido, honestidad de la compresión, gates de compra. |
| 08 | [`08-executive-pitch-EN.md`](08-executive-pitch-EN.md) | **EN** | El **pitch de una página al manager** (para presentar/enviar). |

### Rutas de lectura sugeridas
- **Para presentar al jefe:** 08 (pitch EN) → 01 (plan) → 02 (lo que pides).
- **Para montarlo tú:** 02 (conseguir todo) → 03 (entender la estructura) → 04 (ejecutar) → 05 (embebido en paralelo) → 06 (probar y demostrar).
- **Para de-riesgar antes de comprar:** 07 (gates G1–G12) → 02 (checklist de "listo para empezar").

---

## Estado de "listo para empezar"

Antes del Día 1 hay que cerrar los gates que **bloquean el sprint** (G3, G4, G6†, G8, G9, G10, firmware congelado, seguridad aprobada) y tener autorizado el hardware Digi mínimo. La checklist completa está en [`02-requisitos.md`](02-requisitos.md) §8 y los gates en [`07-riesgos-y-verificaciones.md`](07-riesgos-y-verificaciones.md) §5.

## Origen

- `plan1` — 4 proyectos (O1/O2 Opengear, D1/D2 Digi). Aporta la historia Starlink, el clonado de flota (O2), el lazo de auto-remediación (O1) y el **track embebido D2 → central**.
- `plan2` — laboratorio de referencia de 3 sitios/10 semanas. Aporta la arquitectura, el rigor de verificación, la BOM, la matriz de fallos y los criterios de aceptación.
