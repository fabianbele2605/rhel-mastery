# Módulo 00 — Registro con subscription-manager, entitlements, instalación de RHEL 10

## Objetivo

Instalar RHEL 10 en `rhel10-server` (VirtualBox) desde la ISO oficial y dejar
el sistema **registrado y con entitlements activos**, de forma que `dnf`
pueda usar los repos BaseOS y AppStream. Sin esto, RHEL se comporta como una
distro "muda": booteable pero sin acceso a software más allá de lo que ya
trae la ISO.

## Teoría

### ¿Por qué RHEL necesita registro y Fedora no?

Fedora y CentOS Stream son upstream/downstream de RHEL pero **no requieren
cuenta**: sus repos son públicos y gratuitos sin autenticación. RHEL es un
producto comercial — Red Hat vende soporte y garantías sobre un ciclo de
vida de 10 años, y el mecanismo para controlar quién tiene derecho a esos
repos (y a qué versión) es la suscripción.

Esto no es solo burocracia: es la diferencia entre "descargué una ISO" y
"tengo acceso a paquetes parcheados, seguridad, y compatibilidad garantizada
durante una década". El detalle completo del ciclo de vida (EUS/ELS) se ve
en el módulo 01; acá nos concentramos en el mecanismo técnico del registro.

### Subscription-manager: las tres piezas

1. **Cuenta Red Hat** (la tuya, gratis vía Developer Subscription for
   Individuals) — identifica quién sos.
2. **Registro del sistema** (`subscription-manager register`) — vincula
   *esta VM en particular* a tu cuenta. Cada sistema registrado consume un
   "slot" de tu suscripción (la Developer da hasta 16).
3. **Entitlements** — certificados X.509 que `subscription-manager` instala
   en `/etc/pki/entitlement/` una vez registrado el sistema. Son los que le
   dicen a `dnf` qué repos puede ver. Sin entitlement válido, los repos de
   Red Hat están configurados pero inaccesibles (vas a ver el error
   exacto en la práctica).

Con RHEL 10, Red Hat simplificó esto con **Simple Content Access (SCA)**:
ya no hace falta "attachear" manualmente una suscripción específica
(`subscription-manager attach`) — al registrar la cuenta, si tiene SCA
habilitado (lo tiene por defecto en cuentas nuevas de Developer
Subscription), los entitlements quedan disponibles automáticamente. Esto
es un cambio real de comportamiento frente a versiones anteriores de RHEL
donde vas a ver documentación vieja que menciona `attach --auto`.

### AppStream vs BaseOS

- **BaseOS**: el núcleo mínimo del sistema operativo (kernel, systemd,
  utilidades base). Paquetes con ciclo de vida largo y estable.
- **AppStream**: aplicaciones, lenguajes, bases de datos — con streams de
  versiones que podés elegir (ej. distintas versiones de PostgreSQL)
  independientemente del ciclo de BaseOS.

Esta separación es específica de RHEL 8+ (reemplazó el modelo monolítico
de RHEL 7) y es clave para el examen: preguntas de gestión de software
asumen que sabés en qué repo vive cada cosa.

## Práctica

### 1. Instalación desde la ISO

Instalación estándar en VirtualBox: 4 GB RAM, 2 vCPU, 40 GB disco (según
la tabla del entorno del curso). Durante Anaconda podés registrar el
sistema ahí mismo si tenés las credenciales a mano, pero en este módulo lo
hacemos **después** de instalar, para ver el proceso paso a paso.

Configuración recomendada en Anaconda:
- Particionado automático (LVM) — el módulo 05 profundiza en LVM manual.
- Hostname: `rhel10-server.local` (coherente con la tabla del entorno).
- Usuario no-root creado con permisos sudo, root deshabilitado para login
  directo por SSH (buena práctica que vas a reforzar en el módulo 14).

### 2. Error intencional: `dnf` antes de registrar

Antes de registrar, corré:

```bash
sudo dnf repolist
```

Vas a ver algo como `This system is not registered with an entitlement
server` o una lista vacía de repos — los repos de RHEL vienen preconfigurados
en el `.repo` pero **no hay entitlement que los habilite**. Esto no es un
bug: es el comportamiento esperado, y vale la pena verlo con tus propios
ojos antes de registrar, para entender qué es lo que el registro realmente
resuelve.

### 3. Registro

```bash
sudo subscription-manager register --username TU_USUARIO --password 'TU_PASSWORD'
```

Si tu cuenta tiene 2FA activado, `subscription-manager` te va a pedir un
token o vas a necesitar generar una API key desde el portal — documentá acá
lo que realmente te pida (regla del curso: verdad sobre guión).

### 4. Verificación de status y entitlements

```bash
sudo subscription-manager status
sudo subscription-manager list --available
sudo subscription-manager list --consumed
```

`status` debe decir `Overall Status: Current`. Si tu cuenta usa SCA, es
esperable que `--available` y `--consumed` muestren la suscripción de
Developer Subscription sin que hayas corrido ningún `attach` manual.

### 5. Confirmar acceso real a los repos

```bash
sudo dnf repolist
sudo dnf repolist --all
```

Ahora sí deberían aparecer `rhel-10-for-x86_64-baseos-rpms` y
`rhel-10-for-x86_64-appstream-rpms` (los nombres exactos de los repos
quedan documentados acá una vez que los veas — pueden variar según arch y
canal).

## Hallazgos reales (verdad sobre guión)

Durante la práctica aparecieron varios comportamientos que la teoría no
anticipaba. Quedan documentados acá porque son tan valiosos como el
camino feliz:

1. **`subscription-manager status` ya no dice "Overall Status: Current"**.
   En esta versión (RHEL 10 con Simple Content Access) el output se
   simplificó a `Estatus general: Registrado` — el modelo viejo de
   "suscripciones actuales vs vencidas" ya no aplica de la misma forma
   porque con SCA no hay nada que "vencer" a nivel de attach individual.
2. **`subscription-manager list --available` y `--consumed` no existen
   más**. Son remanentes de documentación pre-SCA. Las únicas opciones
   reales de `list` en esta versión son `--installed` (default) y
   `--matches`.
3. **El controlador gráfico VMSVGA de VirtualBox genera warnings
   `vmwgfx *ERROR*` en el boot de RHEL 10**, pero son cosmético — no
   bloquean nada. Cambiar a `VBoxSVGA` pensando que "arreglaba" el
   warning en realidad **rompió el modo gráfico de Anaconda**
   (`Wayland startup failed, falling back to text mode`). Lección:
   `VMSVGA` es la opción correcta para este guest, a pesar del ruido en
   el log.
4. **Los `kernel-headers`/`kernel-devel` instalados desde los repos no
   coincidían con el kernel de la ISO** (`211.7.3` corriendo vs
   `211.61.1` en repos) — esto rompió la primera compilación de los
   módulos de Guest Additions (`Kernel headers not found for target
   kernel`). Se resolvió con `sudo dnf update kernel -y` + `reboot`, lo
   cual además disparó automáticamente el rebuild de los módulos de
   Guest Additions como hook post-instalación del paquete `kernel`.

## Checklist

- [x] RHEL 10 instalado en `rhel10-server` (4 GB RAM, 2 vCPU, ~41 GB disco)
- [x] Reproducido el error de `dnf repolist` sin registrar
- [x] Sistema registrado con `subscription-manager register`
- [x] `subscription-manager status` → Registrado (ver hallazgo #1 sobre el output real)
- [x] Entitlements confirmados (ver hallazgo #2 sobre las flags reales de `list`)
- [x] `dnf repolist` muestra BaseOS y AppStream activos
- [x] Evidencias curadas en `evidencias/`
- [x] (bonus, fuera del alcance original) Guest Additions instaladas y funcionando

## Evidencias

1. ![GRUB del instalador con las 4 opciones de boot](evidencias/01-grub-menu-instalador.png)
   Menú de GRUB del instalador de RHEL 10.2: instalación normal, test+install, FIPS, troubleshooting.

2. ![Pantalla de bienvenida de Anaconda en español](evidencias/02-anaconda-bienvenida-idioma.png)
   Primer arranque exitoso de Anaconda en modo gráfico, con VMSVGA, selección de idioma español (Colombia).

3. ![Hallazgo: VBoxSVGA rompe Wayland](evidencias/03-hallazgo-vboxsvga-wayland-fallo.png)
   Al cambiar el controlador gráfico a VBoxSVGA, Anaconda no pudo iniciar Wayland y cayó a modo texto/RDP — evidencia del hallazgo #3.

4. ![Boot con VMSVGA revertido, warnings cosméticos](evidencias/04-boot-vmsvga-warnings-cosmeticos.png)
   Con VMSVGA de nuevo, reaparecen los warnings `vmwgfx`/`e1000` pero el boot continúa sin problema — confirma que son ruido, no errores reales.

5. ![Destino de instalación confirmado](evidencias/05-destino-instalacion-confirmado.png)
   Disco de 41.28 GiB seleccionado, particionado automático, sin cifrado.

6. ![Creación de usuario fbeleno](evidencias/06-creacion-usuario-fbeleno.png)
   Usuario administrador `fbeleno` creado con membresía al grupo `wheel` (sudo), contraseña fuerte.

7. ![Hostname aplicado](evidencias/07-hostname-rhel10-server-local.png)
   Nombre de equipo configurado como `rhel10-server.local` desde la pantalla de Red y nombre de equipo.

8. ![Instalación completada](evidencias/08-instalacion-completada.png)
   Mensaje "Red Hat Enterprise Linux se ha logrado instalar y está preparado para su uso."

9. ![Error de dnf sin registrar](evidencias/09-error-dnf-repolist-sin-registrar.png)
   `sudo dnf repolist` antes de registrar: "No se pudo leer identidad del consumidor" / "No hay ningún repositorio disponible" — el error intencional central del módulo.

10. ![Registro exitoso](evidencias/10-subscription-manager-register-exitoso.png)
    `subscription-manager register` exitoso, con notificación nativa de GNOME confirmando "Registration Successful".

11. ![Status registrado y flags obsoletas](evidencias/11-status-registrado-y-opciones-list-obsoletas.png)
    `subscription-manager status` → "Estatus general: Registrado", y el error real al usar `--available`/`--consumed` (hallazgos #1 y #2).

12. ![dnf repolist con BaseOS y AppStream](evidencias/12-dnf-repolist-baseos-appstream-activos.png)
    Objetivo del módulo cumplido: `subscription-manager list --installed` confirma el producto instalado, y `dnf repolist` muestra BaseOS y AppStream activos.

13. ![Hallazgo: kernel headers desfasados](evidencias/13-hallazgo-kernel-headers-desfasados.png)
    `uname -r` vs `rpm -q kernel kernel-devel kernel-headers` mostrando el desfase de versión que rompió la primera compilación de Guest Additions (hallazgo #4).

14. ![Guest Additions funcionando](evidencias/14-guest-additions-vboxguest-cargado.png)
    `lsmod | grep vbox` confirma el módulo `vboxguest` cargado tras actualizar el kernel y reiniciar — resolución ajustada automáticamente.
