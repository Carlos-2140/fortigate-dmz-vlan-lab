# FortiGate DMZ VLAN Lab

Laboratorio de seguridad de red desarrollado en **GNS3** utilizando **FortiGate 7.0.9**, un switch **Cisco IOSvL2**, dos estaciones Windows y tres servidores Ubuntu en una DMZ.

El objetivo principal es implementar segmentación mediante VLAN, control de acceso entre redes, protección de una DMZ, filtrado web, acceso SSH restringido y salida a Internet limitada para los servidores.

## Video de demostración

> Pendiente: agregar aquí el enlace del video de demostración del laboratorio.

---

## Topología

![Diagrama de topología](Topologia/Diagrama%20Topolog%C3%ADa.png)

La topología está compuesta por:

- **Cloud1 / Internet** conectado al **port1** del FortiGate.
- **FortiGate 7.0.9-1** como firewall y gateway de las redes.
- **Cisco IOSvL2** conectado al **port2** del FortiGate mediante trunk 802.1Q.
- **VLAN 10** para usuarios.
- **VLAN 20** para administración.
- **DMZ-SW** en el FortiGate, formada por los puertos físicos **port3, port4 y port5**.
- Tres servidores Ubuntu:
  - Sistema de Caja.
  - Sistema de Inventario.
  - DB-Server.

También se incluye la captura original de la topología implementada en GNS3:

[Ver topología original en GNS3](Topologia/Captura%20de%20pantalla%202026-10-04%20210438.png)

---

## Direccionamiento IP

| Segmento / Dispositivo | Dirección IP | Máscara | Gateway |
|---|---|---|---|
| **VLAN 10 – Usuarios** | `10.21.40.0/25` | `255.255.255.128` | `10.21.40.1` |
| FortiGate VLAN10 | `10.21.40.1/25` | `255.255.255.128` | — |
| Windows VLAN10 | `10.21.40.10/25` | `255.255.255.128` | `10.21.40.1` |
| **VLAN 20 – Administración** | `10.21.40.128/25` | `255.255.255.128` | `10.21.40.129` |
| FortiGate VLAN20 | `10.21.40.129/25` | `255.255.255.128` | — |
| Windows VLAN20 | `10.21.40.140/25` | `255.255.255.128` | `10.21.40.129` |
| **DMZ – Servidores** | `10.21.41.0/28` | `255.255.255.240` | `10.21.41.1` |
| FortiGate DMZ-SW | `10.21.41.1/28` | `255.255.255.240` | — |
| Sistema de Caja | `10.21.41.2/28` | `255.255.255.240` | `10.21.41.1` |
| Sistema de Inventario | `10.21.41.3/28` | `255.255.255.240` | `10.21.41.1` |
| DB-Server | `10.21.41.4/28` | `255.255.255.240` | `10.21.41.1` |
| **WAN / port1** | DHCP desde Cloud1 | Dinámica | Gateway dinámico |

La documentación completa del direccionamiento se encuentra en:

[Direccionamiento/Direccionamiento-IP.md](Direccionamiento/Direccionamiento-IP.md)

---

## Configuración del FortiGate

### Interfaces principales

- **port1**: WAN, configurado por DHCP desde Cloud1.
- **port2**: interfaz física usada como trunk hacia el switch Cisco.
- **VLAN10** sobre port2:
  - VLAN ID: `10`
  - Gateway: `10.21.40.1/25`
- **VLAN20** sobre port2:
  - VLAN ID: `20`
  - Gateway: `10.21.40.129/25`
- **DMZ-SW**:
  - Gateway: `10.21.41.1/28`
  - Miembros: `port3`, `port4`, `port5`

### Servidores DMZ

| Puerto FortiGate | Servidor | IP | Servicio principal |
|---|---|---|---|
| port3 | Sistema de Caja | `10.21.41.2/28` | Nginx + SSH |
| port4 | Sistema de Inventario | `10.21.41.3/28` | Nginx + SSH |
| port5 | DB-Server | `10.21.41.4/28` | MariaDB + SSH |

### Políticas de seguridad implementadas

Entre las principales políticas configuradas se encuentran:

- **VLAN20_to_DMZ_SSH**: permite SSH desde VLAN 20 hacia la DMZ.
- **VLAN10_to_CAJA_WEB**: permite acceso HTTP/HTTPS desde VLAN 10 al Sistema de Caja.
- **VLAN10_INVENTARIO_WEBFILTER**: aplica filtrado web al acceso desde VLAN 10 hacia Inventario.
- **BLOQUEO_VLAN10_INVENTARIO**: política de denegación de respaldo para el servidor de Inventario.
- **DENY_VLAN10_SSH_DMZ**: bloquea SSH desde VLAN 10 hacia la DMZ.
- **DMZ_to_DNS**: permite únicamente consultas DNS hacia servidores autorizados.
- **DMZ_to_Ubuntu_Updates**: permite acceso HTTP/HTTPS a los repositorios de actualización de Ubuntu autorizados.
- **DENY_DMZ_TO_INTERNET**: bloquea el resto del tráfico de la DMZ hacia Internet.
- **DENY_DMZ_TO_VLAN10**: bloquea tráfico iniciado desde la DMZ hacia VLAN 10.
- **DENY_DMZ_TO_VLAN20**: bloquea tráfico iniciado desde la DMZ hacia VLAN 20.

### Web Filter

Se configuró el perfil:

`WF_BLOQUEO_INVENTARIO`

para bloquear el acceso al servidor:

`10.21.41.3`

desde VLAN 10. La evidencia muestra la página de FortiGuard indicando que el acceso fue bloqueado.

La documentación gráfica de la configuración del FortiGate está disponible en:

[Configuración FortiGate.pdf](Configuracion%20fortigate/Configuracion%20FortiGate.pdf)

---

## Configuración del switch Cisco

El switch actúa como dispositivo de capa 2 entre el FortiGate y las estaciones Windows.

### VLAN configuradas

- **VLAN 10** – Usuarios.
- **VLAN 20** – Administración.

### Interfaces

| Interfaz | Configuración | Destino |
|---|---|---|
| Gi0/0 | Trunk 802.1Q, VLAN 10 y 20 | FortiGate port2 |
| Gi0/1 | Access VLAN 20 | Windows VLAN20 |
| Gi0/2 | Access VLAN 10 | Windows VLAN10 |

También se configuraron medidas básicas de seguridad:

- Port Security.
- Sticky MAC.
- Máximo de una MAC por puerto de acceso.
- Violación en modo `restrict`.
- PortFast.
- BPDU Guard.
- SSH versión 2 para administración del switch.

El running-config del switch se encuentra en:

[Show running-config Switch.txt](Show%20running-config/Show%20running-config%20Switch.txt)

---

## Servidores

Los tres servidores utilizan **Ubuntu Server 24.04**.

### Sistema de Caja

- Hostname: `Sistema-Caja`
- IP: `10.21.41.2/28`
- Gateway: `10.21.41.1`
- Servicios:
  - Nginx
  - OpenSSH

[Ver comandos utilizados](Script%20Servidores/comandos-sistema-caja.txt)

### Sistema de Inventario

- Hostname: `Sistema-Inventario`
- IP: `10.21.41.3/28`
- Gateway: `10.21.41.1`
- Servicios:
  - Nginx
  - OpenSSH

[Ver comandos utilizados](Script%20Servidores/comandos-sistema-inventario.txt)

### DB-Server

- Hostname: `DB-Server`
- IP: `10.21.41.4/28`
- Gateway: `10.21.41.1`
- Servicios:
  - MariaDB
  - OpenSSH

[Ver comandos utilizados](Script%20Servidores/comandos-db-server.txt)

---

## Evidencias y pruebas realizadas

Las evidencias del laboratorio documentan, entre otras, las siguientes validaciones:

- VLAN 10 obtiene y utiliza el direccionamiento `10.21.40.0/25`.
- VLAN 20 utiliza el direccionamiento `10.21.40.128/25`.
- El trunk del switch transporta VLAN 10 y VLAN 20.
- Port Security se encuentra activo en los puertos de acceso.
- SSH funciona desde VLAN 20 hacia los servidores autorizados.
- SSH desde VLAN 10 hacia la DMZ es bloqueado.
- VLAN 10 puede acceder al Sistema de Caja.
- El acceso desde VLAN 10 al Sistema de Inventario es bloqueado mediante Web Filter.
- La DMZ no puede iniciar tráfico hacia VLAN 10 ni VLAN 20.
- Los servidores DMZ pueden realizar consultas DNS hacia destinos permitidos.
- Los servidores DMZ pueden acceder a endpoints autorizados de actualización de Ubuntu.
- El resto del acceso a Internet desde la DMZ es bloqueado.
- Nginx se encuentra activo en Caja e Inventario.
- MariaDB se encuentra activo en DB-Server.

El documento de evidencias se encuentra en:

[Evidencias/Evidencias.pdf](Evidencias/Evidencias.pdf)

---

## Running Config

Se incluyen las configuraciones actuales de los dispositivos principales:

- [FortiGate running-config](Show%20running-config/fortigate-running-config.txt)
- [Switch running-config](Show%20running-config/Show%20running-config%20Switch.txt)

> **Nota de seguridad:** antes de reutilizar estas configuraciones fuera del laboratorio, se recomienda eliminar o reemplazar cualquier contraseña, hash o valor cifrado presente en los archivos.

---

## Estructura del repositorio

```text
fortigate-dmz-vlan-lab/
├── README.md
├── Configuracion fortigate/
│   └── Configuracion FortiGate.pdf
├── Direccionamiento/
│   └── Direccionamiento-IP.md
├── Evidencias/
│   └── Evidencias.pdf
├── Script Servidores/
│   ├── comandos-db-server.txt
│   ├── comandos-sistema-caja.txt
│   └── comandos-sistema-inventario.txt
├── Show running-config/
│   ├── Show running-config Switch.txt
│   └── fortigate-running-config.txt
└── Topologia/
    ├── Captura de pantalla 2026-10-04 210438.png
    └── Diagrama Topología.png
```

---

## Tecnologías utilizadas

- GNS3
- FortiGate VM 7.0.9
- Cisco IOSvL2 15.2
- Windows 10
- Ubuntu Server 24.04
- Nginx
- MariaDB
- OpenSSH

---

## Objetivos cumplidos

El laboratorio demuestra la implementación de una arquitectura segmentada y protegida mediante un firewall FortiGate, aplicando controles de acceso entre VLAN y DMZ, políticas explícitas de denegación, filtrado web, control de administración mediante SSH y restricción de salida a Internet desde los servidores.

---

## Autor

**Carlos-2140**

Proyecto desarrollado como práctica de configuración y seguridad de redes con FortiGate, Cisco IOSvL2 y GNS3.
