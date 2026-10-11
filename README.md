# Guide: Forcefully Canceling a Stuck Software Center
Follow these steps in exact order to stop a frozen operating system deployment and clear the stuck "Installing" visual loop.

## Step 0: Login as Admin

To run this whole process you need to be logged in using your admin account. 

1. Restart PC
2. Sign-in with the default user
3. Hold left shift key and sign-out
4. Sign-in with your credentials as the other user

## Step 1: Forcefully Stop the Core Deployment Windows Services

This breaks the underlying data pipeline feeding the installer.

1. Click the Start Menu, type cmd.
2. Right click Command Prompt and select Run as Administrator.
3. Run the following three commands one by one, pressing Enter after each:

   ```cmd
   net stop wuauserv
   net stop bits
   net stop cryptsvc
   ```

   Use code with caution.

## Step 2: Clear the Update Cache Files

This completely wipes out the temporary update container to ensure the system doesn't try to automatically pick up where it left off.

1. In the same Command Prompt window, type the following command and press Enter (it runs silently if successful):

   ```cmd
   rd /s /q %windir%\SoftwareDistribution\DataStore
   ```

   Use code with caution.

## Step 3: Switch to PowerShell and Drop the Stuck Task Request

Software Center uses its own database (WMI/CIM) to track progress. To drop the stuck task execution request across modern versions of PowerShell, you must use CIM instance commands.

1. Click the Start Menu, type powershell.
2. Right click Powershell and select Run as Administrator (the prompt changes to show a PS prefix).
2. Type this command to delete the stuck task request, then press Enter:

   ```powershell
   Get-CimInstance -Namespace root\ccm\SoftMgmtAgent -ClassName CCM_TSExecutionRequest | Remove-CimInstance
   ```

   Use code with caution.

## Step 4: Refresh the Software Center Background Agent

This forces the local Software Center client to apply the changes and pull a completely fresh status report.

1. Still in Powershell type the following command and press Enter:

   ```powershell
   Restart-Service ccmexec -Force
   ```

   Use code with caution.

2. Restart the PC and you should see that the Software Center says the Windows Update failed/installed and you can try installing it again. 
