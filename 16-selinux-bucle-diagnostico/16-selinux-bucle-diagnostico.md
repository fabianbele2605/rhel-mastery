# Módulo 16 — Bucle diagnosticar → corregir → verificar (SELinux)

- Estado: En progreso
- Fecha: 2026-10-05
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: en Fedora SELinux fue conceptual
  (modos, contextos, booleans en teoría). Acá se provoca una denegación
  real sirviendo contenido desde una ruta no estándar, y se resuelve con
  el bucle completo de diagnóstico esperado en el RHCSA.

## Concepto diferencial

Instalar `httpd` y servir contenido desde una ruta **no estándar** (ej.
`/opt/webdata` en vez de `/var/www/html`) — SELinux bloquea el acceso
aunque los permisos Unix estén perfectos, porque el contexto de archivo
no es el esperado para un proceso `httpd_t`.

**Herramientas del bucle**: `ausearch -m avc -ts recent` o `journalctl -t
setroubleshoot` para la denegación cruda; `sealert -a
/var/log/audit/audit.log` para la explicación humana con solución
sugerida; `semanage fcontext -a -t httpd_sys_content_t '/ruta(/.*)?'` +
`restorecon -Rv /ruta` para la corrección persistente (no `chcon` suelto,
que se pierde en el próximo relabel).

**La trampa clásica**: `setenforce 0` "arregla" cualquier cosa, pero es
la respuesta incorrecta en el examen — apaga la protección en vez de
resolver el contexto.

## Práctica guiada

```bash
getenforce
sudo dnf install -y httpd setroubleshoot-server
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload

# provocar la denegación: servir desde una ruta no estándar
sudo mkdir -p /opt/webdata
echo "<h1>Pagina de prueba SELinux</h1>" | sudo tee /opt/webdata/index.html
sudo sed -i 's#DocumentRoot "/var/www/html"#DocumentRoot "/opt/webdata"#' /etc/httpd/conf/httpd.conf
sudo sed -i 's#<Directory "/var/www/html">#<Directory "/opt/webdata">#' /etc/httpd/conf/httpd.conf
sudo systemctl restart httpd
curl localhost

# diagnosticar
sudo ausearch -m avc -ts recent
ls -ldZ /opt/webdata

# corregir
sudo semanage fcontext -a -t httpd_sys_content_t '/opt/webdata(/.*)?'
sudo restorecon -Rv /opt/webdata

# verificar
curl localhost
getenforce
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta provocar la denegación real, diagnosticarla y corregirla.
