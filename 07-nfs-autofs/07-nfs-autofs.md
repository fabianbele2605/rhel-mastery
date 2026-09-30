# Módulo 07 — NFS y autofs con `rhel10-client`

- Estado: En progreso
- Fecha: 2026-09-30
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: NFS/autofs no se cubrió en el
  curso de Fedora — contenido nuevo real. El ángulo específico de RHEL es
  que **SELinux enforcing bloquea NFS por defecto** salvo que se usen los
  booleans correctos o el contexto SELinux adecuado en el directorio
  exportado, algo que en laboratorios con SELinux permissive ni se nota.

## Concepto diferencial

**Servidor NFS**: paquete `nfs-utils`, exports declarados en `/etc/exports`
(opciones `rw`/`ro`, `sync`/`async`, `no_root_squash` vs el squash por
defecto que mapea root remoto a `nobody`), servicio `nfs-server`, y el
puerto/servicio abierto en `firewalld` (`--add-service=nfs`).

**Cliente manual vs autofs**: montar a mano con `mount -t nfs` funciona
pero no es lo que evalúa el examen — `autofs` (`/etc/auto.master` + un
mapa indirecto en `/etc/auto.misc`) monta bajo demanda y desmonta solo
tras inactividad, el patrón real esperado en el RHCSA.

**SELinux**: con SELinux enforcing (default en RHEL), compartir un
directorio por NFS puede requerir ajustar el contexto SELinux del
directorio exportado o habilitar booleans como `nfs_export_all_rw`,
según el escenario.

## Práctica guiada

### Antes de arrancar: crear `rhel10-client`

VM nueva en VirtualBox: 1 GB RAM, 1 vCPU, 15 GB disco, misma ISO de RHEL
10.2, misma red NAT (o red interna compartida con `rhel10-server` si se
prefiere aislar el tráfico NFS del resto). Instalación mínima (sin GUI),
usuario con sudo, hostname `rhel10-client.local`.

### En `rhel10-server` (servidor NFS)

```bash
sudo dnf install -y nfs-utils
sudo mkdir -p /export/compartido
sudo chmod 755 /export/compartido
echo '/export/compartido *(rw,sync,no_root_squash)' | sudo tee -a /etc/exports
sudo systemctl enable --now nfs-server
sudo exportfs -rav
sudo firewall-cmd --add-service=nfs --permanent
sudo firewall-cmd --reload
```

### En `rhel10-client`

```bash
sudo dnf install -y nfs-utils autofs
showmount -e rhel10-server   # o la IP directa

# montaje manual de prueba
sudo mkdir -p /mnt/compartido
sudo mount -t nfs rhel10-server:/export/compartido /mnt/compartido

# autofs (montaje bajo demanda)
echo '/mnt/auto /etc/auto.misc' | sudo tee -a /etc/auto.master
echo 'compartido -rw rhel10-server:/export/compartido' | sudo tee -a /etc/auto.misc
sudo systemctl enable --now autofs
ls /mnt/auto/compartido
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Crear la VM `rhel10-client` — falta ejecutar toda la práctica.
