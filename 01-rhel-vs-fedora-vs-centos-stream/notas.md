# Módulo 01 — Flujo Fedora → CentOS Stream → RHEL, ciclo de vida de 10 años (EUS/ELS)

## Objetivo

Entender el flujo real de desarrollo upstream/downstream de RHEL y el
mecanismo de ciclo de vida de 10 años, incluyendo EUS y ELS — no como
trivia histórica, sino como la base de decisiones reales de sysadmin
(cuándo actualizar, cuándo clavar una versión) y de preguntas directas
del EX200.

## Teoría

### El flujo real: Fedora → CentOS Stream → RHEL

No es una cadena lineal de "derivar" en el sentido clásico. Desde RHEL 8
en adelante:

1. **Fedora** es el laboratorio de features nuevas, ciclo de vida corto
   (~13 meses por versión). Acá se prueba todo lo que eventualmente puede
   llegar a RHEL, pero sin garantía ni orden fijo.
2. **CentOS Stream** es una rama de desarrollo continuo (rolling dentro
   de una major) que corre **ligeramente adelante** de la próxima minor
   release de RHEL. Ya no es "RHEL gratis reconstruido desde el código
   fuente publicado" (el modelo viejo de CentOS Linux, descontinuado) —
   es donde Red Hat y la comunidad desarrollan en público lo que va a
   convertirse en la próxima RHEL X.Y.
3. **RHEL** congela una foto de CentOS Stream en cada minor release y la
   convierte en un producto estable, soportado y con garantías
   contractuales.

Esto explica por qué tu sistema real se autodeclara `ID_LIKE="centos
fedora"` en `/etc/os-release` — es literal: RHEL desciende de ambos en
el linaje de desarrollo, no es marketing.

### Ciclo de vida de 10 años

Cada **major version** de RHEL (ej. RHEL 10) tiene tres fases:

- **Full Support** (~5 años): actualizaciones de funcionalidad y
  seguridad normales, nuevas minor releases (10.0 → 10.1 → 10.2...).
- **Maintenance Support** (~5 años más): solo parches críticos de
  seguridad y bugs graves, sin features nuevas.
- Después de los 10 años totales, el sistema queda sin soporte salvo que
  se pague **ELS**.

### EUS vs ELS — la diferencia que importa

- **EUS (Extended Update Support)**: te permite quedarte clavado en una
  **minor version específica** (ej. RHEL 9.2) recibiendo parches de
  seguridad por más tiempo del que esa minor tendría normalmente, sin
  verte forzado a saltar a 9.3, 9.4, etc. Es un add-on de suscripción,
  vive **dentro** del ciclo de 10 años.
  - **How to apply**: producción que no puede tolerar cambios de
    comportamiento por actualizaciones menores (ej. compliance
    regulatorio, certificaciones de hardware/software de terceros).
- **ELS (Extended Lifecycle Support)**: extiende el soporte **después**
  de que termina el ciclo completo de 10 años de una major version. Es
  el salvavidas para sistemas que todavía no pudieron migrar a la
  siguiente major.

En la práctica, `subscription-manager release --set=X.Y` es el comando
que clava tu sistema a una minor version particular y habilita los repos
EUS correspondientes (si tu suscripción los incluye).

### El fin (parcial) de AppStream modules

En RHEL 8, AppStream introdujo "module streams" (elegir versión de
PostgreSQL, Node.js, etc. vía `dnf module`). Desde RHEL 9 y consolidado
en RHEL 10, la mayoría de esos streams se simplificaron o eliminaron —
hoy `dnf module list` en RHEL 10 devuelve vacío para la gran mayoría de
paquetes, porque ya no hay múltiples streams que elegir: el paquete
correcto está directo en AppStream. Esto es un cambio real de
comportamiento frente a la documentación de RHEL 8, y lo vas a
comprobar vos mismo en la práctica.

## Práctica

```bash
# ver qué minor releases están disponibles para clavar con EUS
sudo subscription-manager release --list

# confirmar el linaje real del sistema
cat /etc/os-release

# comprobar el estado real de AppStream modules en RHEL 10
sudo dnf module list
```

## Hallazgos reales (verdad sobre guión)

1. **`/etc/os-release` declara `ID_LIKE="centos fedora"`** — confirmación
   literal, en el propio sistema instalado, del linaje upstream/downstream
   explicado en la teoría. También aparece `LOGO="fedora-logo-icon"`,
   un remanente curioso de theming compartido entre RHEL y Fedora.
2. **`dnf module list` no devolvió ninguna tabla de módulos** en RHEL
   10.2 — solo los mensajes de actualización de metadata, sin error.
   Confirma que el modelo de module streams de RHEL 8 prácticamente
   desapareció: no hay que memorizar sintaxis de `dnf module` para el
   examen de RHEL 10 de la misma forma que hubiera hecho falta en RHEL 8.
3. **`subscription-manager release --list`** mostró `10, 10.0, 10.1,
   10.2` — confirma que se puede clavar tanto la major (`10`, sigue las
   minors automáticamente vía repos generales) como una minor específica
   (`10.2`, para EUS real).

## Checklist

- [ ] `subscription-manager release --list` ejecutado y resultado revisado
- [ ] `/etc/os-release` inspeccionado, linaje `ID_LIKE` identificado
- [ ] `dnf module list` ejecutado, comportamiento de RHEL 10 confirmado
- [ ] Evidencias curadas en `evidencias/`

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_
