# Fix: Software Center Stuck on "Installing"

| | |
|---|---|
| **Category** | Software Deployment / Windows Update |
| **Platform** | Windows 10/11 managed by Microsoft Configuration Manager (SCCM / MECM) |
| **Tools** | Command Prompt (Admin), PowerShell (Admin) |
| **Time to resolve** | ~15 minutes, plus a reboot |

## Problem

A Windows Update or OS deployment in Software Center froze partway through. Software Center kept showing "Installing" with no progress, and the user couldn't cancel or retry it.

## Symptoms

- Software Center shows "Installing" indefinitely with no progress
- No option to cancel or retry the deployment
- Restarting the PC alone doesn't clear the stuck status

## Root Cause

The deployment hung while the Windows Update services were still running and the update cache was half-written. The Configuration Manager client also kept a pending task request in its local WMI repository. Because of that request, Software Center kept trying to resume the same stuck job.

## Resolution

Follow these steps in this exact order.

### Step 0: Sign in as an admin

The whole process needs your admin account.

1. Restart the PC.
2. Sign in with the default user.
3. Hold the left **Shift** key and sign out.
4. Sign in with your admin credentials as **Other user**.

### Step 1: Stop the Windows Update services

This cuts off the data pipeline that feeds the installer.

1. Open the Start Menu and type `cmd`.
2. Right-click **Command Prompt** and select **Run as administrator**.
3. Run these commands one at a time:

   ```cmd
   net stop wuauserv
   net stop bits
   net stop cryptsvc
   ```

### Step 2: Clear the update cache

This deletes the temporary update data so the system can't resume where it left off.

1. In the same Command Prompt window, run the command below. It prints nothing if it succeeds.

   ```cmd
   rd /s /q %windir%\SoftwareDistribution\DataStore
   ```

### Step 3: Delete the stuck task request in PowerShell

Software Center tracks deployment progress in WMI/CIM. Use the CIM cmdlets to delete the stuck task request; they work in both Windows PowerShell and PowerShell 7.

1. Open the Start Menu and type `powershell`.
2. Right-click **PowerShell** and select **Run as administrator**. The prompt starts with `PS` once it's ready.
3. Run:

   ```powershell
   Get-CimInstance -Namespace root\ccm\SoftMgmtAgent -ClassName CCM_TSExecutionRequest | Remove-CimInstance
   ```

### Step 4: Restart the Configuration Manager client

This makes the client pick up the changes and send a fresh status report.

1. In the same PowerShell window, run:

   ```powershell
   Restart-Service ccmexec -Force
   ```

2. Restart the PC. The services stopped in Step 1 start again on their own.

## Verification

- After the reboot, Software Center shows the update as **Failed** or **Installed** instead of **Installing**.
- You can start the installation again from Software Center.