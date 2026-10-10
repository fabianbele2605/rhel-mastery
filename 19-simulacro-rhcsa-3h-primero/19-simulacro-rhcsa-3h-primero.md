# Módulo 19 — Primer simulacro RHCSA completo (3 horas)

- Estado: En progreso
- Fecha: 2026-10-10
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: no aplica — 100% examen RHCSA.

## Concepto diferencial

Arranca la Fase VI. Primer simulacro completo de 3 horas, mezclando todo
lo visto en las Fases 0-V sin indicar qué módulo corresponde a cada
tarea — tal como el examen real. Regla del simulacro: enunciado completo
de una sola vez, reloj corrido por el alumno, sin pistas en el medio
salvo problemas de infraestructura (no de conocimiento).

## Enunciado del simulacro

Usá `rhel10-server` como principal y `rhel10-client` donde se indique.
Todas las tareas deben sobrevivir un reboot salvo que se diga lo
contrario. Meta: 3 horas.

1. **Almacenamiento**: creá un Volume Group `examen_vg` sobre un disco
   nuevo de al menos 2 GB, un Logical Volume `datos_lv` de 1.5 GB,
   formateado XFS, montado de forma persistente en `/examen/datos`.
2. **Swap**: agregá 256 MB de swap adicional en archivo, persistente.
3. **ACLs**: en `/examen/datos`, dale a tu usuario normal permisos `rwx`
   vía ACL (no por el dueño/grupo clásico), y activá set-GID en el
   directorio.
4. **Usuario con expiración**: creá el usuario `auditor`, con la cuenta
   expirando el 2027-06-01, y sudo granular para que pueda correr
   **solo** `systemctl status *` sin contraseña (ningún otro comando).
5. **Red**: agregá una dirección IPv6 estática a la interfaz de red
   interna (la que conecta con `rhel10-client`), distinta a la que ya
   tenés, y confirmá conectividad con `ping6` desde el cliente.
6. **Firewall**: abrí el puerto `9191/tcp` de forma permanente,
   solamente en la zona de la interfaz externa (NAT), no en la interna.
7. **Systemd timer**: creá un timer+service que, cada 10 minutos,
   registre el porcentaje de uso de `/examen/datos` en
   `/examen/datos/uso.log` (usando `df` o similar).
8. **Script con exit code**: el script que corre el timer de la tarea 7
   debe devolver exit code `1` si el uso supera el 80%, y `0` si no —
   probalo forzando ambos casos.
9. **NFS**: exportá `/examen/datos` de forma **solo lectura** hacia la
   red `192.168.100.0/24`, y montalo en `rhel10-client` vía `autofs` en
   `/mnt/auto/examen`.
10. **Grupos**: agregá el usuario `auditor` a un grupo secundario nuevo
    llamado `revisores`, y confirmalo con `id auditor`.
11. **SELinux**: instalá `httpd`, serví contenido desde una ruta no
    estándar de tu elección, y dejalo funcionando con SELinux en
    `Enforcing` (diagnosticando y corrigiendo vos mismo, sin mirar los
    módulos anteriores si podés evitarlo).
12. **Break & Fix autoimpuesto**: corrompé a propósito una entrada de
    `/etc/fstab` (no la de `/examen/datos`, otra distinta para no perder
    el trabajo de la tarea 1), reiniciá, y recuperate vos mismo desde
    emergency mode.
13. **Tiempo**: confirmá que `chronyd` está sincronizado y que el
    journal es persistente (si alguna VM quedó con journal volátil,
    corregilo).
14. **Verificación final**: documentá con capturas que las 13 tareas
    anteriores siguen funcionando después de un reboot completo de
    `rhel10-server`.

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta ejecutar las 14 tareas del simulacro.
