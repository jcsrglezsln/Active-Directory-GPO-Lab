# GPO Validation Procedures

## Purpose

This document describes the commands used to inspect Group Policy processing and verify the registry-based test settings in the Active Directory lab.

## Lab Machines

- `DC01`: domain controller and DNS server.
- `CLIENT01`: domain client in the original `Equipos` OU.
- `CLIENT02`: domain client in the nested `Equipos-Soporte` OU.

Domain: `adlab.test`

## 1. Refresh Group Policy

Run the following commands on the client being tested, from an elevated Command Prompt or PowerShell session:

```powershell
gpupdate /force
gpresult /r /scope computer
```

Review the applied computer policies and check whether the expected GPOs appear. A GPO being linked to an OU does not by itself prove that it applied; security filtering and other processing conditions also matter.

## 2. Inspect Test Registry Values

Run on `CLIENT01` or `CLIENT02`:

```powershell
Get-ItemProperty -Path 'HKLM:\SOFTWARE\GPO-Lab'
```

To inspect the inheritance test value:

```powershell
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\GPO-Lab' -Name 'PruebaHerencia'
```

To inspect the parent-child conflict test value:

```powershell
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\GPO-Lab' -Name 'PruebaConflicto'
```

These commands read the local registry on the computer where they run. They do not query the registry of another machine remotely.

## 3. Validate CLIENT01

CLIENT01 remains in `OU=Equipos,OU=Lab,DC=adlab,DC=test`.

Expected observations from the recorded lab tests:

- `GPO_Lab_Equipos` appeared in the applied computer policies.
- `GPO_Lab_Bloqueo` applied when the computer account was included in `GG_GPO_Bloqueo`.
- The link-precedence exercise was validated by comparing the effective test registry value after changing link order.

Use `gpresult` and the registry commands above to re-check the current state.

## 4. Validate CLIENT02

CLIENT02 is in `OU=Equipos-Soporte,OU=Equipos-Oficina,DC=adlab,DC=test`.

Expected observations from the recorded lab tests:

- `GPO_Lab_Soporte` and `GPO_Lab_Herencia` appeared in the applied computer policies.
- `PruebaHerencia` had the value `Heredada-de-Oficina`.
- `PruebaConflicto` had the value `Valor-OU-Hija` when the parent link was not enforced.
- During the enforced-link test, `PruebaConflicto` changed to `Valor-OU-Padre`.
- After restoring the parent link to non-enforced, the effective value returned to `Valor-OU-Hija`.

These are results observed during the lab exercises. Re-run the commands to verify the current state if policies have changed since then.

## 5. Inspect GPO Links on DC01

Run on DC01 in PowerShell with the Group Policy module and appropriate permissions:

```powershell
Get-GPInheritance -Target 'OU=Equipos,OU=Lab,DC=adlab,DC=test'

Get-GPInheritance -Target 'OU=Equipos-Oficina,DC=adlab,DC=test'

Get-GPInheritance -Target 'OU=Equipos-Soporte,OU=Equipos-Oficina,DC=adlab,DC=test'
```

Get-GPInheritance -Target 'OU=Equipos-Oficina,DC=adlab,DC=test' |
    Select-Object -ExpandProperty GpoLinks |
    Select-Object DisplayName, Order, Enabled, Enforced


Review direct links, inherited links, and whether inheritance is blocked. Check enforced status separately when needed:

```powershell
Get-GPO -Name 'GPO_Lab_Herencia' |
    Select-Object DisplayName, Id, GpoStatus
```

`Get-GPO` reports GPO metadata; inspect the link on the relevant OU in Group Policy Management or use the appropriate GPO link cmdlets to confirm link-specific settings such as enforcement.

## 6. Troubleshooting Checklist

If a GPO or test value is missing:

1. Confirm the client is joined to `adlab.test`.
2. Confirm the computer's current OU in Active Directory.
3. Run `gpupdate /force` and review any errors.
4. Run `gpresult /r /scope computer` to inspect applied and denied policies.
5. Check GPO links, security filtering, permissions, and inheritance.
6. Confirm that the GPO configures the registry value being inspected.
7. Check whether another applicable GPO configures the same setting.

## Scope and Limitations

This procedure documents a controlled learning lab. It does not establish production readiness, remote registry access, or successful testing across a production domain.

No passwords, authentication tokens, or other secrets should be stored in this repository.
