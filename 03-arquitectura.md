# 03 · Arquitectura — Estructura de cómo quedaría

Este documento describe **cómo se ve el laboratorio montado**: topología, cableado y separación por sitio, direccionamiento y plan de AS, protocolos, y flujo de datos. Es la referencia visual y de contratos; los pasos para llegar aquí están en [`04-pasos-montaje.md`](04-pasos-montaje.md).

> Convención: un sitio en el sprint (Anillo 0), tres sitios idénticos en la expansión (Anillo 1/2). Todo el hardware base es idéntico entre sitios; solo cambia el **perfil** (native / bonding / candidate) que se rota entre unidades.

---

## 1. Vista de conjunto (dos planos + control central)

```
                         ┌───────────────────────────────────────────────┐
                         │            CONTROL CENTRAL (nube + host)         │
                         │                                                 │
   Internet ────────────┤  • Digi Remote Manager Premier (cloud)          │
                         │  • Core / route-collector  AS65000              │
                         │  • Terminación de overlay (IKEv2/IPsec)         │
                         │  • SIEM / receptor de Push Monitor (webhook)    │
                         │  • Plataforma de impairment (loss/jitter/lat.)  │
                         │  • Servidor externo Digi WAN Bonding (Tier 3)   │
                         │  • [Fase 2] Lighthouse Enhance VM               │
                         └───────────────▲─────────────────────▲───────────┘
                                         │ overlay + BGP        │ gestión (HTTPS/API)
        ┌────────────────────────────────┼──────────────────────┼───────────────────┐
        │                                 │                      │                   │
 ┌──────┴───────┐                 ┌───────┴──────┐        ┌───────┴──────┐
 │  SITIO A     │                 │  SITIO B     │        │  SITIO C     │
 │  AS65001     │                 │  AS65002     │        │  AS65003     │
 │  perfil:     │                 │  perfil:     │        │  perfil:     │
 │  native      │                 │  bonding     │        │  candidate   │
 └──────────────┘                 └──────────────┘        └──────────────┘
   (Anillo 0: solo Sitio A en el sprint; B y C en expansión)
```

Dos planos deliberadamente separados:

- **Plano in-band (producción):** Starlink + 5G del IX40 → overlay → BGP → servicio. Es el que falla en las pruebas.
- **Plano out-of-band (rescate, fase 2):** Opengear OM1304 con **5G de OTRO carrier y OTRA energía** → Lighthouse. Debe seguir accesible cuando el in-band cae. Si compartieran energía o carrier, el supuesto de independencia sería falso.

---

## 2. Anatomía de un sitio

```
                         ┌───────────── SITIO (AS6500x) ─────────────────────┐
                         │                                                    │
   Starlink Performance  │   Starlink dishy ──(WAN primaria)──► IX40 WAN/ETH1 │
   Kit (Gen 3)           │                                       (cobre)      │
                         │                                                    │
   Carrier A (5G) ───────┼───► IX40 módem 5G ──(WAN backup in-band)──►        │
                         │                                                    │
                         │   IX40-05 ──SFP/ETH5 (1 Gb/s, SFP NO SFP+)──►      │
                         │      │        fabric de servicio (red anunciada    │
                         │      │        por BGP; IPv4 /24 + IPv6 /64)        │
                         │      │                                             │
                         │      ├── mgmt Ethernet ──► switch de gestión       │
                         │      │                                             │
                         │      ├── Digi Container (collector de métricas,    │
                         │      │     límites de recursos)                    │
                         │      │                                             │
                         │      └── DB9 serial ──► [Fase 2] OM1304 (modo Login)│
                         │                                                    │
                         │   XBee Hive ──► LAN sitio ; XBee 3 Zigbee ──► sensor│
                         │      (temperatura / puerta / entrada digital)      │
                         │                                                    │
                         │   ConnectCore MP255 (sensor embebido)              │
                         │      • imagen DEY (Yocto); secure boot: fase 2     │
                         │      • agente → SIEM (buffer offline + replay)     │
                         │      • OTA vía ConnectCore Cloud Services (DRM)    │
                         │                                                    │
                         │   [Fase 2] OM1304-4E-C-LSP (OOB):                  │
                         │      • 5G Carrier B (independiente)                │
                         │      • switch de gestión 4 puertos                 │
                         │      • consola serial al IX40                      │
                         │                                                    │
                         │   Energía: PDU/UPS primaria (Starlink, IX40, carga)│
                         │            PDU/UPS secundaria (OOB + su módem)     │
                         └────────────────────────────────────────────────────┘
```

---

## 3. Cableado y separación (paso a paso, por sitio)

1. **Starlink Performance Kit → IX40 `WAN/ETH1` (cobre).** `SFP/ETH1` queda **vacío**: al poblarlo se desactiva el puerto de cobre (son mutuamente excluyentes). Ver G6 para el SFP validado del segundo socket.
2. **IX40 módem 5G → Carrier A.** WAN de respaldo in-band, con SIM/APN propio.
3. **IX40 `SFP/ETH5` → fabric/carga de gateway (fibra 1 Gb/s).** Aquí vive la red de servicio que el sitio anuncia por BGP. **Solo SFP, nunca SFP+** (límite deliberado del IX40). `SFP/ETH5` **sí** es independiente del par cobre/SFP de ETH1.
4. **[Fase 2] OM1304 5G → Carrier B**, distinto del IX40, con antenas y alimentación separadas.
5. **[Fase 2] OM1304 serial → DB9 del IX40.** El puerto IX40 en modo `Login`; el IX40 es DTE RS-232 y el adaptador Opengear para X2/DTE es **`319015`** (DB9F–RJ45 crossover). Validar baud rate, 8N1 y flow control en banco.
6. **[Fase 2] OM1304 Ethernet management switch → management del IX40, Hive y PDU.** Este enlace **no** transporta producción.
7. **Hive → LAN del sitio; XBee 3 Zigbee → sensor físico.** El sensor mide temperatura, estado de puerta/alimentación o una entrada digital; **no** forma parte del forwarding crítico.
8. **ConnectCore MP255 → LAN de gestión del sitio** (Ethernet). Recolecta red local + ambientales y envía al SIEM; recibe OTA por CCCS.
9. **Energía:** Starlink, IX40 y carga en PDU/UPS **primaria**; Opengear y su módem en circuito/UPS **secundario**.

---

## 4. Direccionamiento y plan de routing

### 4.1 Underlay (medido, nunca asumido)

| Underlay | Detalle | A medir/registrar |
|---|---|---|
| Starlink | DHCPv4; donde se entregue, SLAAC (`/64`) + DHCPv6-PD (`/56`). CGNAT `100.64.0.0/10` por defecto; IPv4 pública opcional en planes Priority. | CGNAT vs pública, IPv6-PD observado, MTU, latencia, jitter (G9). |
| 5G IX40 (Carrier A) | SIM/APN propio. | IPv4/IPv6, MTU, NAT, soporte 5G SA/APN (G10). |
| [Fase 2] 5G OM1304 (Carrier B) | Otro operador, otro plano de energía. | Independencia real respecto a Starlink y Carrier A. |

### 4.2 Overlay y BGP

```
        AS65000  (core / route-collector, endpoint público del overlay)
           ▲          ▲          ▲
   eBGP    │          │          │   eBGP (sobre overlay IKEv2/IPsec)
           │          │          │
      AS65001     AS65002     AS65003
      Sitio A     Sitio B     Sitio C
      /24 v4      /24 v4      /24 v4      ← cada sitio origina 1 red IPv4 RFC1918
      /64 v6      /64 v6      /64 v6      ← y 1 IPv6 de documentación (2001:db8::/32)
```

- **Overlay** iniciado por el sitio hacia un endpoint central público. Baseline: **IKEv2/IPsec route-based con NAT traversal** (obligatorio por el CGNAT de Starlink). DMVPN es variante documentada por DAL, pero no baseline hasta probarla tras CGNAT.
- **eBGP** sobre el overlay entre cada IX40 y el core/collector.
- **Políticas:** prefix-lists de entrada/salida, límite máximo de prefijos, prohibición de anunciar default salvo prueba explícita, sin redistribución automática entre BGP y rutas de administración.
- **IPv6 de documentación** `2001:db8::/32`: jamás se anuncia a Internet.
- El core AS65000 es **estándar y vendor-neutral** porque Digi no vende un router de backbone carrier.
- **Pregunta abierta que el lab mide, no declara:** si el cambio Starlink→5G mantiene el túnel/BGP o provoca reconvergencia.

### 4.3 Failover nativo (SureLink) — árbol DAL

```
network interface {ETH1|Modem}
  ├─ ipv4 metric      (ETH1=1 menor → mayor prioridad ; Modem=3)
  ├─ ipv4/ipv6 weight (métricas iguales + pesos → ECMP; NO es bonding)
  └─ surelink
       ├─ enable true
       ├─ tests   → interface_up · ping(gateway) · ping(externo)
       │            · dns · http/https · variantes IPv6 · custom
       └─ actions → subir métrica · reiniciar interfaz · reset módem
                    · cambiar SIM · reboot · custom · power-cycle módem
```

Reglas de diseño: **múltiples destinos** (evitar que el monitor sea un nuevo punto único de fallo), **acciones escalonadas** (primero métrica/reinicio; reset de módem/cambio de SIM solo tras fallos persistentes), **histéresis + hold-down** contra flapping, y **failback automático a Starlink solo tras una ventana estable**.

### 4.4 WAN Bonding (experimento A/B)

- UI `Network > SD-WAN > WAN bonding`; CLI `network sdwan wan_bonding`; estado `show wan-bonding`.
- Requiere licencia habilitada en DRM + **servidor externo/VPS** de bonding + credenciales de túnel.
- Interfaces `ETH1` y `Modem` dentro del túnel; cifrado habilitado.
- Tier 3 de **1 Gb/s** para que la licencia no sea el cuello de botella.
- Perfiles a comparar si la versión los expone: Automatic, Low Latency, Cellular Optimized.
- **Es una prueba separada** del failover por métrica/SureLink, con el **mismo** impairment trace.

---

## 5. Plano de gestión unificado (el detalle que sorprende)

```
                 ┌──────────────── Digi Remote Manager Premier ───────────────┐
                 │                                                             │
   IX40 (router) │  • Configuration Manager (golden config + settings/sitio)  │
     ───────────►│  • Alerts (Device Offline, noncompliance)                   │
                 │  • Push Monitor (webhook http, topic alert_status) → SIEM   │
   XBee Hive     │  • Data streams / DataPoints (telemetría RF)                │
     ───────────►│  • Digi Containers (despliegue del collector al IX40)       │
                 │                                                             │
   ConnectCore   │  • ConnectCore Cloud Services (CCCS):                        │
     MP255       │      OTA .swu ← firmware repository de Remote Manager        │
     ───────────►│      (mismo tenant que gestiona los routers)                │
                 └─────────────────────────────────────────────────────────────┘
```

El punto clave para el manager y para Starlink: **un solo plano de gestión** (DRM) opera routers, containers, telemetría RF **y** el OTA del sensor embebido. No hay tres consolas distintas.

### API — contratos base (validados; detalle en pasos)

| Sistema | Base | Auth |
|---|---|---|
| DRM | `https://remotemanager.digi.com` · REST v1 `/ws/v1/*` (heredado XML/SCI `/ws/*` solo si no hay v1) | Basic, o API key con cabeceras `X-API-KEY-ID` / `X-API-KEY-SECRET` (creadas con `POST /ws/v1/api_keys/inventory`). |
| Lighthouse (fase 2) | `https://{host}/api/v3.7` | `POST /sessions` → `session`; luego cabecera `Authorization: Token {session}` (no `Bearer`). |

---

## 6. Consolidación de funciones (qué corre dónde y por qué)

| Función | Dónde corre | Ventaja | Riesgo / mitigación |
|---|---|---|---|
| Collector de métricas de red | **Digi Container en el IX40** | Menos hardware; despliegue central desde DRM; demuestra Containers. | Comparte CPU/RAM con el router → **límites de recursos + prueba de carga** para probar que no degrada el forwarding. |
| Adquisición RF / lógica de sensores | **XBee Hive** | Aísla la app del forwarding; añade radio XBee. | Otro equipo/licencia/superficie; aceptable por el aislamiento. |
| Sensor embebido / detección de anomalías | **ConnectCore MP255** | Carril de software embebido (Yocto en Ring 0; secure boot/OTA en fase 2); diferenciador Starlink. | Es el track más difícil; ver [`05-embebido-connectcore.md`](05-embebido-connectcore.md) y riesgos R-E*. |
| OOB / rescate | **Opengear OM1304 (fase 2)** | Independiente en carrier y energía. | Se mantiene **austero** a propósito: el equipo que rescata la red no debe cargar experimentos. |

---

## 7. Topología de tres sitios y rotación de perfiles (expansión)

| Zona | Componentes | Función / perfil |
|---|---|---|
| Control central | DRM Premier, core/collector AS65000, SIEM, impairment, servidor WAN Bonding, [fase 2] Lighthouse Enhance | Gestión de ambos planos, terminación de overlay, telemetría, alarmas, pruebas. |
| Sitio A — AS65001 | Starlink → IX40+5G; [fase 2] OM1304; Hive+XBee; MP255; carga tras SFP | Perfil **native** (control): failover por métricas + SureLink. |
| Sitio B — AS65002 | Igual que A | Perfil **bonding**: Starlink y 5G dentro del túnel agregado. **OOB con Digi Connect IT 4** (en lugar del OM1304). |
| Sitio C — AS65003 | Igual que A; MP255 como carril OEM opcional | Perfil **candidate/chaos**: Containers, config nueva, firmware candidato. |

Tras la primera campaña, los **perfiles A/B/C se rotan entre las tres unidades físicas** para eliminar el sesgo de una unidad, un terminal Starlink o una celda concreta.

---

## 8. Resumen de contratos de direccionamiento (para no improvisar)

| Elemento | Valor | Nota |
|---|---|---|
| AS central | **65000** | Core/collector vendor-neutral. |
| AS sitios | **65001 / 65002 / 65003** | Uno por sitio. |
| Red IPv4 por sitio | RFC1918 `/24` | Detrás de `SFP/ETH5`. |
| Red IPv6 por sitio | Documentación `/64` dentro de `2001:db8::/32` | Nunca anunciada a Internet. |
| Overlay | IKEv2/IPsec route-based + NAT-T | Baseline; DMVPN a evaluar. |
| Starlink IPv4 | CGNAT `100.64.0.0/10` (pública opcional) | Registrar el observado (G9). |
| SFP | Solo **1 Gb/s SFP** | **Prohibido SFP+**; P/N validado en G6. |

Todo lo anterior se materializa siguiendo [`04-pasos-montaje.md`](04-pasos-montaje.md). Los supuestos marcados (G*) se cierran según [`07-riesgos-y-verificaciones.md`](07-riesgos-y-verificaciones.md) **antes** de depender de ellos.
