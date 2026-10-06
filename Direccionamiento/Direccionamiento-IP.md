# Direccionamiento IP

Este documento resume el esquema de direccionamiento utilizado en el laboratorio **FortiGate DMZ VLAN Lab**, actualizado para la topología con **2 switches**: **SW-USUARIOS** y **SW-DMZ**.

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
- Switch: `SW-USUARIOS`
- Puerto de usuario: `Gi0/2`

### VLAN 20 – Administración

- Red: `10.21.40.128/25`
- Máscara: `255.255.255.128`
- Gateway: `10.21.40.129`
- Hosts válidos: `10.21.40.129 - 10.21.40.254`
- Broadcast: `10.21.40.255`
- Interfaz FortiGate: `VLAN20` sobre `port2`
- VLAN ID: `20`
- Switch: `SW-USUARIOS`
- Puerto de usuario: `Gi0/1`

### DMZ – Servidores

- Red: `10.21.41.0/28`
- Máscara: `255.255.255.240`
- Gateway: `10.21.41.1`
- Hosts válidos: `10.21.41.1 - 10.21.41.14`
- Broadcast: `10.21.41.15`
- Interfaz lógica FortiGate: `DMZ-SW`
- Miembros de `DMZ-SW`: `port3`, `port4`, `port5`
- Enlace físico usado hacia la DMZ: `FortiGate port3 -> SW-DMZ Gi0/0`
- VLAN de capa 2 en SW-DMZ: `VLAN 30 (DMZ)`

> **Importante:** VLAN 30 se utiliza únicamente para segmentación de capa 2 en el switch SW-DMZ. No crea una nueva subred IP. Los tres servidores continúan perteneciendo a la red `10.21.41.0/28`.

## Relación física de puertos

### FortiGate

| Puerto / Interfaz | Conexión |
|---|---|
| `port1` | Cloud1 / Internet |
| `port2` | Trunk hacia `SW-USUARIOS Gi0/0`, transportando VLAN 10 y VLAN 20 |
| `port3` | Enlace hacia `SW-DMZ Gi0/0` para la red DMZ |
| `port4` | Sin conexión física |
| `port5` | Sin conexión física |
| `DMZ-SW` | Interfaz lógica de la DMZ, gateway `10.21.41.1/28` |

### SW-USUARIOS

| Puerto | Configuración | Conexión |
|---|---|---|
| `Gi0/0` | Trunk 802.1Q, VLAN 10 y 20 | FortiGate `port2` |
| `Gi0/1` | Access VLAN 20 | Windows VLAN20 – `10.21.40.140/25` |
| `Gi0/2` | Access VLAN 10 | Windows VLAN10 – `10.21.40.10/25` |

### SW-DMZ

| Puerto | Configuración | Conexión |
|---|---|---|
| `Gi0/0` | Access VLAN 30 | FortiGate `port3` |
| `Gi0/1` | Access VLAN 30 | Sistema de Caja – `10.21.41.2/28` |
| `Gi0/2` | Access VLAN 30 | Sistema de Inventario – `10.21.41.3/28` |
| `Gi0/3` | Access VLAN 30 | DB Server – `10.21.41.4/28` |

## Esquema lógico

```text
Cloud1
  |
FortiGate port1
  |
  +-- port2 --> SW-USUARIOS Gi0/0
  |              |-- Gi0/1 --> VLAN20 --> 10.21.40.140/25
  |              `-- Gi0/2 --> VLAN10 --> 10.21.40.10/25
  |
  `-- port3 --> SW-DMZ Gi0/0
                 |-- Gi0/1 --> Sistema-Caja        10.21.41.2/28
                 |-- Gi0/2 --> Sistema-Inventario  10.21.41.3/28
                 `-- Gi0/3 --> DB-Server           10.21.41.4/28
```

> **Nota WAN:** La interfaz `port1` del FortiGate está configurada en modo DHCP, por lo que su dirección IP y gateway dependen de la red Cloud1 utilizada en GNS3.
