# Módulo 03 — Flatpak: remoto, búsqueda, instalación, listado

- Estado: En progreso
- Fecha: 2026-09-28
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: en Fedora Workstation/Silverblue
  el remoto Flathub viene preconfigurado de fábrica, y en Silverblue
  Flatpak es central a todo el sistema vía rpm-ostree. En RHEL 10 no hay
  ningún remoto por defecto, ni el paquete `flatpak` garantizado en toda
  instalación — se agrega todo a mano.

## Concepto diferencial

Flatpak es nuevo en los objetivos oficiales del EX200 sobre RHEL 10 —
forma de distribuir apps de escritorio/GUI sandboxeadas, independiente del
ciclo de vida de AppStream. Para el examen se espera manejo de: agregar un
remoto, buscar e instalar una app puntual, y listar lo instalado.

**Comando central**: `flatpak remote-add --if-not-exists flathub
https://flathub.org/repo/flathub.flatpakrepo` — el `--if-not-exists` es
clave para idempotencia (relevante para Ansible en la fase VIII).

**Búsqueda e instalación**: `flatpak search <término>` contra los remotos
configurados, `flatpak install <remoto> <app-id>`, `flatpak list` (con
`--app` o `--runtime` para filtrar) para ver qué quedó instalado.

## Práctica guiada

```bash
# confirmar si flatpak está instalado
which flatpak || sudo dnf install -y flatpak

# remotos configurados (esperado: ninguno por defecto en RHEL)
flatpak remotes

# agregar Flathub
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# confirmar el remoto agregado
flatpak remotes
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Búsqueda, instalación y listado de una app concreta — falta ejecutar.
