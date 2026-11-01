# IT Support Fix Log

A collection of problems I've troubleshot and fixed, written up as step-by-step guides.

Each entry records the problem, the symptoms, the root cause, the exact fix, and how I confirmed it worked. That's the same structure I'd use for a help desk ticket or a knowledge base article.

## Skills Demonstrated

- **Windows administration**: services, system cache, local admin access
- **Command line**: Command Prompt and PowerShell (including CIM/WMI)
- **Endpoint management**: Microsoft Configuration Manager (SCCM / MECM) and Software Center
- **Security**: BitLocker and TPM protector management with `manage-bde`
- **Troubleshooting method**: isolate the cause, fix it in a safe order, and verify the result
- **Technical writing**: clear, repeatable documentation for end users and other technicians

## Fixes

| Fix | Category | Platform |
|---|---|---|
| [Software Center Stuck on "Installing"](fixes/software-center-stuck-install.md) | Software Deployment / Windows Update | Windows + SCCM/MECM |
| [BitLocker Recovery Prompt on Every Boot After 24H2 Upgrade](fixes/bitlocker-recovery-after-24h2.md) | Security / Disk Encryption | Windows 11 24H2 |

## How Each Fix Is Structured

1. **Problem**: what the user reported
2. **Symptoms**: what I observed
3. **Root Cause**: why it happened
4. **Resolution**: the exact steps and commands
5. **Verification**: how I confirmed it was fixed
6. **Notes**: caveats and follow-up checks

New entries follow the [fix template](fixes/_TEMPLATE.md).

## Disclaimer

These guides reflect environments I've worked in. Always test in your own environment, and follow your organization's change-management policies before you run admin commands on production machines.
