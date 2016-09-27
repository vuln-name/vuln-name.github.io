---
layout: post
author: thezero
title: "Shellshock"
category: vulnerability
date: "09/2014"
---

![Shellshock logo](/logos/Shellshock.png "Shellshock logo")

## Shellshock

Also known as Bashdoor, the name “Shellshock” is a bit of wordplay based on the fact that bash is a “shell”.
<!-- more -->

# CVEs (NVD/MITRE)
[CVE-2014-6271](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6271); ([CVE-2014-6277](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6277), [CVE-2014-6278](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6278), [CVE-2014-7169](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7169), [CVE-2014-7186](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7186), and [CVE-2014-7187](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7187))
<br/>**NOTE:** *Some vulnerability exists because of an incomplete fix for other ones.*

# Detection

**Detected by:** Stéphane Chazelas *(first Deteciton)*, David A. Wheeler and Norihiro Tanaka, Michał Zalewski, Tavis Ormandy, Florian Weimer and Todd Sabin *(Additional bug/issue/exploit)*

## Timeline
12/09/2014 - First Detection<br/>
24/09/2014 - Public Disclosure<br/>
26/09/2014 - Additional bug detected (on patched Systems) + Exploit released<br/>
27/09/2014 - Additional bug detected<br/>
30/09/2014 - Final patch released<br/>


# Description

## TL;DR
GNU Bash through 4.3 processes trailing strings after function definitions in the values of environment variables, which allows remote attackers to execute arbitrary code via a crafted environment, as demonstrated by vectors involving the ForceCommand feature in OpenSSH sshd, the mod\_cgi and mod\_cgid modules in the Apache HTTP Server, scripts executed by unspecified DHCP clients, and other situations in which setting the environment occurs across a privilege boundary from Bash execution, aka "ShellShock." NOTE: the original fix for this issue was incorrect; [CVE-2014-7169](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7169) has been assigned to cover the vulnerability that is still present after the incorrect fix for [CVE-2014-6271](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6271).

## Full Description
The vulnerability leverages the Bash shell, a command language interpreter. <br/>
An attacker can attach malicious code to environment variables that affect the way processes are run on a computer.

Shellshock is a **Privilege Escalation + OS Command Injection** vulnerability; it offers a way for users of a system to execute commands that should be unavailable to them. This happens through the "function export" feature, whereby command scripts created in one running instance of Bash can be shared with subordinate instances. This feature is implemented by encoding the scripts within a table that is shared between the instances, known as the environment variable list. Each new instance of Bash scans this table for encoded scripts, assembles each one into a command that defines that script in the new instance, and executes that command. The scripts are assumed to come from another instance, but the new instance cannot verify this, nor can it verify that the command that it has built is a properly formed script definition. Therefore, an attacker can execute arbitrary commands on the system or exploit other bugs that may exist in Bash's command interpreter, if the attacker has a way to manipulate the environment variable list and then cause Bash to run.

Also known as **Bashdoor**, it is a family of security bugs in the widely used Unix Bash shell, the first of which was disclosed on 24 September 2014. <br/>
On 12 September 2014 was discovered the original bug, which was called "Bashdoor" and soon a patch was released as well. The bug was assigned the CVE identifier CVE-2014-6271.<br/>
It was announced to the public on 24 September 2014 when Bash updates with the fix were ready for distribution.<br/>
The first bug causes Bash to unintentionally execute commands when the commands are concatenated to the end of function definitions stored in the values of environment variables.<br/>
Within days of the publication of this, intense scrutiny of the underlying design flaws discovered a variety of related vulnerabilities, (CVE-2014-6277, CVE-2014-6278, CVE-2014-7169, CVE-2014-7186, and CVE-2014-7187); addressed with a series of further patches.<br/>
Analysis of the source code history of Bash shows the vulnerabilities had existed since version 1.03 of Bash released in September 1989.<br/>

Attackers exploited Shellshock within hours of the initial disclosure by creating botnets of compromised computers to perform distributed denial-of-service attacks and vulnerability scanning.<br/>
Security companies recorded millions of attacks and probes related to the bug in the days following the disclosure.<br/>
Shellshock could potentially compromise millions of unpatched servers and other systems.

# CVSS (v2 base score)
10.0/10.0 HIGH

# Resources
[NVD NIST](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6271)<br/>
[CVE MITRE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271)<br/>
[https://shellshocker.net/ on the WayBackMachine](https://web.archive.org/web/20150913063755/https://shellshocker.net/)<br/>
[Official Shellshocker GitHub Repository](https://github.com/wreiske/shellshocker)<br/>