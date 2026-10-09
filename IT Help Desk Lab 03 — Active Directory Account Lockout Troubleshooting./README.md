# IT Help Desk Lab 03 — Active Directory Account Lockout Troubleshooting

## Project Overview

This hands-on IT support lab demonstrates how to investigate, reproduce, and troubleshoot an Active Directory account lockout using Windows Server, PowerShell, Event Viewer, and Spiceworks Cloud Help Desk.

The lab simulates an employee who cannot access their Windows domain account after multiple incorrect password attempts.

The objective was to identify the lockout policy, reproduce the problem in a controlled environment, examine Windows Security logs, identify the source of the lockout, and verify that the account returned to an unlocked state.

**Lab Type:** Simulated IT Help Desk Incident  
**Ticket:** #5  
**Category:** Account / Login Issue  
**Priority:** Medium  
**User:** John Smith  
**Username:** `jsmith`  
**Domain:** `farhanlab.local`  
**Domain Controller:** `WIN2K25-DC01`

---

## Technologies Used

- Windows Server 2025
- Active Directory Domain Services (AD DS)
- Active Directory Users and Computers (ADUC)
- Windows PowerShell
- Active Directory PowerShell Module
- Fine-Grained Password Policies (FGPP)
- Windows Event Viewer
- Windows Security Audit Logs
- Spiceworks Cloud Help Desk

## Lab Environment

| Component | Configuration |
|---|---|
| Active Directory Domain | `farhanlab.local` |
| Domain Controller | `WIN2K25-DC01` |
| Test User | John Smith |
| SAM Account Name | `jsmith` |
| Organizational Unit | `OU=IT,OU=IAM-Lab03` |
| Fine-Grained Password Policy | `Helpdesk-Lab03-Lockout` |
| Lockout Threshold | 5 attempts |
| Lockout Duration | 10 minutes |
| Observation Window | 10 minutes |
| Security Event | 4740 |

All testing was performed against a dedicated lab user account.

---

## Step 1 — Create a Help Desk Ticket

Created a simulated incident in Spiceworks Cloud Help Desk.

**Ticket Subject:** Active Directory User Account Locked Out

**Incident Description:**

Employee John Smith reports being unable to sign in to the Windows domain account due to an account lockout.

**Business Impact:** Employee cannot access their domain account or associated resources.

**Troubleshooting Objective:** Identify the cause of the account lockout, investigate authentication failures, restore account access, and document the resolution.

---

## Step 2 — Create the Active Directory Test User

Opened Active Directory Users and Computers.

Command:

```powershell
dsa.msc
```

Created the following user account:

| Attribute | Value |
|---|---|
| First Name | John |
| Last Name | Smith |
| User Logon Name | `jsmith` |
| Domain | `farhanlab.local` |
| Department / OU | IT |
| Account Enabled | Yes |

Configured a test password and enabled the account.

### Verify the Account Using PowerShell

```powershell
Import-Module ActiveDirectory
```

```powershell
Get-ADUser -Identity jsmith -Properties Enabled,LockedOut | Select-Object Name,SamAccountName,Enabled,LockedOut
```

**Observed output:**

```text
Name        SamAccountName Enabled LockedOut
----        -------------- ------- ---------
John Smith  jsmith         True    False
```

**Finding:** The account existed, was enabled, and was not initially locked.

---

## Step 3 — Check the Default Domain Lockout Policy

Executed:

```powershell
Get-ADDefaultDomainPasswordPolicy | Select-Object LockoutThreshold,LockoutDuration,LockoutObservationWindow
```

**Observed output:**

```text
LockoutThreshold LockoutDuration LockoutObservationWindow
---------------- --------------- ------------------------
0                00:10:00        00:10:00
```

### Interpretation

- `LockoutThreshold = 0` — Automatic account lockout was disabled under the default domain policy.
- `LockoutDuration = 10 minutes` — Configured lockout duration.
- `LockoutObservationWindow = 10 minutes` — Configured observation window.

Because the threshold was zero, a separate Fine-Grained Password Policy was required to simulate the account lockout.

---

## Step 4 — Check for an Existing Fine-Grained Password Policy

Executed:

```powershell
Get-ADUserResultantPasswordPolicy -Identity jsmith
```

**Observed result:** No policy was returned.

This indicated that no Fine-Grained Password Policy was effective for the test account at that time.

---

## Step 5 — Create a Fine-Grained Password Policy

Created a dedicated password policy for the lab account rather than modifying the default domain password policy.

```powershell
New-ADFineGrainedPasswordPolicy `
-Name "Helpdesk-Lab03-Lockout" `
-Precedence 10 `
-MinPasswordLength 8 `
-ComplexityEnabled $true `
-PasswordHistoryCount 5 `
-MinPasswordAge "00:00:00" `
-MaxPasswordAge "90.00:00:00" `
-LockoutThreshold 5 `
-LockoutDuration "00:10:00" `
-LockoutObservationWindow "00:10:00"
```

### Policy Configuration

| Setting | Value |
|---|---|
| Name | Helpdesk-Lab03-Lockout |
| Precedence | 10 |
| Minimum Password Length | 8 |
| Password Complexity | Enabled |
| Password History | 5 |
| Minimum Password Age | 0 |
| Maximum Password Age | 90 days |
| Lockout Threshold | 5 |
| Lockout Duration | 10 minutes |
| Observation Window | 10 minutes |

### Assign the Policy to John Smith

```powershell
Add-ADFineGrainedPasswordPolicySubject `
-Identity "Helpdesk-Lab03-Lockout" `
-Subjects "jsmith"
```

### Verify the Effective Policy

```powershell
Get-ADUserResultantPasswordPolicy -Identity jsmith | Select-Object Name,LockoutThreshold,LockoutDuration,LockoutObservationWindow
```

**Observed output:**

```text
Name                    LockoutThreshold LockoutDuration LockoutObservationWindow
----                    ---------------- --------------- ------------------------
Helpdesk-Lab03-Lockout  5                00:10:00        00:10:00
```

### Verify the Policy Assignment

```powershell
Get-ADFineGrainedPasswordPolicySubject -Identity "Helpdesk-Lab03-Lockout"
```

**Observed details:**

```text
Name               : John Smith
ObjectClass        : user
SamAccountName     : jsmith
DistinguishedName  : CN=John Smith,OU=IT,OU=IAM-Lab03,DC=farhanlab,DC=local
```

**Finding:** The Fine-Grained Password Policy was successfully assigned and effective for John Smith.

---

## Step 6 — Attempt Authentication Using RUNAS

Initially attempted to test authentication with:

```powershell
runas /user:farhanlab\jsmith cmd
```

**Observed error:**

```text
RUNAS ERROR: Unable to run - cmd
1385: Logon failure: the user has not been granted the requested logon type at this computer.
```

### Troubleshooting Analysis

Error 1385 indicated that the user was not permitted to perform the requested logon type on the server.

The test was being performed on a domain controller, where ordinary domain users should not be granted local interactive logon privileges.

**Action Taken:** Discontinued the RUNAS approach and used a different authentication testing method.

**Lesson Learned:** Logon-rights errors are different from incorrect-password errors.

---

## Step 7 — Attempt Authentication Using a Network Share

Created a test credential:

```powershell
$credential = Get-Credential -UserName "FARHANLAB\jsmith" -Message "Lab 3 - Enter an incorrect test password"
```

Attempted authentication using a network share:

```powershell
try {
    $null = New-PSDrive -Name "LabTest" -PSProvider FileSystem -Root "\\localhost\SYSVOL" -Credential $credential -ErrorAction Stop
    Write-Host "Authentication succeeded"
    Remove-PSDrive -Name "LabTest"
}
catch {
    Write-Host "Authentication test failed:" $_.Exception.Message
}
```

**Observed output:**

```text
Authentication succeeded
```

Checked the failed-password counter:

```powershell
Get-ADUser -Identity jsmith -Properties LockedOut,badPwdCount | Select-Object Name,LockedOut,badPwdCount
```

**Observed output:**

```text
Name        LockedOut badPwdCount
----        --------- -----------
John Smith  False     0
```

### Troubleshooting Analysis

The network share operation succeeded, but the failed-password counter remained at zero.

The test therefore did not demonstrate a failed authentication against the target account.

The result may have been affected by existing session credentials or how the local network share connection was handled.

**Action Taken:** Switched to direct Active Directory credential validation.

---

## Step 8 — Validate Credentials Against Active Directory

Loaded the required .NET assembly:

```powershell
Add-Type -AssemblyName System.DirectoryServices.AccountManagement
```

Created a domain authentication context:

```powershell
$context = New-Object System.DirectoryServices.AccountManagement.PrincipalContext(
    [System.DirectoryServices.AccountManagement.ContextType]::Domain,
    "farhanlab.local"
)
```

Entered an intentionally incorrect password for the test account:

```powershell
$password = Read-Host "Enter an incorrect LAB password"
```

Validated the credentials:

```powershell
$result = $context.ValidateCredentials("jsmith", $password)
```

Displayed the result:

```powershell
Write-Host "Authentication successful: $result"
```

**Observed output:**

```text
Authentication successful: False
```

### Verify the Failed-Password Counter

```powershell
Get-ADUser -Identity jsmith -Properties LockedOut,badPwdCount | Select-Object Name,LockedOut,badPwdCount
```

**Observed output:**

```text
Name        LockedOut badPwdCount
----        --------- -----------
John Smith  False     1
```

**Finding:** Active Directory rejected the incorrect password and incremented the failed-password counter.

---

## Step 9 — Simulate the Account Lockout

Repeated incorrect-password authentication attempts against the dedicated test account.

Commands:

```powershell
$password = Read-Host "Enter an incorrect LAB password"
```

```powershell
$context.ValidateCredentials("jsmith", $password)
```

Repeated the test until the configured lockout threshold was reached.

### Verify Account Lockout

```powershell
Get-ADUser -Identity jsmith -Properties LockedOut,badPwdCount | Select-Object Name,LockedOut,badPwdCount
```

**Observed output:**

```text
Name        LockedOut badPwdCount
----        --------- -----------
John Smith  True      5
```

**Finding:** Active Directory successfully locked the account after five failed password attempts.

This confirmed that the Fine-Grained Password Policy was functioning as intended.

---

## Step 10 — Investigate the Account Lockout in Event Viewer

Opened Windows Event Viewer:

```powershell
eventvwr.msc
```

Navigated to:

Windows Logs → Security → Filter Current Log

Filtered for:

```text
Event ID: 4740
```

### Initial Investigation

The Event Viewer filter initially returned zero matching events.

The account lockout had already been confirmed through PowerShell, so the investigation continued using direct Security log queries.

### Verify Security Audit Policy

Executed:

```powershell
auditpol /get /subcategory:"User Account Management"
```

**Observed output:**

```text
Category/Subcategory          Setting
--------------------          -------
User Account Management       Success
```

This confirmed that successful User Account Management auditing was enabled.

---

## Step 11 — Find Event ID 4740 Using PowerShell

Executed:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4740
} -MaxEvents 10 | Format-List TimeCreated,Id,Message
```

**Observed result:**

```text
TimeCreated : 10/9/2026 4:22:17 PM
Id          : 4740
Message     : A user account was locked out.

Account Name        : jsmith
Account Domain      : FARHANLAB
Caller Computer Name: WIN2K25-DC01
```

### Event Analysis

| Event Field | Observed Value |
|---|---|
| Event ID | 4740 |
| Event Description | A user account was locked out |
| Locked Account | jsmith |
| Account Domain | FARHANLAB |
| Caller Computer | WIN2K25-DC01 |
| Date | October 9, 2026 |
| Time | 4:22:17 PM |

**Finding:** Windows Security logs confirmed that John Smith's account was locked out.

The caller computer was identified as `WIN2K25-DC01`, consistent with the controlled authentication tests performed on the lab domain controller.

### Additional Authentication Events

These commands can be used for further investigation:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4771,4776
} -MaxEvents 10 | Format-List TimeCreated,Id,Message
```

Relevant Event IDs:

| Event ID | Description |
|---|---|
| 4740 | User account locked out |
| 4771 | Kerberos pre-authentication failed |
| 4776 | Domain controller credential validation using NTLM |

These additional events were included as optional investigation commands; their results were not documented in this lab.

---

## Step 12 — Verify Account Unlock Status

After the account lockout, checked the account again:

```powershell
Get-ADUser -Identity jsmith -Properties LockedOut,badPwdCount | Select-Object Name,LockedOut,badPwdCount
```

**Observed output:**

```text
Name        LockedOut badPwdCount
----        --------- -----------
John Smith  False     5
```

**Finding:** The account was no longer locked.

The Fine-Grained Password Policy configured a 10-minute lockout duration, so automatic unlock after that period was a possible explanation.

The screenshot did not establish whether the account unlocked automatically or through a manual administrative action.

### Optional Manual Unlock Command

If an account remains locked and the administrator has authorization to restore access:

```powershell
Unlock-ADAccount -Identity jsmith
```

This command was not confirmed as executed during this lab.

### Verify Account Status

```powershell
Get-ADUser -Identity jsmith -Properties Enabled,LockedOut,badPwdCount | Select-Object Name,Enabled,LockedOut,badPwdCount
```

### Verify Authentication Using the Correct Password

```powershell
Add-Type -AssemblyName System.DirectoryServices.AccountManagement

$context = New-Object System.DirectoryServices.AccountManagement.PrincipalContext(
    [System.DirectoryServices.AccountManagement.ContextType]::Domain,
    "farhanlab.local"
)

$securePassword = Read-Host "Enter John's correct test password" -AsSecureString

$credential = [System.Net.NetworkCredential]::new("", $securePassword)

$context.ValidateCredentials("jsmith", $credential.Password)
```

**Expected result:**

```text
True
```

**Verification status:** Successful authentication using the correct password has not yet been confirmed in the provided lab evidence.

---

## Root Cause Analysis

**Confirmed Root Cause:**

The test account was locked after five deliberately incorrect password attempts.

The lockout occurred because the Fine-Grained Password Policy configured a threshold of five failed authentication attempts.

**Supporting Evidence:**

1. Effective FGPP showed a threshold of five.
2. Incorrect credentials returned `False`.
3. The failed-password counter increased.
4. `LockedOut` changed to `True`.
5. Windows Security Event ID 4740 confirmed the lockout.
6. The event identified the caller computer.

The root cause was intentionally reproduced as part of the simulated help desk incident.

---

## Spiceworks Ticket Documentation

**Ticket:** #5  
**Subject:** Active Directory User Account Locked Out  
**User:** John Smith  
**Category:** Account / Login Issue  
**Priority:** Medium

### Troubleshooting Summary

- Verified the account was enabled.
- Checked the default domain lockout policy.
- Created a Fine-Grained Password Policy for the test account.
- Simulated incorrect-password attempts.
- Confirmed the account was locked after five attempts.
- Investigated Security Event ID 4740.
- Identified the caller computer.
- Confirmed the account subsequently returned to an unlocked state.

### Resolution Status

**Account Lockout:** Cleared  
**Root Cause:** Identified  
**Event Log Investigation:** Completed  
**Successful Authentication Retest:** Pending

**Ticket Recommendation:** Keep the ticket open until the correct password is tested successfully and access is verified.

---

## Troubleshooting Challenges and Lessons Learned

### Challenge 1 — PowerShell Pipeline Syntax

Encountered:

```text
An empty pipe element is not allowed.
```

**Cause:** A pipeline operator was entered without a following command in the same submitted statement.

**Solution:** Executed the complete pipeline together.

```powershell
Get-ADUser -Identity jsmith -Properties LockedOut | Select-Object Name,LockedOut
```

### Challenge 2 — RUNAS Error 1385

The test account lacked the requested local logon right on the domain controller.

**Lesson:** Do not grant ordinary users interactive logon rights on a domain controller merely to test authentication.

### Challenge 3 — Network Share Authentication Test

The network share command reported success without increasing `badPwdCount`.

**Lesson:** A successful command does not always prove that the intended account credentials were validated.

### Challenge 4 — Event Viewer Returned No Results

The graphical Security log filter initially returned zero events.

**Solution:** Queried the Security event log directly using `Get-WinEvent`, successfully locating Event ID 4740.

### Challenge 5 — Account Unlock Timing

The account was initially locked but later appeared unlocked.

**Lesson:** Lockout duration and elapsed time must be considered when investigating account status.

---

## Skills Demonstrated

- Active Directory user administration
- Windows Server administration
- PowerShell troubleshooting
- Fine-Grained Password Policy configuration
- Account lockout policy analysis
- Failed authentication testing
- Account lockout investigation
- Windows Security Event ID 4740 analysis
- Audit policy verification
- Root cause analysis
- Help desk ticket documentation
- Incident escalation and resolution verification
- Identity and Access Management (IAM)

---

## Suggested GitHub Screenshots

Store screenshots in a `screenshots` folder.

```text
Lab-03-AD-Account-Lockout/
│
├── README.md
│
└── screenshots/
    ├── 01-active-directory-user.png
    ├── 02-user-verification.png
    ├── 03-default-lockout-policy.png
    ├── 04-fgpp-configuration.png
    ├── 05-fgpp-assignment.png
    ├── 06-runas-error-1385.png
    ├── 07-failed-authentication.png
    ├── 08-account-locked.png
    ├── 09-event-viewer.png
    ├── 10-security-event-4740.png
    └── 11-account-unlocked.png
```

Before publishing, redact sensitive information such as passwords, tokens, and unnecessary security identifiers.

---

## Lab Results

| Objective | Status |
|---|---|
| Create AD test user | Completed |
| Verify account status | Completed |
| Review default password policy | Completed |
| Configure Fine-Grained Password Policy | Completed |
| Apply FGPP to test user | Completed |
| Simulate failed authentication | Completed |
| Trigger account lockout | Completed |
| Investigate Event ID 4740 | Completed |
| Identify caller computer | Completed |
| Verify unlocked account status | Completed |
| Verify correct-password sign-in | Pending |
| Close Spiceworks ticket | Pending |

## Conclusion

This lab provided practical experience investigating an Active Directory account lockout in a Windows Server domain environment.

I successfully configured a Fine-Grained Password Policy, reproduced a user account lockout, examined authentication behavior, and identified the lockout event using Windows Security logs.

The exercise strengthened my understanding of Active Directory account management, PowerShell administration, Windows authentication, audit logging, and Tier 1/Tier 2 help desk troubleshooting.

**Final Outcome:** Account lockout investigation completed successfully. Final authentication verification remains pending.

---

*This project was completed in an isolated Active Directory lab environment for educational purposes and professional IT support/IAM portfolio development.*
