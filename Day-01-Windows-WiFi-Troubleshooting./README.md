# Day 01 — Windows 11 Wi-Fi Troubleshooting

**Platform:** Spiceworks Cloud Help Desk  
**Ticket ID:** #3  
**Category:** Network  
**Priority:** High  
**Status:** Closed  
**Lab Type:** Simulated IT Support Scenario

## Objective

Practice the IT service desk ticket lifecycle, including ticket creation, prioritization, assignment, technical troubleshooting, documentation, and closure.

## Problem Description

A fictional employee reported being unable to connect a Windows 11 laptop to the office Wi-Fi network. Other devices were reportedly connecting successfully.

## Troubleshooting Performed

I used Windows PowerShell on my local test computer to practice network diagnostics.

**Commands:**

```powershell
ipconfig /all
ping 8.8.8.8
nslookup google.com
```

## Diagnostic Results

- Windows detected multiple physical and virtual network adapters.
- Ping test: 4 packets sent, 4 received, 0% packet loss.
- Average latency: 17 ms.
- DNS lookup successfully resolved google.com.

These results confirmed that IP connectivity and DNS resolution were functioning on the local test computer. They did not establish the root cause of the fictional employee's issue.

## Simulated Resolution

The scenario assumed that an outdated saved Wi-Fi profile caused the connection problem.

The documented resolution involved forgetting the saved network, reconnecting with the correct credentials, and verifying connectivity.

This resolution was simulated for training purposes.

## Ticket Management

- Created a new Spiceworks support ticket.
- Assigned the ticket to myself.
- Set priority to High.
- Categorized the issue as Network.
- Added internal troubleshooting notes.
- Documented diagnostic results and the simulated resolution.
- Closed Ticket #3.

## Skills Demonstrated

- IT Help Desk ticket management
- Windows 11 troubleshooting
- TCP/IP and DNS fundamentals
- PowerShell network diagnostic commands
- Technical documentation
- Incident prioritization and resolution workflow

## Outcome

Successfully completed the simulated ticket lifecycle in Spiceworks and practiced documenting technical troubleshooting results.
