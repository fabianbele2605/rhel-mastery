# Módulo 18 — Contexto roto tras mover archivos 🔧 Break & Fix (cierra Fase V)

- Estado: En progreso
- Fecha: 2026-10-10
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: mismo mecanismo que en el módulo
  17 (`mv` preserva el contexto de origen) pero a escala real — un árbol
  completo de archivos, con diagnóstico que debe medir qué tan extendido
  está el problema antes de corregir, no solo arreglar un archivo suelto.

## Concepto diferencial

Simular una migración real: contenido web armado en una ubicación
temporal (`/tmp/sitio-nuevo/`), movido con `mv -v` recursivo a
`/var/www/html` — la operación típica de un sysadmin apurado sin pensar
en SELinux.

**Herramienta nueva**: `matchpathcon -V /ruta/*` verifica en lote si los
archivos coinciden con el contexto que la política espera, diagnóstico
proactivo antes de que algo falle.

## Cambio deliberado

```bash
sudo mkdir -p /tmp/sitio-nuevo
for n in index about contact; do echo "<h1>$n</h1>" | sudo tee /tmp/sitio-nuevo/$n.html > /dev/null; done
ls -lZ /tmp/sitio-nuevo/

sudo dnf install -y httpd
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload

sudo mv -v /tmp/sitio-nuevo/*.html /var/www/html/
ls -lZ /var/www/html/
curl localhost/index.html
```

## Síntoma

_(se completa con lo que salga)_

## Diagnóstico y recuperación

_(se completa con el proceso real)_

## Impacto y riesgos

_(se completa)_

## Cómo evitar recurrencia

_(se completa)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta provocar el síntoma completo, diagnosticar con matchpathcon, y
corregir con restorecon -Rv.
