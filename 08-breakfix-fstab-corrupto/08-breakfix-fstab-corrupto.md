# Módulo 08 — `/etc/fstab` corrupto — recuperación desde rescate 🔧 Break & Fix

- Estado: En progreso
- Fecha: 2026-09-30
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: el mecanismo de rescate (systemd
  emergency target, `journalctl -xb`, remount de `/` en RW) es genérico
  de systemd, ya visto en Arch. Lo específico de RHEL/RHCSA es el flujo
  exacto que espera el examen: loguear en emergency mode con la
  contraseña de root, remount, editar `fstab`, `daemon-reload`, y
  confirmar con reboot.

## Concepto diferencial

Un error de tipeo o un UUID inventado en `/etc/fstab` puede dejar el
sistema sin bootear completo, cayendo en **emergency mode** de systemd —
uno de los escenarios más comunes y más temidos del RHCSA. La cuenta root
está deshabilitada para login normal (decisión del módulo 00), pero el
modo emergencia de systemd **sí pide contraseña de root** — sin ella, no
hay forma de entrar.

Cierra la Fase II (Almacenamiento) del curso, construyendo sobre el LV
`/datos` armado en el módulo 05.

## Cambio deliberado

1. Establecer contraseña de root (necesaria para emergency mode):
   `sudo passwd root`.
2. Respaldar el `/etc/fstab` real antes de tocarlo.
3. Corromper la entrada de `/datos` — UUID inventado que no corresponde
   a ningún filesystem real.
4. Reiniciar la VM.

```bash
sudo passwd root
sudo cp /etc/fstab /etc/fstab.bak
cat /etc/fstab
```

## Síntoma

_(se completa con lo que salga al reiniciar — pantalla de emergency mode,
mensaje exacto de systemd)_

## Diagnóstico y recuperación

_(se completa con el proceso real: observación, journalctl, hipótesis,
prueba de causa raíz, solución aplicada, validación)_

## Impacto y riesgos

_(se completa)_

## Cómo evitar recurrencia

_(se completa)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta ejecutar todo el Break & Fix.
