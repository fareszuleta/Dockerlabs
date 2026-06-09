# BigWear - DockerLabs Write-up

This repository contains my write-up for the **BigWear** machine from DockerLabs.

## Information

* **Platform:** DockerLabs
* **Machine:** BigWear
* **Difficulty:** Medium
* **Category:** Web / WordPress

## Summary

The machine involves exploiting a vulnerable WordPress installation. After enumerating plugins with WPScan, an outdated version of Pie Register was identified and exploited through **CVE-2025-34077**, resulting in administrator access. From there, a file management plugin was used to obtain remote code execution, followed by credential discovery and privilege escalation to root.

## Skills Practiced

* Nmap Enumeration
* WordPress Enumeration
* WPScan
* CVE Research
* Session Hijacking
* Remote Code Execution (RCE)
* Linux Enumeration
* Credential Discovery
* Privilege Escalation

## Files

* `BigWear.pdf` — Full write-up with screenshots and step-by-step explanation.

## Disclaimer

This write-up is intended for educational purposes only and was performed in a controlled lab environment provided by DockerLabs.
