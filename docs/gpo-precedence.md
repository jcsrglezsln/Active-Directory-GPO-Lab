# GPO Link Precedence

## Objective

Understand how Group Policy link order affects the effective configuration when multiple GPOs configure the same setting.

## Configuration

- Organizational Unit: `OU=Equipos,OU=Lab,DC=adlab,DC=test`
- GPOs: `GPO_Lab_Equipos` and `GPO_Lab_Bloqueo`
- Test client: `CLIENT01`

The lab used registry-based settings to make the effective policy result easy to inspect.

## Test Procedure

1. Configure the same registry value in two GPOs linked to the same OU.
2. Inspect the GPO link order on the OU.
3. Run `gpupdate /force` on CLIENT01.
4. Inspect the effective registry value on the client.
5. Change the GPO link order and repeat the validation.

## Expected Behavior

When multiple applicable GPOs configure the same setting, link order determines precedence among GPO links at the same container. Link order `1` has the highest precedence at that container.

The effective result can also be affected by inheritance, enforced links, security filtering, and whether the GPO actually configures the setting.

## Validation Commands

Run on a domain client:

```powershell
gpupdate /force
gpresult /r /scope computer
Get-ItemProperty -Path 'HKLM:\SOFTWARE\GPO-Lab'
```

To inspect link order, run on a machine with the Group Policy PowerShell module and suitable domain permissions:

```powershell
Get-GPInheritance -Target 'OU=Equipos,OU=Lab,DC=adlab,DC=test'
```

## Evidence

Add screenshots showing the OU link order and the corresponding effective registry value on CLIENT01.

## Scope and Limitations

Document the actual values and link order observed during the test. Do not infer precedence from the GPO names alone; verify the configuration and effective client result.
