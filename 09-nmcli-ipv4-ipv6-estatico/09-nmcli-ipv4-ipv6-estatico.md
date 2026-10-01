# Módulo 09 — `nmcli` IPv4/IPv6 estático

- Estado: En progreso
- Fecha: 2026-10-01
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: `nmcli` en sí ya se conoce. Lo
  específico de RHEL para el examen es que los viejos scripts
  `/etc/sysconfig/network-scripts/ifcfg-*` están **deprecados desde
  RHEL 8/9** — el backend por defecto ahora es **keyfile**
  (`/etc/NetworkManager/system-connections/*.nmconnection`), y el RHCSA
  espera manejo de `nmcli`/`nmtui`, no edición manual de esos archivos.

## Concepto diferencial

En el módulo 07 ya se usó `nmcli con add ... ipv4.addresses ...
ipv4.method manual` para una IP estática simple. Acá se completa el
perfil: dirección + prefijo + gateway + DNS en un solo lugar, más la
contraparte IPv6 estática.

**IPv4 estático completo**: `ipv4.addresses`, `ipv4.gateway`,
`ipv4.dns`, todo en el mismo `nmcli con mod`.

**IPv6 estático**: mismo patrón con `ipv6.method manual` —
**explícito**, si no NetworkManager intenta autoconfiguración SLAAC
igual aunque se declare una dirección IPv6 estática.

**Verificación**: `nmcli con show <nombre>`, `nmcli device show`, `ip
-4/-6 addr`, y revisar (sin editar a mano) el archivo `.nmconnection`
generado para entender qué escribió `nmcli` realmente.

## Práctica guiada

```bash
# estado actual de conexiones
nmcli con show
cat /etc/NetworkManager/system-connections/redlab.nmconnection

# completar IPv4 estático en la conexión redlab (módulo 07) con gateway y DNS
sudo nmcli con mod redlab ipv4.gateway 192.168.100.1
sudo nmcli con mod redlab ipv4.dns "192.168.100.1 8.8.8.8"
sudo nmcli con up redlab

# agregar IPv6 estático a la misma conexión
sudo nmcli con mod redlab ipv6.method manual ipv6.addresses fd00:192:168:100::1/64
sudo nmcli con up redlab

# verificación
nmcli con show redlab
ip -4 addr show enp0s8
ip -6 addr show enp0s8
cat /etc/NetworkManager/system-connections/redlab.nmconnection
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta ejecutar toda la práctica.
