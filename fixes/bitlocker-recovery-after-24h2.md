# Fix: BitLocker Recovery Prompt on Every Boot After Windows 11 24H2 Upgrade

| | |
|---|---|
| **Category** | Security / Disk Encryption |
| **Platform** | Windows 11 24H2 with BitLocker and TPM |
| **Tools** | Command Prompt (Admin), `manage-bde` |
| **Time to resolve** | ~10 minutes, plus two restarts |

## Problem

After an in-place upgrade to Windows 11 24H2, the device asks for the BitLocker recovery key every time it starts.

## Symptoms

- The blue **BitLocker Recovery** screen appears at every startup or restart.
- Entering the 48-digit recovery key lets Windows load normally.
- The prompt comes back on the next boot, even though nothing else has changed.

## Root Cause

The TPM chip only releases the BitLocker key if the device's boot files match what it recorded when BitLocker was set up. The 24H2 upgrade replaces those boot files. On affected devices, BitLocker didn't refresh its TPM binding afterwards, so the TPM sees a mismatch on every boot and falls back to the recovery key.

## Confirming the Problem

1. Boot the device with the recovery key so Windows loads.
2. Open **Command Prompt** as administrator.
3. Run:

   ```cmd
   manage-bde -protectors -get C:
   ```

4. Find the **TPM** entry in the output and check its **PCR Validation Profile** line.

| What you see under TPM | Meaning |
|---|---|
| `PCR Validation Profile: 7, 11` | Healthy. This issue is not the cause. |
| PCR Validation Profile line missing or blank | **Problem confirmed.** Follow the resolution below. |

The device also counts as affected if it keeps prompting for recovery after more than one restart following the upgrade.

## Resolution

**Before you start:** have the device's 48-digit recovery key on hand, and check that the output above lists a **Numerical Password** protector. These steps don't decrypt the drive or change the recovery key.

In an elevated Command Prompt:

### Step 1: Remove the old TPM protector

```cmd
manage-bde -protectors -delete C: -type TPM
```

### Step 2: Add a fresh TPM protector

The new protector binds to the new boot files.

```cmd
manage-bde -protectors -add C: -tpm
```

### Step 3: Check the new protector

The TPM entry should now show **PCR Validation Profile: 7, 11**.

```cmd
manage-bde -protectors -get C:
```

### Step 4: Refresh policy and restart

```cmd
gpupdate /force
shutdown /r /t 0
```

## Verification

- The device boots to the Windows sign-in screen **without** the recovery prompt.
- Restart once more to be sure it stays fixed.

## Notes

- The Numerical Password protector is what lets you unlock the drive with the recovery key. Don't delete it. Steps 1 and 2 only replace the TPM protector.
- If the device is managed, make sure the recovery key is backed up to Active Directory or Entra ID/Intune before you change protectors.
