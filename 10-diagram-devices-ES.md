# 10 · Diagrama del laboratorio — Función de cada aparato (referencia)

Complemento de [`assets/lab-diagram-tabloid-EN.pdf`](assets/lab-diagram-tabloid-EN.pdf). Lista **qué hace cada aparato del diagrama**. Este documento **no** modifica el diagrama; solo lo explica.

> Convención: un sitio en el sprint (Anillo 0 = Sitio A); tres sitios + control central en la expansión (Anillos 1–2). Todo el hardware base es idéntico entre sitios; solo cambia el **perfil** (native / bonding / candidate) que se rota entre unidades.

---

## Control central — nube + host del lab (Proxmox) · AS65000

| Aparato | Función |
|---|---|
| **Digi Remote Manager Premier** (nube) | Plano único de gestión: golden config + ajustes por sitio, alertas (Device Offline / noncompliance), webhook Push Monitor → SIEM, despliegue de Digi Containers y OTA por CCCS del sensor embebido. |
| **Core / route collector** | Endpoint público del overlay (IKEv2/IPsec) y peer eBGP de cada sitio (AS65000). Recoge la red que origina cada sitio. |
| **SIEM** | Recibe los webhooks de Push Monitor y el stream del agente del MP255; correlación de eventos / seguridad. |
| **Plataforma de impairment** | Inyecta pérdida / jitter / latencia controlados para reproducir condiciones Starlink durante las pruebas. |
| **Servidor WAN Bonding** | Servidor externo Tier 3 (1 Gb/s) que exige el experimento de WAN bonding (A/B vs SureLink). |
| **Lighthouse Enhance VM** *(Fase 2)* | Plano de gestión OOB de Opengear. **Solo se necesita si el aparato OOB es el Opengear OM1304**; con un **Digi Connect IT 4** el OOB se gestiona desde Digi Remote Manager (sin VM aparte). |

## Sitio A — AS65001 · perfil NATIVE · Anillo 0 (sprint de 2 semanas)

| Aparato | Función |
|---|---|
| **Starlink Performance Kit (Gen 3)** | WAN primaria. Cobre a `WAN/ETH1` del IX40; entrega CGNAT `100.64.0.0/10` (IPv4 pública opcional). |
| **Carrier A — 5G** | WAN de respaldo in-band por el módem 5G del IX40; SIM/APN propio. |
| **Digi IX40-05** | El gateway resiliente. Corre el failover SureLink, el overlay IKEv2/IPsec, eBGP (AS65001) y el Digi Container. |
| **Digi Container** | Collector de métricas que corre **en** el IX40 con límites de recursos (demuestra Containers sin degradar el forwarding). |
| **Service fabric (switch)** | Red de servicio detrás de `SFP/ETH5` (SFP 1 Gb/s, nunca SFP+): IPv4 `/24` + IPv6 `/64` anunciada por eBGP. |
| **Digi XBee Hive** | Gateway de RF; aísla la app de sensores del forwarding de paquetes. |
| **Digi XBee 3 (Zigbee)** | Radio Zigbee que enlaza el Hive con el sensor físico. |
| **Sensor físico** | Mide temperatura / estado de puerta / una entrada digital; no forma parte del forwarding crítico. |
| **Digi ConnectCore MP255** | SoM embebido. Imagen DEY (Yocto); agente → SIEM (buffer offline + replay); OTA vía CCCS. |
| **Console server OOB** *(Fase 2)* — **Opengear OM1304-4E-C-LSP** *y/o* **Digi Connect IT 4** | Rescate out-of-band: consola serial (DB9, modo `Login`) al IX40, **5G Carrier B** independiente, switch de gestión y energía independiente. Dos opciones intercambiables — ver la nota abajo. Candidato a desplegarse en el **Sitio B**. |
| **Carrier B — 5G** | Carrier independiente que alimenta el plano OOB (operador y energía distintos a Carrier A / Starlink). |
| **PDU/UPS primaria** | Alimenta Starlink + IX40 + carga. |
| **PDU/UPS secundaria** | Alimenta el aparato OOB + módem B (plano de energía independiente). |

## Sitios B y C (expansión · Anillos 1–2)

| Aparato | Función |
|---|---|
| **Sitio B — AS65002** | Hardware idéntico al Sitio A. Perfil **BONDING**: Starlink + 5G dentro del túnel agregado. **OOB previsto con Digi Connect IT 4**. |
| **Sitio C — AS65003** | Idéntico al Sitio A. Perfil **CANDIDATE / CHAOS**: containers, config y firmware candidatos; rotación de perfiles A/B/C. |

---

## Console server OOB — dos opciones intercambiables

Ambos hacen el mismo trabajo en el diagrama (consola serial al IX40 + 5G independiente + energía propia). Se diferencian sobre todo en el plano de gestión:

| | **Opengear OM1304-4E-C-LSP** | **Digi Connect IT 4** |
|---|---|---|
| Serial al IX40 | DB9 ↔ RJ45, adaptador **319015** (pinout Opengear X2) | RJ45 RS-232, cable con pinout Digi/Cisco |
| Celular | 5G Carrier B | LTE Cat 4 Carrier B (módem Digi CORE) |
| Puertos de gestión | Switch de gestión de 4 puertos | 2× Ethernet 10/100 (agregar un switch chico para 4 puertos) |
| Gestionado por | Opengear **Lighthouse** (requiere la VM de Fase 2) | **Digi Remote Manager** (mismo tenant que los routers → un solo plano) |

> **En evaluación: Connect IT 4 en el Sitio B.** Elegirlo unifica la gestión bajo DRM y elimina la VM de Lighthouse; mantener el OM1304 conserva un OOB con diversidad de proveedor. Se listan los dos a propósito — por ahora el diagrama queda sin modificar.

## Leyenda de enlaces (tal como está dibujado)

- **WAN primaria Starlink / overlay + eBGP** — azul grueso
- **Respaldo in-band 5G / enlaces entre sitios** — azul
- **Service fabric (fibra SFP 1 Gb/s)** — verde
- **Gestión / LAN (HTTPS · API · DRM)** — gris punteado
- **Out-of-band / elementos de Fase 2** — naranja discontinuo
- **Zigbee / RF celular** — onda violeta
- **Caja discontinua = Fase 2**

---

*Fuente: [`assets/lab-diagram-tabloid-EN.tex`](assets/lab-diagram-tabloid-EN.tex) y [`03-arquitectura.md`](03-arquitectura.md). Los logos de producto son propiedad de sus respectivos dueños.*
