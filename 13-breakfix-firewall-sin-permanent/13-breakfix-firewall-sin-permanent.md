# Módulo 13 — Regla de firewall sin `--permanent` 🔧 Break & Fix

- Estado: En progreso
- Fecha: 2026-10-02
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: el concepto runtime-vs-permanent
  de `firewalld` es el mismo en cualquier distro. Lo específico de este
  módulo es el flujo de diagnóstico esperado en el RHCSA: comparar
  `firewall-cmd --list-all` (runtime activo) contra `firewall-cmd
  --permanent --list-all` (lo que sobrevive un reboot) para detectar la
  discrepancia **sin necesitar reiniciar**.

## Concepto diferencial

Cierra la Fase III del curso (Red, servicios y scheduling). El error más
común en producción con `firewalld`: un cambio que "funciona" en el
momento pero solo tocó el runtime, y desaparece en el próximo reboot o
`firewall-cmd --reload`. Ya se advirtió en el módulo 10 — acá se vive en
carne propia.

## Cambio deliberado

Agregar una regla **sin `--permanent`**: un puerto nuevo que funciona de
inmediato, pero que no quedó guardado para sobrevivir un reinicio.

```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=9090/tcp
sudo firewall-cmd --list-ports
```

## Síntoma

_(se completa con lo que salga al comparar runtime vs permanent, y al
reiniciar)_

## Diagnóstico y recuperación

_(se completa con el proceso real)_

## Impacto y riesgos

_(se completa)_

## Cómo evitar recurrencia

_(se completa)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta el diagnóstico sin reboot, el reboot real, y la recuperación.
