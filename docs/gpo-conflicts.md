# GPO Conflicts and Enforced Links

## Objective

Understand how Group Policy settings from a parent OU and a child OU interact when both configure the same registry value.

## Lab Structure

- Parent OU: `OU=Equipos-Oficina,DC=adlab,DC=test`
- Child OU: `OU=Equipos-Soporte,OU=Equipos-Oficina,DC=adlab,DC=test`
- Test client: `CLIENT02`

## Configuration

The parent GPO, `GPO_Lab_Herencia`, was linked to `Equipos-Oficina`.

The child GPO, `GPO_Lab_Soporte`, was linked directly to `Equipos-Soporte`.

Both GPOs configured the same registry value:

- Registry key: `HKLM\SOFTWARE\GPO-Lab`
- Value name: `PruebaConflicto`
- Value type: `String`

The configured values were:

| GPO | Value |
|---|---|
| `GPO_Lab_Herencia` | `Valor-OU-Padre` |
| `GPO_Lab_Soporte` | `Valor-OU-Hija` |

## Test 1: Parent Link Not Enforced

With `Enforced` disabled on the parent GPO link, CLIENT02 returned:

`PruebaConflicto = Valor-OU-Hija`

The child OU setting took precedence over the conflicting setting inherited from the parent OU.

## Test 2: Parent Link Enforced

The parent GPO link was changed to `Enforced = True`.

After running `gpupdate /target:computer /force` on CLIENT02, the effective value changed to:

`PruebaConflicto = Valor-OU-Padre`

This demonstrated that the enforced parent GPO took precedence over the conflicting setting from the child OU.

## Restoration

The parent link was restored to `Enforced = False`.

The inheritance block on `Equipos-Soporte` was also disabled, leaving inheritance enabled.

## Key Takeaways

- GPO link precedence affects which conflicting setting is effective.
- A GPO linked to a child OU normally takes precedence over a conflicting GPO inherited from a parent OU.
- An enforced parent GPO link can override conflicting settings from lower-level GPOs.
- `gpresult` and registry inspection help validate the effective configuration on a client.

## Validation Commands

```powershell
gpupdate /target:computer /force
gpresult /scope computer /r

Get-ItemProperty `
  -Path "HKLM:\SOFTWARE\GPO-Lab" `
  -Name "PruebaConflicto"
```

## Scope

This exercise was performed in an isolated Hyper-V lab. The results describe the observed behavior in this environment.
