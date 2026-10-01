# Módulo 10 — `firewalld` con `--permanent --reload`

- Estado: En progreso
- Fecha: 2026-10-01
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: `firewalld` en sí ya se conoce de
  Fedora. Lo que se aprovecha acá es que `rhel10-server` tiene **dos
  interfaces reales** (`enp0s3` NAT externa, `enp0s8` red interna
  confiable hacia `rhel10-client`) — se les puede asignar **zonas
  distintas**, algo que en un desktop Fedora de una sola NIC casi no se
  practica.

## Concepto diferencial

**Runtime vs permanent**: `firewall-cmd` tiene dos capas — la que corre
ahora (runtime) y la que persiste tras reboot (`--permanent`, guardada en
XML). Cambiar una no toca la otra automáticamente — hay que elegir:
tocar ambas a la vez, o `--permanent` + `firewall-cmd --reload` para que
el runtime adopte lo permanente.

**Zonas**: `--get-default-zone`, `--get-active-zones`, `--list-all-zones`
— cada zona tiene su propio set de servicios/puertos permitidos; una
interfaz vive en una zona a la vez.

**El comando que evalúa el examen**: `firewall-cmd --permanent
--add-service=X` **sin** `--reload` después no tiene efecto inmediato
(se sufre a propósito en el módulo 13, el Break & Fix de este tema) —
acá se practica el flujo correcto antes de ver el flujo roto.

**Puertos vs servicios**: `--add-service=http` (nombre conocido, puertos
ya definidos) vs `--add-port=8080/tcp` (puerto suelto).

## Práctica guiada

```bash
# estado actual de zonas
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all

# mover la interfaz interna a una zona más confiable
sudo firewall-cmd --zone=internal --change-interface=enp0s8 --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --get-active-zones

# agregar un servicio/puerto con el flujo correcto (permanent + reload)
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-services

# puerto suelto
sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-ports
```

## Hallazgos reales

1. **Ambas interfaces (`enp0s3` y `enp0s8`) estaban en la misma zona
   `public`** por defecto, confundiendo tráfico externo (NAT) con
   tráfico interno confiable (red hacia `rhel10-client`) — exactamente
   la oportunidad de separarlas en zonas distintas que plantea la teoría
   de este módulo.
2. **La zona `public` ya traía servicios de módulos anteriores**: `nfs`,
   `rpc-bind`, `mountd` (módulo 07), además de `cockpit`, `dhcpv6-client`,
   `ssh` por defecto — confirma que las reglas de firewall persisten
   entre módulos, como se espera.

## Evidencias

_(se completa con lo que salga en la práctica)_

## Pendientes

Falta mover `enp0s8` a una zona interna y practicar `--add-service`/
`--add-port` con el flujo correcto.
