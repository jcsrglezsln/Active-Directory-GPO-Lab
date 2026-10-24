# GPO Link Precedence

## Objective

Understand how Group Policy link order affects the effective configuration when multiple GPOs configure the same registry value.

## Configuration

- Organizational Unit: `OU=Equipos,OU=Lab,DC=adlab,DC=test`
- GPOs: `GPO_Lab_Bloqueo` and `GPO_Lab_Equipos`
- Test client: `CLIENT01`
- Registry key: `HKLM\SOFTWARE\GPO-Lab`
- Registry value: `PruebaPrioridad`

## Test Procedure

1. Configure the same registry value in both GPOs linked to the same OU.
2. Inspect the GPO link order on the OU.
3. Refresh computer Group Policy on CLIENT01.
4. Inspect the effective registry value on the client.
5. Compare the effective value with the settings configured in each GPO.

## Observed Configuration

The current link configuration on the OU was verified with `Get-GPInheritance`:

| GPO | Link order | Enabled | Enforced | Configured value |
|---|---:|---|---|---|
| `GPO_Lab_Bloqueo` | 1 | True | False | `Politica-Especifica` |
| `GPO_Lab_Equipos` | 2 | True | False | `Politica-General` |

Both GPOs configure `PruebaPrioridad` as a string value under `HKLM\SOFTWARE\GPO-Lab`.

## Validation Results

On CLIENT01, the effective value was inspected before and after a computer policy refresh.

Command used:

```powershell
gpupdate /target:computer /force
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\GPO-Lab' -Name 'PruebaPrioridad'
```

The computer policy refresh completed successfully. The observed effective value after the refresh was:

```text
Politica-Especifica
```

This result is consistent with `GPO_Lab_Bloqueo` having link order 1 and `GPO_Lab_Equipos` having link order 2.

## Validation Commands

Inspect the effective registry value on CLIENT01:

```powershell
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\GPO-Lab' -Name 'PruebaPrioridad'
```

Inspect the configured value in each GPO on a system with the Group Policy PowerShell module and appropriate domain permissions:

```powershell
Get-GPRegistryValue -Name 'GPO_Lab_Bloqueo' -Key 'HKLM\SOFTWARE\GPO-Lab'
Get-GPRegistryValue -Name 'GPO_Lab_Equipos' -Key 'HKLM\SOFTWARE\GPO-Lab'
```

Inspect the GPO link order:

```powershell
Get-GPInheritance -Target 'OU=Equipos,OU=Lab,DC=adlab,DC=test' |
    Select-Object -ExpandProperty GpoLinks |
    Select-Object DisplayName, Order, Enabled, Enforced
```


## Evidence

The following screenshots in the repository README support this test:

- `screenshots/03-gpo-links.png` - GPO links and link order.
- `screenshots/05-client01-gpresult.png` - CLIENT01 applied Group Policy results.

The effective registry value after `gpupdate /target:computer /force` was also validated on CLIENT01 as `Politica-Especifica`. The command output documents the observed value.
