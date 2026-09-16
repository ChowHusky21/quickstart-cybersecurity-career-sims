# Career Simulation 3: Capture the Flag

**TL;DR:** Compromised a Windows CTF target over a local NAT network using the EternalBlue SMB exploit (CVE-2017-0144, CVSS 8.8), landed a Meterpreter shell, and captured the hidden `hoot.txt` flag from the target's desktop — then used Meterpreter's file-search capability to locate and exfiltrate an additional hidden file as a proof-of-concept for post-exploitation data access.

## Scenario

The objective was to penetrate a Windows machine on the local network and uncover a concealed flag by employing a range of penetration-testing methods to exploit vulnerabilities, secure access, and reveal hidden data.

> **Disclaimer (as stated in the original exercise):** *"This challenge is designed for educational purposes within a controlled environment. Ensure that you have the necessary permissions to perform penetration testing activities. Unauthorized access or exploitation of systems is strictly prohibited by law and outside of ethical boundaries."*

## Environment & Architecture

A NAT Network named **"Quickstart"** was created in VirtualBox, allowing multiple local VMs to share a single public-facing address for internet access while remaining on the same private segment. Two hosts were installed on it:

- **Kali-CTF** — `10.0.2.4` (attacker machine)
- **Windows-CTF** — `10.0.2.5` (target machine)

**No network diagram exists for this simulation** — unlike Sims 1 and 2, this exercise did not produce a Prezi/diagram artifact beyond the presentation slides themselves, so none is included here rather than reusing or fabricating one.

## Tools & Methodology

1. Confirmed the attacker VM's own network configuration on Kali-CTF with `ip addr` / `ifconfig`, verifying its address on the shared NAT segment (`10.0.2.4`).
2. Ran `nmap` against the NAT network (`-sS` SYN scan for stealthy port discovery, `-sV` for service/version detection, `--script vuln` to flag known vulnerabilities), identifying Windows-CTF (`10.0.2.5`) with SMB-related ports open (135, 139, 445, and others) and an SMB negotiation weakness flagged by Nmap's scripting engine.
3. Identified the flagged weakness as consistent with **EternalBlue (CVE-2017-0144, CVSS 8.8 High)** — a flaw in Microsoft's SMBv1 implementation that mishandles specially crafted packets, allowing an attacker to remotely execute code on the target. Microsoft shipped a patch for it in March 2017; this training target was deliberately left unpatched.
4. Exploited the vulnerability via **Metasploit**, landing a **Meterpreter** session on Windows-CTF.
5. From the Meterpreter session, used `shell` to drop into a standard command shell, `whoami` to confirm the active session's identity, `ipconfig` to confirm the compromised host's own network configuration, and `cd`/`type` to navigate the filesystem and read files.
6. Located and read the flag file directly on the target's desktop (`C:\Users\Ninja\Desktop\hoot.txt`).
7. Used Meterpreter's `search` command (`search -d C:\Users -f hashbrowns.txt`) to sweep the target's drives for additional files of interest, locating `hashbrowns.txt` and `hide.jpeg` under `C:\Users\Public\Documents`, and used the `download` command to pull `hide.jpeg` back to the attacker machine as a proof-of-concept for post-exploitation data exfiltration.

## Findings

- **EternalBlue — CVE-2017-0144 — CVSS 8.8 High.** A flaw in the SMBv1 server component of multiple Windows versions that mishandles specially crafted packets, allowing a remote, unauthenticated attacker to execute arbitrary code on the target. Microsoft patched this in its March 2017 security updates; the CTF target was intentionally left unpatched for this exercise.
- **Unrestricted filesystem access post-compromise.** Once a Meterpreter session was established, arbitrary files anywhere under the compromised user's reach (a hash-bearing text file, an image file hidden in a shared documents folder) were freely discoverable and downloadable — illustrating how a single unpatched network-facing service can cascade into full data exposure.

## Proof of Completion

- Captured flag file `hoot.txt`, found on the target's Desktop, containing the token **`RootFlag{061713fa2ad376430ac1555d1895f97876dc58f}`** — a lab-generated CTF flag value, not a real secret or credential.
- Meterpreter `whoami` / `ipconfig` output confirming an active, authenticated session on the Windows-CTF target.
- Successful `download` of `hide.jpeg` from `C:\Users\Public\Documents` back to the attacker machine, demonstrating working post-exploitation data access.

## Lessons Learned

- Unpatched SMBv1 remains a critical, high-impact exposure — applying the March 2017 Microsoft security updates (or disabling SMBv1 entirely where it isn't required) closes off the exact vulnerability class exploited here.

## Authorized Lab Environment Disclaimer

This simulation was conducted entirely within an isolated, sanctioned training environment as part of the QuickStart Cybersecurity Bootcamp (UC Santa Barbara Extension). The EternalBlue exploit and all post-exploitation techniques described were applied only against an intentionally vulnerable Windows training VM built for this exercise. None of the tools or techniques described here are authorized for use against any system without explicit permission from its owner.
