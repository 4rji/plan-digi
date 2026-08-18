# 05 · Track embebido central — Sensor de sitio en ConnectCore MP255

Este es el track que **elevé de opcional a central** (viene del proyecto **D2** de `plan1`). Es "el lado software" de Starlink (*Software Engineer, Ground Stations*) y el diferenciador más fuerte del capstone: casi nadie que aplica a redes sabe hacer Yocto de verdad.

**Misión:** construir una imagen Linux personalizada con **Yocto** sobre un SOM **ConnectCore MP255** que actúe como sensor de sitio: vigila la red local + condiciones ambientales/energía, corre detección ligera de anomalías, y transmite al SIEM con **envío resiliente** (buffer offline). **Secure boot (TrustFence) y OTA se comprometen para una fase posterior, no para el sprint de 2 semanas** (`trustfence close` es irreversible y de tiempo impredecible). Cuando se implemente, el OTA se hace por el **mismo Digi Remote Manager** que gestiona los routers.

> **Verificado por web (jul 2026)** contra la documentación oficial de Digi. Aun así, los detalles exactos de recetas, versiones de capa y comportamiento de `trustfence close` **se validan contra el firmware/DEY exactos del kit** antes de depender de ellos (ver riesgos R-E* en [`07`](07-riesgos-y-verificaciones.md)).

> **Alcance del sprint (Anillo 0):** imagen Yocto que arranca + receta con el agente + envío resiliente con buffer/replay. **Secure boot y OTA son fase posterior (Anillo 2)**; se documentan completos más abajo para tenerlos listos cuando se ejecuten.

---

## 0. Por qué mapea a Starlink (y por qué al manager le sirve)

- **Starlink** pide "escribir software Linux de alta calidad para procesadores comunes (ARM, PowerPC, x86)" y "diseñar redes tolerantes a fallos que operen largos periodos con mantenimiento mínimo". Un sensor desatendido en ARM, con secure boot, buffer offline y OTA seguro, **es** ese enunciado.
- **Digi** obtiene una demostración real de ConnectCore + DEY + TrustFence + ConnectCore Cloud Services integrada con DRM — material de enablement embebido que normalmente no existe en un lab de redes.

---

## 1. Toolchain: Digi Embedded Yocto (DEY)

**Plataforma:** ConnectCore MP255 → identificador de plataforma **`ccmp25`**, rama **DEY 4.0**. Capas Yocto de Digi: **`meta-digi`**.

**Flujo de build (verificado):**

1. Preparar el host de build (Linux, ~100+ GB libres, buena CPU/RAM; el **primer** build tarda **horas**). Puede ser una VM en Proxmox.
2. Obtener las capas de Digi (`repo` / `git clone` de `meta-digi` y dependencias).
3. Crear el proyecto específico de plataforma con **`mkproject.sh`** (para `ccmp25`).
4. Compilar la imagen con **`bitbake <image-recipe>`** desde el directorio del proyecto.
5. Los artefactos quedan en **`<project>/tmp/deploy/images/<platform>`**.

> Nota: por defecto la imagen DEY de ConnectCore MP25 incluye el backend de escritorio Wayland. Para un sensor headless conviene una imagen mínima/custom sin escritorio (menos superficie, arranque más rápido).

**Fuente:** [DEY 4.0 · ConnectCore MP25 · Create and build projects](https://docs.digi.com/resources/documentation/digidocs/embedded/dey/4.0/ccmp25/yocto-create-build-projects_t.html) · [github.com/digi-embedded/meta-digi](https://github.com/digi-embedded/meta-digi).

### Fases del build

| Fase | Qué se hace | "Done" |
|---|---|---|
| 1 | Build **mínimo** que arranque en el MP255 (headless). | La imagen bootea; consola y red del SOM OK. Imagen base reproducible guardada. |
| 2 | **Capa y receta custom** (`meta-<lab>`) que empaqueta el agente. | `bitbake` incluye el agente; rebuild incremental rápido. |
| 3 | Agente recolecta y envía al SIEM. | Datos llegan al SIEM con reloj NTP común e ID por evento. |
| 4 | **Buffer offline + reintento**. | Corte de red → sigue recolectando → reconecta → reenvía sin pérdida. |
| 5 · *fase posterior* | **Secure boot (TrustFence)**. | El dispositivo solo arranca imágenes firmadas; rootfs cifrado. **Fuera del sprint** (irreversible). |
| 6 · *fase posterior* | **OTA** por CCCS. | Actualización remota `.swu` sin ladrillar. |

---

## 2. El agente

**Objetivo:** liviano y que no se caiga en semanas. Python es el camino rápido (hay soporte y librerías de Digi); Rust es el stretch si se quiere un binario aún más robusto y ligero.

**Qué recolecta:**
- **Red local del sitio:** disponibilidad de gateway/LAN, latencia interna, presencia de hosts clave, estado de la interfaz.
- **Ambientales/energía:** temperatura, alimentación/estado de puerta o entradas digitales (complementa lo que ya da el XBee del sitio; el MP255 puede tener sus propios sensores/entradas del kit).
- **Salud del propio SOM:** CPU/RAM/temperatura.

**Detección ligera de anomalías:** empezar simple y honesto — umbrales + estadística por ventana (media/desviación, tasa de cambio), no un modelo pesado. El valor es *demostrar el patrón* (detección en el edge, sin depender del enlace), no la sofisticación del modelo. Marcar como anomalía y anotar el evento con su ID.

**Envío al SIEM (resiliente):**
- Formato estructurado (JSON), reloj **NTP común** con el resto del lab, **ID único por evento** e **ID por inyección de fallo** para poder correlacionar en el SIEM con las alarmas de DRM.
- **Buffer offline:** cuando el enlace cae, escribe a una cola local persistente (en `/mnt/data/private` si se usa el rootfs cifrado). Al reconectar, drena la cola en orden y **sin duplicar ni perder** (idempotencia por ID).

---

## 3. Secure boot y cifrado (TrustFence) — verificado

> **Alcance:** este bloque es **fase posterior (Anillo 2)**, no parte del sprint de 2 semanas. Se documenta completo aquí para que el procedimiento esté listo cuando se ejecute.

TrustFence es la marca de Digi para la cadena de confianza del ConnectCore.

**Secure boot:**
- Al habilitar TrustFence, el entorno de U-Boot se cifra por defecto usando el **CAAM OTPMK** y la clave única interna segura.
- Tras cerrar el dispositivo con **`trustfence close`** en U-Boot, el equipo **solo arranca imágenes correctamente firmadas**.
- TrustFence produce **U-Boot y kernel firmados y cifrados** específicos por variante. Un fichero **`SRK_efuses.bin`** contiene el hash de las claves públicas SRK y se necesita al preparar el dispositivo para secure boot.

**Cifrado de sistema de archivos:**
- Se habilita en el kernel con **`CONFIG_FS_ENCRYPTION`**, activo por defecto cuando TrustFence está habilitado (fscrypt).
- Los datos se guardan cifrados en el medio y se descifran en boot para que las apps accedan; ruta por defecto de datos cifrados: **`/mnt/data/private`**.

> **Advertencia operativa (crítica):** `trustfence close` quema fuses y es **irreversible** — un dispositivo mal firmado o con las claves perdidas queda inservible. Por eso se saca del sprint y va en **fase posterior (F2-C)** con la imagen ya estable, tras practicar el flujo de firma, y sobre una unidad que se puede sacrificar si el kit lo permite. Guardar las claves SRK en el secret store, nunca en el repo.

**Fuentes:** [DEY · Set up secure boot (TrustFence)](https://docs.digi.com/resources/documentation/digidocs/embedded/dey/3.0/cc8x/yocto-trustfence_t_secure-boot-set-up) · [DEY · File-system-level encryption (fscrypt)](https://docs.digi.com/resources/documentation/digidocs/embedded/dey/5.0/cc91/yocto-trustfence-file-system-encryption_c_fscrypt.html).

---

## 4. OTA por ConnectCore Cloud Services (CCCS) — verificado

Esto es lo que **une el track embebido al resto del lab**: el OTA del SOM usa el **mismo Digi Remote Manager** que gestiona los routers.

- **ConnectCore Cloud Services** permite programar firmware nuevo remotamente usando las **APIs de Remote Manager**. Los OEM gestionan OTA continuo de sus productos con CCCS.
- El paquete de actualización es **`.swu`** (SWUpdate); se construye con el flujo "Build a software update package" y se **almacena en el firmware repository de Remote Manager**.
- Se puede usar la librería Python **`python-devicecloud`** para subir paquetes al repositorio y programar dispositivos.
- La versión de firmware reportada y los ajustes de actualización dependen de la configuración de CCCS en **`/etc/cccs.conf`** del dispositivo.

**En el lab:**
- **Sprint (stretch):** construir un `.swu`, subirlo al firmware repo del tenant DRM, disparar una actualización de prueba en el MP255.
- **Anillo 2:** OTA robusto (A/B o con rollback) que no ladrille ante corte de energía a mitad de actualización — el problema real de una flota desatendida.

**Fuentes:** [DEY · Update device firmware (CCCS)](https://docs.digi.com/resources/documentation/digidocs/embedded/dey/4.0/ccmp15/yocto-cccs-web-develop-update-device-firmware_t.html) · [Digi · Reliable & secure OTA firmware updates](https://www.digi.com/resources/videos/reliable-secure-ota-firmware-updates).

---

## 5. Alternativa sin hardware (por si el kit tarda)

plan1 lo dice: se puede hacer **~80 % con QEMU** emulando ARM + una Raspberry Pi como sustituto de campo para la parte física de sensores; el aprendizaje de Yocto es idéntico. Estrategia:

- El **build de DEY y el aprendizaje de Yocto/recetas** no requieren el SOM: se hacen en el host de build desde el Día 1.
- Sin MP255 físico, se valida el arranque en QEMU y se difiere secure boot (TrustFence necesita el hardware real y sus fuses).
- Cuando llega el `CC-WMP255-KIT`, se flashea la imagen ya construida; TrustFence + OTA quedan para la fase posterior.

Esto mantiene el track embebido **en el camino central sin depender de que el hardware llegue el Día 1**.

---

## 6. Entregable "wow" del track

> El MP255 arranca **tu** imagen Yocto custom con **tu agente**. En la demo le cortas la red: **sigue recolectando**; al volver, **reenvía todo sin perder un dato**, y el SIEM muestra la serie completa correlacionada con el ID del fallo. *(Fase posterior: secure boot solo-firmado y OTA `.swu` por DRM.)*

---

## 7. Checklist del track

- [ ] Host de build DEY listo (Día 1).
- [ ] `meta-digi` obtenido; `mkproject.sh` para `ccmp25`; primer `bitbake` corriendo (Día 1).
- [ ] Imagen mínima arranca en MP255 (Día 2).
- [ ] Receta custom empaqueta el agente (Día 3).
- [ ] Agente recolecta y envía al SIEM con ID por evento (Día 4).
- [ ] Buffer offline + reenvío sin pérdida (Día 5).
- [ ] *(Fase posterior)* TrustFence: claves, `SRK_efuses.bin`, firma, `trustfence close`, rootfs cifrado.
- [ ] *(Fase posterior)* OTA `.swu` por CCCS/firmware repo de DRM.
- [ ] (Anillo 2) OTA robusto con rollback; carril OEM MP255 en Site C.

---

## 8. Riesgos específicos (detalle en [`07`](07-riesgos-y-verificaciones.md))

| ID | Riesgo | Mitigación |
|---|---|---|
| R-E1 | Primer `bitbake` tarda horas / dependencias que se rompen. | Arrancar el Día 1; host con recursos; fijar versiones de capa. |
| R-E2 | `trustfence close` es irreversible → brick. | Se ejecuta en fase posterior (no en el sprint); practicar firma antes; imagen estable; guardar claves; unidad sacrificable. |
| R-E3 | Versiones de receta/DEY y comportamiento exacto varían por SOM/firmware. | Validar contra el kit real; congelar DEY 4.0/`ccmp25` y las revisiones. |
| R-E4 | Kit MP255 tarda en llegar. | Ruta QEMU + Raspberry Pi (§5) para no bloquear el track. |
| R-E5 | CCCS/OTA requiere entitlements del tenant. | Confirmar en el license audit (G8) que el tenant permite subir `.swu` al firmware repo. |
