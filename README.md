# Active Directory Group Policy Lab

## Overview

This project documents a hands-on Active Directory Group Policy (GPO) lab built with Hyper-V virtual machines. It explores Group Policy inheritance, link precedence, security filtering, and enforced policies.

## Lab Environment

| Machine | Role | Configuration |
|---|---|---|
| DC01 | Domain Controller | Windows Server 2025 Standard Evaluation; AD DS and DNS |
| CLIENT01 | Domain Client | Windows client joined to `adlab.test` |
| CLIENT02 | Domain Client | Windows client joined to `adlab.test` |

- Domain: `adlab.test`
- NetBIOS domain name: `ADLAB`
- Lab network: `192.168.50.0/24`
- DC01 IP: `192.168.50.10`
- CLIENT01 IP: `192.168.50.20`
- CLIENT02 IP: `192.168.50.21`

## Organizational Units

- `OU=Equipos,OU=Lab,DC=adlab,DC=test`
- `OU=Equipos-Oficina,DC=adlab,DC=test`
- `OU=Equipos-Soporte,OU=Equipos-Oficina,DC=adlab,DC=test`

CLIENT01 remains in the original `Equipos` OU. CLIENT02 was moved to `Equipos-Soporte` to test Group Policy inheritance and conflicts between parent and child OUs.

## Group Policy Objects

- `GPO_Lab_Equipos`
- `GPO_Lab_Bloqueo`
- `GPO_Lab_Herencia`
- `GPO_Lab_Soporte`

## Lab Exercises

### 1. GPO Link Precedence

Configured the same registry value in two GPOs linked to the same OU. Changed the link order and observed the effective value on CLIENT01.

### 2. Security Filtering

Configured `GPO_Lab_Bloqueo` to apply to computer accounts in `GG_GPO_Bloqueo`. Verified that CLIENT01 was a member of the group and applied the GPO. CLIENT02 was not a member of the group, and the GPO did not appear in its applied GPO report while it was in the original `Equipos` OU. CLIENT02 was subsequently moved to `Equipos-Soporte`, where its current policy results depend on the GPO links and security permissions applicable to that OU.

### 3. GPO Inheritance

Linked `GPO_Lab_Herencia` to `Equipos-Oficina` and verified that CLIENT02 received a test registry value from the parent OU.

### 4. Block Inheritance

Enabled Block Inheritance on `Equipos-Soporte` and used `gpresult` to validate the effect on inherited, non-enforced GPOs. The inheritance configuration was subsequently restored for the final parent-child conflict tests.

### 5. Enforced GPOs

Marked `GPO_Lab_Herencia` as Enforced and verified that it continued to apply across the inheritance block. The link was then returned to its non-enforced state.

### 6. Conflicting GPO Settings

Configured `PruebaConflicto` with different values in `GPO_Lab_Herencia` and `GPO_Lab_Soporte`. Observed `Valor-OU-Hija` when the parent link was not enforced and `Valor-OU-Padre` when it was enforced. After restoring the non-enforced state, CLIENT02 returned to `Valor-OU-Hija`.

## Key Findings

- **Link precedence:** the effective setting changed when the GPO link order changed.
- **Security filtering:** `GPO_Lab_Bloqueo` applied to CLIENT01 when its computer account was authorized through `GG_GPO_Bloqueo`.
- **GPO inheritance:** CLIENT02 received a test registry value from the parent OU.
- **Enforced GPO:** the enforced parent link affected the inheritance and conflict tests. Its link was restored to non-enforced.
- **Parent-child conflict:** the child OU value won when the parent link was not enforced; the parent value won when its link was enforced.

## Evidence

| Evidence | Screenshot |
|---|---|
| Hyper-V private network | [01-hyperv-network.png](screenshots/01-hyperv-network.png) |
| Active Directory OU structure | [02-active-directory-ous.png](screenshots/02-active-directory-ous.png) |
| GPO links | [03-gpo-links.png](screenshots/03-gpo-links.png) |
| Security filtering | [04-security-filtering.png](screenshots/04-security-filtering.png) |
| CLIENT01 policy results | [05-client01-gpresult.png](screenshots/05-client01-gpresult.png) |
| CLIENT02 policy results | [06-client02-gpresult.png](screenshots/06-client02-gpresult.png) |
| GPO inheritance | [07-gpo-inheritance.png](screenshots/07-gpo-inheritance.png) |
| Conflicting GPO settings | [08-gpo-conflict.png](screenshots/08-gpo-conflict.png) |
| CLIENT02 registry validation | [09-client02-registry-validation.png](screenshots/09-client02-registry-validation.png) |
| CLIENT02 final validation after Group Policy refresh | [10-client02-final-validation.png](screenshots/10-client02-final-validation.png) |


## Technical Documentation

- [Lab Architecture](docs/lab-architecture.md) — Lab network, virtual machines, and Active Directory structure.
- [GPO Security Filtering](docs/security-filtering.md) — Security group filtering and observed results.
- [GPO Link Precedence](docs/gpo-precedence.md) — Link order and effective policy settings.
- [GPO Conflicts](docs/gpo-conflicts.md) — Parent and child OU policy conflicts.
- [GPO Validation Procedures](docs/validation-procedures.md) — Commands to inspect applied policies and registry values.

## Validation Tools

- PowerShell GroupPolicy module
- Active Directory PowerShell module
- `Get-GPInheritance`
- `Get-GPO`
- `Get-GPRegistryValue`
- `Set-GPRegistryValue`
- `Set-GPLink`
- `Set-GPInheritance`
- `gpupdate`
- `gpresult`
- Windows Registry inspection

## Scope and Limitations

This is an isolated learning lab, not a production deployment. Results were validated in the described virtual environment. Remote administration and production-scale behavior were not evaluated as part of these exercises.

## Disclaimer

All names, addresses, and configurations in this repository are for a private lab environment.
