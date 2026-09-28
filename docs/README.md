# rhel-mastery

Cuarto curso autodidacta de Linux, tutorado por IA (Claude), sobre **Red Hat
Enterprise Linux** con la mira puesta en servidores enterprise y el camino
hacia el **RHCSA (EX200)**. Continuación directa de:

- [`arch-linux-mastery`](https://github.com/fabianbele2605/arch-linux-mastery) — sysadmin, servidores, seguridad, kernel.
- [`arch-linux-desktop`](https://github.com/fabianbele2605/arch-linux-desktop) — Arch como sistema de escritorio completo.
- [`fedora-mastery`](https://github.com/fabianbele2605/fedora-mastery) — RPM/DNF, SELinux, rpm-ostree, Rust en Fedora.

## Alcance

Este curso **no repite** fundamentos genéricos de Linux ya cubiertos en los
cursos de Arch, ni lo ya visto en Fedora (RPM/DNF básico, SELinux
conceptual, Rust + `.spec`). Se enfoca en lo **específico de RHEL** y en la
preparación real para el RHCSA:

- Diferencias RHEL vs Fedora: upstream/downstream, ciclo de vida de 10 años,
  `subscription-manager`
- Podman en vez de Docker: rootless, pods, Quadlets con systemd
- SELinux enforcing en contexto de producción, no solo laboratorio
- Ansible específico de Red Hat: `ansible.posix`, `redhat.rhel_system_roles`
- Temario alineado a los **objetivos oficiales del EX200 sobre RHEL 10**
  (incluye Flatpak y systemd timers, agregados este año; los contenedores
  ya no figuran en los objetivos oficiales, pero se cubren igual por su
  peso real en la infraestructura Red Hat)
- Rust como daemon systemd endurecido (SELinux + sandboxing de systemd) en
  un servidor RHEL real

## Metodología

Igual que en los cursos anteriores:

- Cada módulo combina **teoría + práctica real + evidencias** (capturas
  curadas con descripción).
- **Break & Fix**: incidentes reales provocados y resueltos, documentados
  sin ocultar errores.
- Todo se ejecuta sobre **VirtualBox**, con Red Hat Developer Subscription
  for Individuals para las licencias de RHEL 10.
- Commits con `Co-Authored-By: Claude`.
- Fase VI incluye simulacros cronometrados de 3 horas, como el examen real.

## Entorno

| VM | Rol | RAM | Disco |
|---|---|---|---|
| `rhel10-server` | Casi todos los módulos | 4 GB, 2 vCPU | 40 GB |
| `rhel10-client` | Solo NFS/autofs/SSH/red | 1 GB | 15 GB |

Ambas VMs pueden correr juntas sin problema en un anfitrión de 8 GB.

## Progreso

**Módulo actual: 03 / 31**

### Fase 0 — RHEL frente al resto del ecosistema
- [x] 00. Registro con `subscription-manager`, entitlements, instalación de RHEL 10 ([módulo](../00-subscription-manager-instalacion/00-subscription-manager-instalacion.md))
- [x] 01. Flujo Fedora → CentOS Stream → RHEL, ciclo de vida de 10 años (EUS/ELS) ([módulo](../01-rhel-vs-fedora-vs-centos-stream/01-rhel-vs-fedora-vs-centos-stream.md))

### Fase I — Software: dnf/rpm/Flatpak al ritmo del examen
- [x] 02. Repos vía `subscription-manager repos`, AppStream/BaseOS ([módulo](../02-repos-appstream-baseos/02-repos-appstream-baseos.md))
- [ ] 03. Flatpak: remoto, búsqueda, instalación, listado
- [ ] 04. Simulacro cronometrado de gestión de software

### Fase II — Almacenamiento
- [ ] 05. LVM completo cronometrado (meta: bajar de 6 minutos)
- [ ] 06. Swap, ACLs (setfacl/getfacl), directorios set-GID
- [ ] 07. NFS y autofs con `rhel10-client`
- [ ] 08. `/etc/fstab` corrupto — recuperación desde rescate 🔧 *Break & Fix*

### Fase III — Red, servicios y scheduling
- [ ] 09. `nmcli` IPv4/IPv6 estático
- [ ] 10. `firewalld` con `--permanent --reload`
- [ ] 11. `chrony` y journal persistente
- [ ] 12. Systemd timers (`.service`/`.timer` a mano)
- [ ] 13. Regla de firewall sin `--permanent` 🔧 *Break & Fix*

### Fase IV — Usuarios y scripting mínimo viable
- [ ] 14. Usuarios/grupos, `chage`, sudo granular
- [ ] 15. Shell scripting a nivel de examen

### Fase V — SELinux en producción
- [ ] 16. Bucle diagnosticar → corregir → verificar
- [ ] 17. Romper y arreglar un servicio real x5
- [ ] 18. Contexto roto tras mover archivos 🔧 *Break & Fix*

### Fase VI — Simulacro RHCSA completo
- [ ] 19. Primer simulacro de 3 horas
- [ ] 20. Segundo simulacro + reparación de puntos débiles
- [ ] 21. Simulacro final + checklist de errores comunes

### Fase VII — Podman en serio
- [ ] 22. Arquitectura daemonless y rootless
- [ ] 23. Pods y Quadlets
- [ ] 24. Por qué importa más allá del examen

### Fase VIII — Ansible específico de Red Hat
- [ ] 25. `ansible.posix` (selinux/firewalld/mount)
- [ ] 26. `redhat.rhel_system_roles` (timesync/storage/network/selinux)
- [ ] 27. System role fallido depurado con `-vvv` 🔧 *Break & Fix*

### Fase IX — Rust como daemon systemd en RHEL real
- [ ] 28. Binario Rust como unidad systemd con dominio SELinux propio
- [ ] 29. Endurecimiento con directivas de sandboxing de systemd
- [ ] 30. Proyecto final: daemon + Ansible + firewalld + SELinux, verificado tras reboot

## Estructura del repo

```
rhel-mastery/
├── docs/
│   ├── README.md
│   └── tutor.txt
├── img/                                  (capturas crudas, gitignored)
├── 00-subscription-manager-instalacion/
│   ├── 00-subscription-manager-instalacion.md
│   └── evidencias/
├── 01-rhel-vs-fedora-vs-centos-stream/
│   ├── 01-rhel-vs-fedora-vs-centos-stream.md
│   └── evidencias/
└── ...
```

Cada carpeta de módulo sigue el mismo patrón que en Fedora/Arch: un único
`NN-nombre-modulo.md` (mismo nombre que la carpeta) con encabezado de
metadata (Estado/Fecha/Versión/Objetivo diferencial frente a Fedora),
`## Concepto diferencial`, `## Práctica guiada`, `## Hallazgos reales`,
`## Evidencias` (capturas numeradas en `evidencias/`, cada una con un
párrafo de contexto) y `## Pendientes`. Los incidentes de Break & Fix van
**integrados en el módulo correspondiente** (no en un archivo aparte),
con las mismas subsecciones que en Fedora/Arch: Cambio deliberado →
Síntoma → Diagnóstico y recuperación → Impacto y riesgos → Cómo evitar
recurrencia.

## Cursos siguientes

Tras este curso, la ruta continúa con **Kali** (metapaquetes, metodología
de pentesting ofensivo, laboratorio aislado, reporte profesional).
