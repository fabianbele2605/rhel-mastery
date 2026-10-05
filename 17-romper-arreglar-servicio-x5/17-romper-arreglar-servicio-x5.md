# Módulo 17 — Romper y arreglar un servicio real x5

- Estado: En progreso
- Fecha: 2026-10-05
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: no hay teoría nueva — es puro
  entrenamiento de reflejos sobre herramientas ya vistas en el módulo 16
  (`ausearch`, `semanage`, `setsebool`, `restorecon`), repetidas sobre
  distintos escenarios para automatizar el diagnóstico.

## Concepto diferencial

El módulo 16 fue el diagnóstico profundo con tiempo para investigar cada
capa. Este módulo es velocidad: 5 ciclos rápidos de romper/diagnosticar/
arreglar, variando el tipo de problema SELinux cada vez.

**Los 5 escenarios**:
1. `sshd` en puerto no estándar (`2222`) — `semanage port -a -t
   ssh_port_t -p tcp 2222`.
2. `httpd` en puerto no estándar (`8081`) — mismo patrón con
   `http_port_t`.
3. Archivo movido con `mv` pierde el contexto SELinux correcto (a
   diferencia de `cp`, que asigna contexto nuevo por defecto).
4. Boolean de red: `httpd_can_network_connect`.
5. `vsftpd` con directorio de usuario (`ftp_home_dir`).

## Práctica guiada

### Escenario 1 — sshd en puerto 2222

```bash
sudo semanage port -l | grep ssh
sudo sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config
sudo systemctl restart sshd
systemctl status sshd --no-pager
sudo ausearch -m avc -ts recent

sudo semanage port -a -t ssh_port_t -p tcp 2222
sudo systemctl restart sshd
systemctl status sshd --no-pager
```

### Escenario 2 — httpd en puerto 8081

```bash
sudo semanage port -l | grep http_port_t
sudo sed -i 's/Listen 80/Listen 8081/' /etc/httpd/conf/httpd.conf
sudo systemctl restart httpd
sudo ausearch -m avc -ts recent
sudo semanage port -a -t http_port_t -p tcp 8081
sudo systemctl restart httpd
curl localhost:8081
```

### Escenario 3 — mv pierde contexto

```bash
echo "contenido de prueba" | sudo tee /root/pagina.html
sudo mkdir -p /var/www/html
sudo mv /root/pagina.html /var/www/html/pagina.html
ls -lZ /var/www/html/pagina.html
curl localhost/pagina.html
sudo restorecon -v /var/www/html/pagina.html
curl localhost/pagina.html
```

### Escenario 4 — httpd_can_network_connect

```bash
getsebool httpd_can_network_connect
sudo setsebool -P httpd_can_network_connect on
getsebool httpd_can_network_connect
```

### Escenario 5 — vsftpd y ftp_home_dir

```bash
sudo dnf install -y vsftpd
sudo systemctl enable --now vsftpd
getsebool ftp_home_dir
sudo setsebool -P ftp_home_dir on
getsebool ftp_home_dir
```

## Hallazgos reales

_(se completa con lo que salga en la práctica)_

## Evidencias

_(pendiente — se completa cuando digas "verifica img")_

## Pendientes

Falta ejecutar los 5 escenarios completos.
