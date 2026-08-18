# 02 · Requisitos — Lo que necesito

Todo lo que hay que **pedir, autorizar o tener listo** antes y durante el proyecto. Está ordenado por prioridad de adquisición para el enfoque **Digi primero / Opengear fase 2**: lo que bloquea el sprint va arriba; lo de fase 2 va marcado y **no** bloquea el camino crítico.

> Leyenda de disponibilidad:
> **[SPRINT]** necesario para el Anillo 0 (2 semanas) · **[FASE 2]** para expansión / Opengear · **[YA]** probablemente ya lo tienes en el entorno de demo (Lighthouse, Proxmox, SIEM, RADIUS) · **[VERIF]** requiere cerrar un gate G* antes de comprar (ver [`07`](07-riesgos-y-verificaciones.md)).

---

## 1. Cómo pedirlo (resumen para la autorización)

Puedes solicitar autorización de cualquier equipo de la serie Digi; el hardware Opengear puede tardar. Por eso:

- **Para el sprint, pide primero:** 1× IX40-05 (+ starter kit + DIN), 1× ConnectCore `CC-WMP255-KIT`, 1× XBee Hive + kit XBee Zigbee, licencias DRM Premier + Containers, trial de WAN Bonding Tier 3, 1 plan Starlink Priority, y 1 SIM 5G de un carrier. Con **una** unidad de cada cosa ya se entrega el Anillo 0 completo.
- **En paralelo, autoriza (aunque lleguen después):** 2× más de cada IX40/Starlink/SIM para los sitios 2 y 3, y el hardware Opengear (OM1304 + Lighthouse Enhance) para fase 2.
- **Regla de oro:** no comprar nada marcado **[VERIF]** hasta cerrar su gate. En particular, **no comprar SFP+** (el IX40 es solo SFP), y **no comprar OM1304 y CM8004 a la vez** (el CM8004 es fallback solo si G1 no cierra).

---

## 2. Hardware Digi (camino crítico)

| Cant. sprint | Cant. total | Modelo / P/N | Disp. | Uso y razón |
|---:|---:|---|---|---|
| 1 | 3 | **`IX40-05`** | [SPRINT] | Router 5G / BGP / IPv6 / SFP / edge por sitio. Edge router del laboratorio. |
| 1 | 3 | **`76002147`** | [SPRINT] | Starter kit IX40 (fuente 30 W + antenas celulares). Confirmar contenido regional en cotización. |
| 1 | 3 | **`76002093`** | [SPRINT] | Montaje DIN para IX40. |
| 0–1 | 0–3 | **`DG-ESIM-A`** | [VERIF] | eSIM Digi opcional; solo si el carrier/tenant lo justifica. Una nano-SIM 4FF física por IX40 sigue siendo válida. |
| 1 | 3 | **`XH-24P-TW1-101`** | [SPRINT] | XBee Hive Zigbee dev kit (ConnectCore MP157 + radio XBee + LTE/Wi-Fi/Ethernet + DRM), uno por sitio. Telemetría RF. |
| 1 (kit) | 1 (kit) | **`XK3-Z8S-WZM`** | [SPRINT] | Kit XBee 3 Zigbee Mesh con 3 módulos (uno por sitio). Sensor físico. |
| 1 | 1–3 | **`CC-WMP255-KIT`** | [SPRINT] | ConnectCore MP255 dev kit — **el track embebido central** (Yocto/DEY, TrustFence, TSN/NPU, OTA por CCCS). En sprint: 1 unidad como sensor de sitio. En fase 2: carril OEM opcional en Site C. |

### 2.1 Hardware Opengear (fase 2 — NO bloquea el sprint)

| Cant. total | Modelo / P/N | Disp. | Uso y razón |
|---:|---|---|---|
| 3 | **`OM1304-4E-C-LSP`** *(provisional, ver G1)* | [FASE 2][VERIF] | OOB 5G + 4 consolas + switch de gestión + SFP + LSP, compacto. Plano out-of-band por sitio. |
| 3 | **`CM8004-C-LSP`** *(solo fallback)* | [FASE 2][VERIF] | Sustituye al OM1304 **si G1 no cierra**; obliga a añadir switch de gestión externo. **No comprar junto con OM1304.** |
| 3 | **`319015`** | [FASE 2] | Adaptador DB9F–RJ45 crossover DTE para X2 → DB9 DTE del IX40. |

---

## 3. Licencias y servicios

| Cant. planificada | SKU / servicio | Disp. | Nota |
|---:|---|---|---|
| Hasta 6 device-years | **`DIGI-RM-PRM-1YR`** | [SPRINT][VERIF] | DRM Premier por dispositivo (IX40 + Hive). Cantidad neta se reduce por entitlements Digi 360/Hive ya incluidos. Hacer **license audit** (G8) antes de comprar. |
| 1–3 | **`DIGI-RM-PRM-CS`** | [SPRINT] | Digi Containers, asignación por dispositivo (los IX40). |
| 1 (sprint) / 3 | **`DIGI-SRV-WB-TIER3-TRIAL-30D`** | [SPRINT] | Trial de WAN Bonding 1 Gb/s para la campaña A/B. |
| 0–3 (si se conserva) | **`DIGI-SRV-WB-TIER3-1YR`** | [FASE 2] | Suscripción anual 1 Gb/s; **no** simultánea con el trial. |
| 1 tenant | **Push Monitor entitlement** | [SPRINT][VERIF] | Confirmar que el tenant DRM lo incluye; la API de monitors lo requiere (G8). |
| 1 | **Servidor / VPS WAN Bonding** | [SPRINT] | Externo a los routers, dimensionado y situado para no sesgar medidas. P/N según proveedor de infraestructura. |
| 1 | **`LH-ENHANCE-10-1YR`** (Lighthouse Enhance, ≤10 nodos) | [FASE 2][YA?] | Gestión OOB de flota. Puede que ya tengas acceso a Lighthouse en tu entorno de demo; confirmar si sirve o hace falta esta licencia. |
| 3 | **`OG-FNTS-OM1300-1YR`** (Foundation Support) | [FASE 2] | Un año, si se confirma OM1300. Ajustar SKU si se usa CM8000. |

> **CCCS / OTA embebido:** el OTA del ConnectCore MP255 usa **ConnectCore Cloud Services** sobre el **firmware repository de Remote Manager** — es decir, el mismo tenant DRM. Confirmar en el license audit (G8) que el tenant permite subir paquetes `.swu` al repositorio de firmware.

> **Aclaración de tiers DRM:** el catálogo expone **DRM Premier** en 1/3/5 años. No existe "DRM Essentials". "Essential/Premium" pertenece a *ConnectCore Cloud Services*, que es una oferta embedded distinta de los niveles de DRM — no mezclar.

---

## 4. Conectividad (SIMs, planes, Starlink)

| Cant. sprint | Cant. total | Elemento | Disp. | Nota |
|---:|---:|---|---|---|
| 1 | 3 | Plan Starlink **Priority** | [SPRINT][VERIF] | Local o Global Priority según región. Hardware/SKU final por dirección de servicio; IPv4 pública opcional y no estática por defecto (G9). |
| 1 | 3 | Plan celular **Carrier A** (5G IX40) | [SPRINT][VERIF] | Underlay 5G del IX40. Validar IPv4/IPv6/MTU/NAT y 5G SA/APN (G10). |
| 0 | 3 | Plan celular **Carrier B** (5G Opengear) | [FASE 2][VERIF] | OOB del Opengear, **carrier distinto** del IX40. Solo fase 2. |
| 0–1 | 0–3 | Plan LTE Cat 1 **Hive** | [VERIF] | Solo si se prueba Hive fuera del LAN; Ethernet puede bastar para la demo base. |

**Requisito de independencia (crítico para el OOB):** Carrier A ≠ Carrier B, y validar cobertura de banco de ambos operadores **antes** de la prueba de resiliencia.

---

## 5. Infraestructura auxiliar

| Cant. sprint | Cant. total | Elemento | Disp. | Nota |
|---:|---:|---|---|---|
| 1 par | 3 pares | Ópticas SFP **1 Gb/s** + fibra | [SPRINT][VERIF] | P/N final tras G6 (compatibilidad, longitud de onda, DOM, temperatura). **Prohibido SFP+.** |
| 1 | 1 | Switch / fabric óptico gestionable | [SPRINT] | Vendor-neutral; conecta cargas y entorno de prueba. |
| 1 host/clúster | 1 | Core / route-collector / observabilidad | [SPRINT][YA?] | AS65000, terminación de overlay, receptor de webhook y métricas. Puede correr sobre tu Proxmox existente. |
| 1 | 1 | Plataforma de impairment | [SPRINT][YA?] | Reproduce pérdida, jitter, latencia, blackhole y rate-limit. `tc/netem` sobre un host Linux basta. |
| 1 | 2 | UPS / circuitos de energía | [SPRINT] | Separan producción y OOB (2 en fase 2). |
| 1 | 3+ | Tomas PDU controlables | [SPRINT] | Power-cycle reproducible con logging (para el fallo de reboot/power loss). |
| — | 1 | Rack, patch panels, antenas/cables, etiquetado | [SPRINT] | Construcción limpia y repetible. |
| 1 | 1 | Host de build **DEY** (Yocto) | [SPRINT] | Linux con ~100+ GB libres y buena CPU; el primer `bitbake` tarda horas. Puede ser una VM en Proxmox. Ver [`05`](05-embebido-connectcore.md) §1. |

---

## 6. Cuentas, accesos y credenciales

| Elemento | Disp. | Nota |
|---|---|---|
| Tenant **Digi Remote Manager Premier** con API habilitada | [SPRINT][VERIF] | Usuario de aplicación/lectura o API key (`X-API-KEY-ID`/`X-API-KEY-SECRET`). Confirmar entitlements (Push Monitor, Containers, CCCS/firmware repo) — G8. |
| **SIEM** (receptor de logs + webhook) | [SPRINT][YA] | Ya en tu entorno de demo. Recibe Push Monitor (`alert_status`), DataPoints y los eventos del agente embebido. |
| **RADIUS/TACACS+** | [YA] | Para acceso privilegiado auditado (roles solo-lectura vs admin). Del entorno de demo. |
| **Proxmox** | [YA] | Host de VMs (core/collector, impairment, Lighthouse en fase 2, build DEY). |
| Acceso **Lighthouse** | [FASE 2][YA?] | Confirmar si tu Lighthouse de demo sirve o hace falta `LH-ENHANCE-10-1YR`. |
| `install_code` de cada IX40 | [SPRINT] | Código seguro impreso en la etiqueta del equipo. **No inventar ni guardar en el repo.** |
| **Secret store / vault** del tenant | [SPRINT] | Para API keys, tokens y credenciales de túnel. Nunca dentro de una plantilla exportada ni del repositorio. |

---

## 7. Prerrequisitos de conocimiento / herramientas

- **Ansible + Python** (para IaC de flota y clientes de API DRM/Lighthouse).
- **Git** (config como datos; GitOps es stretch del Anillo 2).
- **Yocto / Digi Embedded Yocto (DEY)** — curva de aprendizaje real; ver [`05`](05-embebido-connectcore.md).
- **BGP / IPv6 / IPsec** operativos (no solo conceptuales).
- `tc/netem` u otra herramienta de impairment.
- Cliente HTTP (curl/httpie) para hablarle **primero** a las APIs a mano antes de automatizar.

---

## 8. Checklist de "listo para empezar el sprint"

Marcar antes del Día 1 (detalle y gates en [`07`](07-riesgos-y-verificaciones.md)):

- [ ] 1× IX40-05 + starter kit + DIN autorizados y en camino/entregados.
- [ ] 1× `CC-WMP255-KIT` autorizado.
- [ ] 1× XBee Hive + kit XBee Zigbee autorizados.
- [ ] Tenant DRM Premier con API, y license audit hecho (G8): Premier + Containers + Push Monitor + CCCS confirmados.
- [ ] Trial WAN Bonding Tier 3 disponible + servidor/VPS de bonding aprovisionado.
- [ ] 1 plan Starlink Priority contratado; CGNAT/pública/IPv6-PD por observar (G9).
- [ ] 1 SIM 5G Carrier A con cobertura de banco validada (G10).
- [ ] Host de build DEY listo (Linux, ~100+ GB, CPU decente).
- [ ] Core/collector, SIEM, impairment y Proxmox operativos (del entorno de demo).
- [ ] SFP 1 Gb/s validado en el IX40 (G6) — **o** aceptar arrancar por cobre y añadir la fibra cuando el SFP esté validado.
- [ ] Firmware IX40 / Hive / MP255 congelado (matriz de versiones).
- [ ] (Fase 2, en paralelo) OM1304 + Lighthouse Enhance autorizados aunque lleguen después; G1 en proceso de cierre.

La justificación de cada selección frente a sus alternativas está en [`01-plan-maestro.md`](01-plan-maestro.md) §6 y [`03-arquitectura.md`](03-arquitectura.md). Los gates de aprobación de compra están en [`07-riesgos-y-verificaciones.md`](07-riesgos-y-verificaciones.md) §Gates.
