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

## Checklist

- [ ] RHEL 10 instalado en `rhel10-server` (4 GB RAM, 2 vCPU, 40 GB disco)
- [ ] Reproducido el error de `dnf repolist` sin registrar
- [ ] Sistema registrado con `subscription-manager register`
- [ ] `subscription-manager status` → `Overall Status: Current`
- [ ] Entitlements confirmados con `list --available` / `list --consumed`
- [ ] `dnf repolist` muestra BaseOS y AppStream activos
- [ ] Evidencias curadas en `evidencias/`

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_
