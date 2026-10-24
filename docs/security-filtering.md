# GPO Security Filtering

## Objective

Verify that a Group Policy Object applies only to computer accounts authorized by its security filter.

## Configuration

- GPO: `GPO_Lab_Bloqueo`
- Linked OU: `OU=Equipos,OU=Lab,DC=adlab,DC=test`
- Security group: `GG_GPO_Bloqueo`
- Authorized computer account: `CLIENT01$`
- Test computer account: `CLIENT02$`

The GPO was configured with the following permissions:

- `GG_GPO_Bloqueo`: Read and Apply Group Policy.
- `Authenticated Users`: Read permission only.

## Test Procedure

1. Confirm that `CLIENT01$` is a member of `GG_GPO_Bloqueo`.
2. Run `gpupdate /force` on the client.
3. Run `gpresult /r /scope computer` to inspect applied computer policies.
4. Verify that `GPO_Lab_Bloqueo` appears in the applied policies for CLIENT01.
5. Test CLIENT02 while it is in the same OU but is not a member of the security group.

## Observed Results

- CLIENT01 applied `GPO_Lab_Bloqueo` while its computer account was included in `GG_GPO_Bloqueo`.
- CLIENT02 was not a member of `GG_GPO_Bloqueo`, and `GPO_Lab_Bloqueo` did not appear in its applied GPO report while it was in the original `Equipos` OU. This result was consistent with the configured security filter.
- CLIENT02 was later moved to `OU=Equipos-Soporte,OU=Equipos-Oficina,DC=adlab,DC=test`. Its current policy results must be evaluated according to its current OU and the GPO links there.

## Key Learning

A GPO link determines where the policy is considered for processing, while security filtering determines which authorized users or computers can apply it. Read permission alone does not grant permission to apply a GPO.

## Validation Commands

Run these commands on the relevant domain client:

```powershell
gpupdate /force
gpresult /r /scope computer
```

## Scope

This exercise was performed in an isolated lab. The result describes the tested configuration and does not imply that every possible Group Policy permission combination was tested.
