# Módulo 04 — Simulacro cronometrado de gestión de software (cierre Fase I)

- Estado: En progreso
- Fecha: 2026-09-28
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: en Fedora se practicó `dnf`/`rpm`
  sin presión de tiempo, enfocado en entender qué hace cada comando. Acá
  el objetivo es velocidad y precisión bajo reloj — la habilidad que
  realmente evalúa el RHCSA no es "saber el comando" sino "ejecutarlo
  rápido sin errores", en un bloque que mezcla herramientas sin avisar
  cuál toca en cada tarea.

## Concepto diferencial

Este módulo no introduce teoría nueva: integra todo lo de la Fase I
(dnf/rpm ya conocido de Fedora, `subscription-manager repos` y Flatpak de
los módulos 02-03) bajo presión de tiempo, imitando el formato de un
bloque de examen real dentro de las 3 horas del RHCSA.

**Meta**: 15 minutos para las 8 tareas de abajo. Regla del simulacro: las
tareas se dan de una sola vez (como el enunciado real), se corre el reloj
sin pausas, y la revisión de qué salió bien/mal se hace al final, no en
el medio — salvo que el problema sea de infraestructura (VM/red), no de
conocimiento.

## Práctica guiada — enunciado del simulacro

Arrancá el reloj y resolvé, en orden o no (como en el examen real):

1. Instalar el paquete `tree` con `dnf`.
2. Averiguar con `rpm` qué paquete provee el archivo `/usr/bin/tree`, y
   ver su información completa (resumen, versión, licencia).
3. Listar todos los archivos que instaló el paquete `tree`.
4. Verificar la integridad del paquete `tree` instalado (no debería
   reportar ningún cambio).
5. Remover `tree` con `dnf`, y luego **deshacer esa transacción**
   específica con `dnf history` para que vuelva a quedar instalado (sin
   reinstalarlo a mano).
6. Instalar el grupo de paquetes **"Development Tools"** con `dnf`.
7. Instalar la app Flatpak **VLC** (`org.videolan.VLC`) desde Flathub.
8. Remover la app VLC y limpiar los runtimes que hayan quedado sin uso.

```bash
# antes de arrancar el reloj, estado limpio conocido
rpm -q tree || echo "tree no instalado (esperado)"
sudo dnf history list | head -5
date
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Correr las 8 tareas cronometradas y revisar resultado — falta ejecutar.
