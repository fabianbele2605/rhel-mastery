# Módulo 14 — Usuarios/grupos, `chage`, sudo granular

- Estado: En progreso
- Fecha: 2026-10-02
- Versión: RHEL 10.2 (Coughlan)
- Objetivo diferencial frente a Fedora: `useradd`/`usermod`/`groupadd` ya
  se conocen de Arch/Fedora. Lo específico de RHEL para el examen es que
  se espera `visudo -f /etc/sudoers.d/nombre-de-archivo` para sudo
  granular — nunca editar `/etc/sudoers` directo ni usar un editor común
  (`visudo` valida sintaxis antes de guardar).

## Concepto diferencial

Los defaults de `useradd` (rango de UID, política de password) vienen de
`/etc/login.defs`. `chage -l usuario` muestra la política de expiración
completa; `-M` (días máximos), `-m` (mínimos entre cambios), `-W` (aviso
antes de expirar), `-E` (fecha de expiración de la **cuenta**, no de la
contraseña — distinción que se presta a confusión en el examen).

**Sudo granular con `Cmnd_Alias`**: en vez de meter a alguien en `wheel`
(sudo total), se le puede dar permiso solo para comandos específicos,
con o sin `NOPASSWD`.

## Práctica guiada

```bash
# defaults del sistema
cat /etc/login.defs | grep -E "^UID_MIN|^PASS_MAX_DAYS|^PASS_MIN_DAYS"

# usuario de prueba
sudo useradd -c "Usuario de prueba" operador
sudo chage -l operador

# política de expiración concreta
sudo chage -M 90 -m 7 -W 14 -E 2027-01-01 operador
sudo chage -l operador

# sudo granular: solo reiniciar un servicio puntual, sin password
echo 'operador ALL=(root) NOPASSWD: /usr/bin/systemctl restart sshd' | sudo tee /etc/sudoers.d/operador-sshd
sudo visudo -c -f /etc/sudoers.d/operador-sshd
sudo chmod 440 /etc/sudoers.d/operador-sshd

# verificación: operador puede reiniciar sshd pero no nada más
sudo -u operador sudo -l
sudo -u operador sudo systemctl restart sshd
sudo -u operador sudo systemctl restart chronyd
```

## Hallazgos reales

1. **`UID_MIN=1000`** — mismo valor que Fedora, confirma que el rango de
   UIDs para usuarios normales no es un diferenciador real entre ambas
   distros (ambas migraron de 500 a 1000 hace tiempo).
2. **`PASS_MAX_DAYS=99999`** por defecto — en la práctica, "nunca expira"
   (99999 días ≈ 273 años). Los usuarios nuevos no tienen política de
   expiración real hasta que se configure explícitamente con `chage`.

## Evidencias

**01 — Defaults de login.defs y usuario creado**
`UID_MIN=1000`, `PASS_MAX_DAYS=99999`, `PASS_MIN_DAYS=0`, y `chage -l operador` mostrando la política por defecto (sin expiración) tras crear el usuario (hallazgos #1 y #2).
![Defaults de login.defs y chage -l inicial](evidencias/01-login-defs-chage-inicial.png)

## Pendientes

Falta configurar la política de expiración concreta con `chage` y
probar el sudo granular.
