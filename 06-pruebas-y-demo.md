# 06 · Pruebas y demo

Qué se inyecta, qué debe demostrar, cómo se mide, cuándo se considera aceptable y cómo se presenta. La disciplina central: **se mide, no se declara**; cada inyección de fallo lleva un **ID único** para correlacionar en el SIEM la telemetría del router, la alarma de DRM y los eventos del sensor embebido.

---

## 1. Telemetría y observabilidad (qué se captura siempre)

Sin esto, ninguna prueba significa nada.

- **Estado in-band:** Starlink/`ETH1`, módem 5G, SureLink, rutas, BGP, túnel, CPU/RAM/temperatura del IX40, señal celular.
- **Métricas de resiliencia:** RTT, jitter, pérdida, throughput, **tiempo de detección**, **tiempo de conmutación**, **tiempo de recuperación**, **failback**.
- **Embebido:** series del agente (red local + ambientales + salud SOM), estado de buffer/replay.
- **RF:** sensor XBee por sitio y lag extremo-a-extremo hasta DRM.
- **OOB (fase 2):** estado Opengear/Lighthouse y disponibilidad de consola.
- **Higiene de datos:** reloj **NTP común**, **ID único por inyección**, datos brutos conservados junto con resumen p50/p95/p99 y **versión de firmware/config**.

---

## 2. Matriz de fallos

Columna **Anillo** indica dónde se ejecuta: **0** = sprint (subconjunto), **2** = programa completo (30 reps).

| # | Fallo inyectado | Qué debe demostrar | Anillo |
|---|---|---|---|
| F1 | Desconectar físicamente Starlink | Detección de link, cambio de ruta, alerta DRM, continuidad por 5G, (fase 2) acceso OOB. | **0** |
| F2 | Blackhole aguas arriba con Ethernet **up** | SureLink detecta un fallo real que la portadora física no ve. | **0** |
| F3 | Fallo **solo DNS** | El sistema distingue DNS de pérdida total y evita una acción desproporcionada. | **0** |
| F4 | Fallo **solo IPv6** | Observabilidad dual-stack y política clara de fallback IPv4. | 0/2 |
| F5 | Pérdida/jitter/latencia progresiva | Umbral antes de flapping; comparación **native vs Bonding**. | **0** |
| F6 | Caída del operador del IX40 / cambio de SIM | Recuperación WWAN, alerta e independencia respecto a Starlink. | 2 |
| F7 | Caída simultánea Starlink + 5G in-band | El servicio cae de forma esperada, pero (fase 2) Lighthouse/OOB sigue accesible. | 2 |
| F8 | Reboot / power loss del IX40 (PDU) | Tiempo de boot, restauración de túnel/BGP/DRM, (fase 2) acceso por consola. | 2 |
| F9 | **Drift de configuración** | Configuration Manager detecta y remedia dentro de la ventana. | **0** |
| F10 | **Receiver de webhook offline** | Push Monitor persistente almacena y reproduce sin perder IDs de evento. | **0** |
| F11 | Caída del servidor WAN Bonding | Se documenta la dependencia y el modo de recuperación. | 0/2 |
| F12 | Sobrecarga del container edge | Aislamiento suficiente o se mueve la carga a Hive. | 2 |
| F13 | **Corte de red al sensor embebido** | El agente sigue recolectando y reenvía sin pérdida al reconectar (buffer/replay). | **0** |
| F14 | (Anillo 2) OTA a mitad + corte de energía | El SOM no se ladrilla; rollback o A/B recupera. | 2 |

**Subconjunto del sprint (≈10 reps c/u):** F1, F2, F3, F5, F9, F10, F13 (+ F4/F11 si da tiempo).
**Programa completo (30 reps c/u, p50/p95/p99):** toda la matriz, con rotación de perfiles A/B/C.

---

## 3. Criterios de aceptación

Objetivos del laboratorio, **no** garantías de Digi, Opengear ni Starlink.

### 3.1 Del sprint (Anillo 0)

- Failover nativo (F1/F2) con objetivo de trabajo **p95 ≤ 30 s** tras afinar SureLink; si no se logra, **se publica la medición y la causa** (no se esconde).
- **Cero route leaks** y **100 % de prefijos** IPv4/IPv6 esperados en el collector.
- Bajo WAN Bonding (F5), la sesión de la app de prueba **no se resetea**; objetivo de pérdida en handoff **≤ 1 %**.
- Alarma/webhook **correlacionable con el ID de fallo**; replay comprobado tras indisponibilidad del receiver (F10).
- Drift **detectado y remediado** dentro de la ventana de scan (F9).
- Sensor embebido **sin pérdida de datos** ante corte (F13); buffer/replay medido.
- La imagen embebida (**Yocto/DEY**) **arranca** y el agente envía al SIEM. *(Secure boot: fase posterior.)*
- **Repetibilidad:** un segundo despliegue (sitio 2, físico o simulado) se levanta con `make deploy`, idempotente.

### 3.2 Del programa completo (Anillo 2)

- **30 repeticiones** por fallo principal y publicación de datos brutos.
- Consola OOB disponible en **30/30** pérdidas del plano in-band, salvo cuando se inyecte expresamente fallo de energía u operador OOB.
- Telemetría XBee **sin huecos no explicados**; cualquier buffering/replay medido.
- Repetibilidad total: **un segundo ingeniero** ejecuta el runbook sin conocimiento oral del autor.

---

## 4. Cómo se mide (método)

1. **Testigo de carga real:** ping continuo + una videollamada/stream que actúa de "canario". La demo muestra el tiempo **real**, no una animación.
2. **Marca temporal por inyección:** cada fallo se dispara con un ID; se anota T=0 y se derivan detección/conmutación/recuperación de las series.
3. **Mismo impairment trace** para native y Bonding (F5) — si no, la comparación no vale.
4. **Impairment reproducible:** perfiles de pérdida/jitter/latencia/blackhole/rate-limit guardados (p. ej. `tc/netem`) para repetir idéntico.
5. **Datos brutos + resumen:** se conserva lo crudo y el p50/p95/p99, con versión de firmware/config.

---

## 5. Guion de la demo (12 minutos)

Adaptado al sprint: si Opengear aún no llegó, el minuto 5–7 (OOB) se presenta como **grabación de fase 2** o se sustituye por el bloque embebido; el resto es en vivo.

1. **Min 0–2 — Estado sano.** Mostrar el sitio (o los 3 si hay hardware): rutas BGP IPv4/IPv6 en el collector, telemetría DRM, sensor XBee y el **MP255 con su imagen Yocto custom**. (Fase 2: encender un nodo `-LSP` para que su onboarding avance en paralelo.)
2. **Min 2–4 — Blackhole de Starlink sin bajar Ethernet (F2).** SureLink detecta, sube la métrica y mueve el tráfico a 5G. Mostrar el **tiempo exacto**.
3. **Min 4–5 — Continuidad + correlación.** El flujo de aplicación y la telemetría continúan; se correlaciona la alerta DRM y el evento recibido por **Push Monitor** con el ID del fallo.
4. **Min 5–7 — OOB (fase 2) o Embebido.**
   - *Fase 2:* abrir por Lighthouse la consola serial del IX40 con el in-band caído y revisar estado/rutas desde el OOB.
   - *Sprint:* cortar la red al **sensor embebido**, mostrar que sigue recolectando y que al reconectar **reenvía sin pérdida** (F13).
5. **Min 7–8 — Drift + remediación (F9).** Introducir un drift seguro; Configuration Manager lo detecta y remedia dentro de una maintenance window.
6. **Min 8–10 — Native vs WAN Bonding (F5).** Repetir el mismo fallo en el perfil Bonding y comparar en pantalla pérdida, continuidad de sesión, RTT y jitter contra el nativo.
7. **Min 10–11 — Restauración.** Restaurar Starlink, mostrar estabilidad previa al failback y la recuperación BGP/telemetría.
8. **Min 11–12 — Cierre.** (Fase 2: confirmar el onboarding LSP iniciado al principio, o grabación con timestamps.) Cerrar con el cuadro **native vs Bonding** y las **limitaciones conocidas** declaradas con honestidad.

### Versión de 90 segundos
El "money shot": tumbas Starlink (o la red del sensor), **no tocas nada**, el sistema se recupera/reenvía solo, y el dashboard del SIEM **narra el incidente en tiempo real** con el ID del fallo.

---

## 6. Qué impresiona a cada audiencia

**A un ingeniero Starlink:**
- Separación precisa de underlay, overlay y routing.
- BGP/IPv6 real con políticas, no solo conectividad "ping".
- Blackhole con link físico activo.
- Métricas de convergencia y pérdida **repetidas**, no una ejecución afortunada.
- (Fase 2) OOB independiente en carrier y energía.
- Software embebido en ARM con envío resiliente (buffer/replay); secure boot como fase posterior.
- Declaración explícita de que **no** se simula la red propietaria ni se afirma eBGP de cliente con Starlink.

**A un manager Digi:**
- Arquitectura de referencia vendible "Starlink primary + Digi 5G + Opengear OOB".
- Comparación cuantitativa SureLink/native vs WAN Bonding.
- Matriz de interoperabilidad por firmware, carrier, IPv4/IPv6, MTU y hardware Starlink.
- Demo reutilizable por preventa y enablement.
- Runbook de soporte y chaos testing.
- Hallazgos concretos para PM/documentación (G1–G12).
- Uso **significativo** de IX40, DRM, Containers, XBee Hive, ConnectCore/Yocto, WAN Bonding y (fase 2) Opengear/Lighthouse.

---

## 7. Artefactos (entregables)

- Diagrama **lógico** y **as-built**.
- BOM y matriz de compatibilidad validadas.
- Documento de intención de configuración por firmware.
- Contrato de API DRM/Lighthouse validado contra los tenants reales.
- Runbook de onboarding, failover, OOB, rollback y recuperación.
- Dataset de pruebas + informe **p50/p95/p99** (sprint: reps representativas; Anillo 2: 30 reps).
- Imagen DEY reproducible + receta del agente. *(Procedimiento TrustFence: entregable de fase posterior.)*
- Video de **12 min** y versión de **90 s**.
- **One-pager ejecutivo** para Digi ([`08`](08-executive-pitch-EN.md)).
- **README público sanitizado** para portafolio profesional.
- **Backlog de gaps y contradicciones** (G1–G12) con dueño recomendado en Digi/Opengear.
