# Módulo 05 — LVM completo cronometrado (meta: bajar de 6 minutos)

- Estado: En progreso
- Fecha: 2026-09-28
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: Fedora Server también usa
  LVM+XFS por defecto en Anaconda — no hay diferencia conceptual ahí, y
  PV/VG/LV ya se cubrieron a fondo en Arch. La diferencia real de este
  módulo es la **velocidad bajo presión de examen** y un detalle técnico
  concreto: XFS (default en RHEL desde la 7) **solo crece, nunca se
  achica** — no hay `resize2fs` ni `lvreduce` seguro como con ext4.

## Concepto diferencial

Armar el stack completo de LVM (disco → PV → VG → LV → filesystem →
mount persistente) es una de las tareas más comunes y "baratas en puntos"
del RHCSA si está automatizada de memoria. El objetivo no es entender qué
es un PV o un VG — eso ya se vio en Arch — sino ejecutarlo sin dudar bajo
reloj.

**Cadena de comandos**: `pvcreate` → `vgcreate` → `lvcreate` → `mkfs.xfs`
→ `mount` + entrada persistente en `/etc/fstab` (con UUID, nunca con
`/dev/sdX` a mano — los nombres de dispositivo pueden cambiar entre
reboots).

**Extender en caliente**: `lvextend -r` hace resize del filesystem en el
mismo comando que extiende el LV, ahorrando un paso separado de
`xfs_growfs`.

**La trampa de XFS**: a diferencia de ext4, **XFS no se puede achicar**.
Si en el examen te piden reducir un LV con XFS, la única forma real es
recrear el filesystem desde cero con el tamaño correcto (backup → borrar
→ recrear más chico → restaurar). Confundir esto en el momento cuesta
carísimo en tiempo.

## Práctica guiada — enunciado del simulacro

**Antes de arrancar el reloj**: agregar un disco virtual nuevo de 5 GB a
`rhel10-server` desde VirtualBox (Configuración → Almacenamiento → Añadir
disco duro, con la VM apagada), para no tocar el disco del sistema.

Arrancá el reloj y resolvé:

1. Confirmar que el disco nuevo aparece en el sistema (`lsblk`) sin LVM
   previo.
2. Crear un Physical Volume sobre el disco completo.
3. Crear un Volume Group llamado `vg_datos` sobre ese PV.
4. Crear un Logical Volume de 2 GB llamado `lv_datos` dentro de `vg_datos`.
5. Formatear `lv_datos` con XFS.
6. Montarlo en `/datos`, y dejar el montaje persistente en `/etc/fstab`
   usando el UUID del filesystem (no la ruta del dispositivo).
7. Verificar que sobrevive: `umount` + `mount -a` sin errores.
8. Extender `lv_datos` en 1 GB más (a 3 GB total) y hacer crecer el
   filesystem en el mismo paso con `lvextend -r`.

```bash
# antes de arrancar el reloj, estado limpio conocido
sudo pvs; sudo vgs; sudo lvs
lsblk
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Agregar el disco virtual en VirtualBox y correr las 8 tareas cronometradas
— falta ejecutar.
