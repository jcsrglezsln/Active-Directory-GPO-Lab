# Lab Architecture

## Purpose

This document describes the isolated Hyper-V environment used to practice Active Directory Group Policy management.

## Network

The lab uses a private Hyper-V virtual switch named `AD-Lab-Private`.

| Setting | Value |
|---|---|
| Network | `192.168.50.0/24` |
| Switch type | Private |
| Domain | `adlab.test` |
| DNS server | `192.168.50.10` |

The private switch isolates communication between the lab virtual machines from the host and external networks.

## Virtual Machines

| Machine | Role | IP address | Operating system |
|---|---|---|---|
| DC01 | Domain Controller and DNS | `192.168.50.10` | Windows Server 2025 Standard Evaluation |
| CLIENT01 | Domain client | `192.168.50.20` | Windows client evaluation |
| CLIENT02 | Domain client | `192.168.50.21` | Windows client evaluation |

The clients use DC01 as their DNS server.

## Active Directory Structure

```text
adlab.test
├── OU=Lab
│   ├── OU=Usuarios
│   └── OU=Equipos
└── OU=Equipos-Oficina
    └── OU=Equipos-Soporte
```

CLIENT01 remains in `OU=Equipos,OU=Lab`.

CLIENT02 was moved to `OU=Equipos-Soporte,OU=Equipos-Oficina` to test Group Policy inheritance and conflicts between parent and child OUs.

## Group Policy Objects

| GPO | Link location | Purpose |
|---|---|---|
| `GPO_Lab_Equipos` | `OU=Equipos,OU=Lab` | General computer policy testing |
| `GPO_Lab_Bloqueo` | `OU=Equipos,OU=Lab` | Security filtering and screen-lock testing |
| `GPO_Lab_Herencia` | `OU=Equipos-Oficina` | Parent OU inheritance and precedence tests |
| `GPO_Lab_Soporte` | `OU=Equipos-Soporte` | Child OU conflict testing |

## Safety and Scope

This is a learning environment using virtual machines and test registry values. It is not a production deployment.

No passwords or authentication secrets should be stored in this repository.
