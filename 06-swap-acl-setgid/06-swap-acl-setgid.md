# Módulo 06 — Swap, ACLs (setfacl/getfacl), directorios set-GID

- Estado: En progreso
- Fecha: 2026-09-29
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: ACLs y set-GID son conceptos
  genéricos ya vistos en Arch. Lo específico de RHEL acá es que **XFS
  soporta ACLs de forma nativa sin mount option extra** (`acl` viene
  implícito) — a diferencia de sistemas ext4 viejos donde había que
  agregarlo explícito en `fstab`.

## Concepto diferencial

**Swap adicional**: el sistema ya tiene swap vía LV (`rhel-swap`, visto en
el módulo 05). Acá se agrega swap **adicional** en un archivo, práctica
real cuando un servidor se queda corto de memoria sin rediseñar el
particionado original.

**ACLs**: `setfacl -m u:usuario:rwx archivo` agrega un permiso puntual
además del dueño/grupo/otros clásico; `getfacl` muestra el resultado;
`-d` en un directorio crea una ACL **default** que se hereda a los
archivos nuevos creados adentro.

**set-GID en directorios**: `chmod g+s directorio` hace que todo archivo
nuevo creado ahí herede el grupo del directorio, no el grupo primario del
usuario que lo crea. Combinado con ACLs default, es el patrón real para
carpetas compartidas de equipo.

Se usa `/datos` (el LV armado en el módulo 05) como terreno de pruebas
para ACLs y set-GID.

## Práctica guiada

```bash
# estado de memoria/swap actual
free -h

# swap adicional en archivo
sudo fallocate -l 512M /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
free -h

# persistencia en fstab
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# ACLs sobre /datos
sudo setfacl -m u:fbeleno:rwx /datos
getfacl /datos

# ACL default (herencia) sobre /datos
sudo setfacl -d -m u:fbeleno:rwx /datos
sudo touch /datos/prueba.txt
getfacl /datos/prueba.txt

# set-GID sobre /datos
sudo chmod g+s /datos
ls -ld /datos
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta ejecutar ACLs y set-GID — solo se corrió el swap adicional hasta
ahora.
