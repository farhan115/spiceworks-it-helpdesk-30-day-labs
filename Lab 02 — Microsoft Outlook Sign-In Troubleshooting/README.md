# IT Help Desk Lab 02 — Microsoft Outlook Sign-In Troubleshooting

## Project Overview

This hands-on IT support lab simulates a help desk ticket involving a user who cannot access Microsoft Outlook using their organizational account.

The objective was to investigate authentication problems, troubleshoot user access in Microsoft Entra ID, verify password reset functionality, and document the findings using Spiceworks Cloud Help Desk.

The investigation identified a potential Microsoft 365 licensing issue that required escalation.

**Lab Type:** Simulated IT Support Scenario  
**Ticket Number:** #4  
**Ticket Priority:** High  
**Category:** Email  
**User:** Alex Brown  
**Ticket Status:** Unresolved — Escalation Required

---

## Technologies Used

- Spiceworks Cloud Help Desk
- Microsoft Entra ID
- Microsoft 365
- Microsoft Outlook on the Web
- Microsoft Entra Sign-in Logs
- Identity and Access Management (IAM)
- Web Browser Troubleshooting

## Problem Description

A simulated employee, Alex Brown, reported being unable to sign in to Microsoft Outlook using their work account.

The troubleshooting investigation involved checking the user's account status, reviewing authentication methods, resetting the password, testing sign-in functionality, and investigating Outlook Error 500.

## Troubleshooting Steps

### Step 1 — Create a Help Desk Ticket

Created Ticket #4 in Spiceworks Cloud Help Desk.

**Ticket Subject:** User cannot sign into Microsoft Outlook using their work account.

**Priority:** High  
**Category:** Email

The ticket was used to document the incident, troubleshooting actions, findings, and escalation requirements.

### Step 2 — Verify the User Account

Accessed Microsoft Entra Admin Center.

Navigation:

Microsoft Entra ID → Users → All Users → Alex Brown

Verified that the user account was enabled.

**Result:** Account was active.

### Step 3 — Review Sign-In Logs

Navigated to:

Microsoft Entra ID → Users → Alex Brown → Sign-in Logs

Reviewed available interactive sign-in records.

**Result:** No sign-in records appeared within the selected time windows during the initial investigation.

### Step 4 — Investigate Authentication Methods

Navigated to:

Microsoft Entra ID → Users → Alex Brown → Authentication Methods

Reviewed the user's registered authentication methods.

**Findings:**

- SMS was configured as the primary authentication method.
- The registered phone number was not functional in the lab environment.
- The Temporary Access Pass was expired.

This introduced a potential MFA verification issue during sign-in testing.

### Step 5 — Reset the User Password

Performed an administrative password reset through Microsoft Entra ID.

After completing the reset, tested the account using:

https://myaccount.microsoft.com

**Result:** Password reset successful. Alex Brown was able to sign in to Microsoft My Account.

This confirmed that the new credentials worked for the tested authentication flow.

### Step 6 — Test Microsoft Outlook

Attempted to access Outlook on the web:

https://outlook.office.com

**Result:** Outlook continued to display HTTP Error 500.

Successful authentication to Microsoft My Account did not restore Outlook access.

### Step 7 — Investigate Microsoft 365 Licensing

Navigated to:

Microsoft Entra ID → Users → Alex Brown → Licenses

**Finding:**

No license assignments were found for the user.

A missing Exchange Online license was identified as a possible explanation for unavailable mailbox access.

The exact cause of Outlook Error 500 was not confirmed.

### Step 8 — Attempt Microsoft 365 Administration

Attempted to access:

https://admin.microsoft.com

The Microsoft 365 Admin Center displayed a permissions error.

The lab environment did not provide the administrative access required to verify Exchange Online mailbox provisioning or assign the necessary licenses.

## Root Cause Analysis

**Confirmed findings:**

- The Microsoft Entra ID account was enabled.
- The password reset was successful.
- The user successfully authenticated to Microsoft My Account.
- Outlook continued to display Error 500.
- No Microsoft 365 license was assigned to the user.
- Microsoft 365 administration was unavailable in the lab environment.

**Suspected cause:**

Missing Exchange Online licensing or mailbox provisioning may have contributed to the Outlook access failure.

**Root cause status:** Not conclusively verified.

## Resolution and Escalation

The password-related portion of the incident was successfully addressed.

However, Microsoft Outlook access remained unavailable.

Because Microsoft 365 administrative permissions and licensing were unavailable, the incident could not be fully resolved within the lab.

**Recommended escalation:**

Microsoft 365 / Exchange Online Administrator

**Required follow-up:**

1. Verify Exchange Online license availability.
2. Confirm mailbox existence and provisioning.
3. Review Microsoft 365 service health.
4. Investigate Outlook Error 500 if the mailbox is properly provisioned.
5. Retest Outlook access after corrective action.

**Final Ticket Status:** Unresolved — Escalation Required

## Skills Demonstrated

- IT help desk ticket management
- Incident documentation
- Microsoft Entra ID user administration
- Password reset troubleshooting
- Authentication method investigation
- Microsoft Entra sign-in log analysis
- Microsoft 365 licensing investigation
- Outlook web access troubleshooting
- Problem isolation
- Tier 1 technical support escalation
- Identity and Access Management fundamentals

## Key Learning Outcomes

This lab demonstrated that successful account authentication does not necessarily guarantee access to Microsoft 365 applications.

It also reinforced the importance of verifying licensing, documenting troubleshooting actions, distinguishing confirmed findings from suspected causes, and escalating incidents when administrative access is insufficient.

An unresolved ticket can still represent a successful Tier 1 troubleshooting exercise when the investigation and escalation are properly documented.

## Lab Completion Summary

**Password Reset:** Completed  
**Account Authentication:** Verified  
**Outlook Access:** Not Restored  
**License Investigation:** Completed  
**Root Cause:** Unconfirmed  
**Escalation Documentation:** Completed  
**Lab Outcome:** Tier 1 Investigation Completed

---

*This project was completed in a simulated IT support environment for educational and professional portfolio development.*
