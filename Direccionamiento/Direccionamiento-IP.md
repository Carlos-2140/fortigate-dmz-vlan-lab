# Direccionamiento IP

Este documento resume el esquema de direccionamiento utilizado en el laboratorio **FortiGate DMZ VLAN Lab**.

## Tabla de direccionamiento

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
| DB Server | `10.21.41.4/28` | `255.255.255.240` | `10.21.41.1` |
| **WAN / port1** | DHCP desde Cloud1 | Dinámica | Gateway dinámico |

## Resumen por subred

### VLAN 10 – Usuarios
- Red: `10.21.40.0/25`
- Máscara: `255.255.255.128`
- Gateway: `10.21.40.1`
- Hosts válidos: `10.21.40.1 - 10.21.40.126`
- Broadcast: `10.21.40.127`
- Interfaz FortiGate: `VLAN10` sobre `port2`
- VLAN ID: `10`

### VLAN 20 – Administración
- Red: `10.21.40.128/25`
- Máscara: `255.255.255.128`
- Gateway: `10.21.40.129`
- Hosts válidos: `10.21.40.129 - 10.21.40.254`
- Broadcast: `10.21.40.255`
- Interfaz FortiGate: `VLAN20` sobre `port2`
- VLAN ID: `20`

### DMZ – Servidores
- Red: `10.21.41.0/28`
- Máscara: `255.255.255.240`
- Gateway: `10.21.41.1`
- Hosts válidos: `10.21.41.1 - 10.21.41.14`
- Broadcast: `10.21.41.15`
- Interfaz FortiGate: `DMZ-SW`
- Miembros del software switch: `port3`, `port4`, `port5`

## Relación de puertos

| Equipo / Puerto | Conexión |
|---|---|
| FortiGate `port1` | Cloud1 / Internet |
| FortiGate `port2` | Trunk hacia Cisco IOSvL2 `Gi0/0` con VLAN 10 y VLAN 20 |
| Cisco IOSvL2 `Gi0/1` | Acceso VLAN 20 |
| Cisco IOSvL2 `Gi0/2` | Acceso VLAN 10 |
| FortiGate `port3` | Sistema de Caja – `10.21.41.2` |
| FortiGate `port4` | Sistema de Inventario – `10.21.41.3` |
| FortiGate `port5` | DB Server – `10.21.41.4` |

> **Nota:** La interfaz WAN `port1` del FortiGate está configurada en modo DHCP, por lo que su dirección IP y gateway dependen de la red Cloud1 utilizada en GNS3.
