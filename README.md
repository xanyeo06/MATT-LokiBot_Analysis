# MATT — LokiBot Static and Dynamic Analysis

LokiBot is a commodity information-stealing Trojan for Windows that collects stored credentials from web browsers, mail clients, FTP clients and the Windows Credential Manager. This repository documents a static and dynamic analysis of a single Win32 sample, carried out in four stages: basic and advanced static analysis, then basic and advanced dynamic analysis. The work establishes that the file is unpacked at rest, that it resolves Windows APIs dynamically at runtime instead of through the import table, and that it attempts a direct HTTP connection to a hardcoded address with no preceding DNS lookup.

## Contents

| File | Description |
| --- | --- |
| `MATT-Report.docx` | Full analysis report |

## Sample

| Attribute | Value |
| --- | --- |
| Family | LokiBot |
| Category | Information-stealing Trojan (password stealer) |
| File type | Win32 executable |
| File name | `db400e3300adaedd5612bdc415b2c224c0ad7c59617f9c19ed5bb0b1907a998d.exe` |
| MD5 | `610cd392b20787dafc7ea94d151c760b` |
| Packed | No — no packer signature detected |

The sample was retrieved from a public sandbox submission and distributed inside a password-protected archive. Its MD5 digest was computed and matched against the reference value before any analysis began.

## Lab environment

- Hypervisor: VMware Workstation Pro
- Guest OS: Windows XP
- Network: isolated, with no route to production systems

The malware sample is not included in this repository; only the written report is published.

## Analysis

### Static analysis

WinMD5 produced the MD5 digest used to verify file integrity against the published reference. PEiD reported an entry point inside `.text` and matched no known packer signature, indicating the file is not packed or obfuscated. PEview confirmed a conventional section layout (`.text`, `.rdata`, `.data`) with one additional custom section, `.x`, a TimeDateStamp of 23 June 2016, and no malformed headers. BinText recovered strings naming browser credential stores — Chrome's Login Data and Firefox's `logins.json` and `signons.sqlite` — alongside SQLite calls (`sqlite3_open16`, `sqlite3_prepare_v2`) and Credential Manager APIs (`VaultEnumerateVaults`, `VaultOpenVault`, `VaultGetItem`), plus Outlook and FileZilla artefacts. Dependency Walker showed imports from ADVAPI32.DLL, KERNEL32.DLL (`CreateMutexW`, `CreateProcessInternalA`, `CreateFileMapping`) and NTDSAPI.DLL (`DsMakePasswordCredentialsW`, `DsCrackNamesW`). VirusTotal returned 66 detections from 71 engines, labelled chiefly as Trojan.LokiBot, Trojan.PWS.ZKD and Win32.LokiBot-A. IDA Pro was used on the start function, whose flow graph shows an initialisation sequence, a loop containing a 10,000 ms delay (`2710h`), and a cleanup block; its import listing adds the Winsock set (`WSAStartup`, `socket`, `connect`, `send`, `recv`, `getaddrinfo`).

### Dynamic analysis

Process Monitor recorded repeated `RegQueryValue` and `RegEnumValue` calls consistent with system profiling, together with loads of `secur32.dll`, `advapi32.dll`, `ws2_32.dll`, `rsaenh.dll`, `crypt32.dll`, `wininet.dll`, `samlib.dll`, `ntdll.dll` and `comctl32.dll`. Process Explorer placed the process beneath `explorer.exe`, confirming interactive execution, and showed active TCP connections alongside the loaded modules. RegShot compared pre- and post-execution registry snapshots and recorded values added, modified and deleted, including the cryptographic RNG seed and UserAssist entries, the latter stored in ROT13. CaptureBAT logged file system access and writes to the `NTUSER.DAT` and `software.LOG` hive files. ApateDNS returned no records, and Wireshark captured TCP SYN traffic toward 94.156.177.41 on port 80 with no corresponding name resolution. OllyDbg showed execution stopped at `kernel32.LoadLibraryW`, with short `push`/`mov`/`call`/`pop`/`retn` wrapper functions and RWE memory regions holding the decrypted code.

## Findings

- Credential targets span Chrome and Firefox stores, Outlook SMTP and POP3 settings, FileZilla (`Filezilla.xml`, `recentservers.xml`), and Windows Credential Manager vaults.
- APIs are resolved at runtime through `LoadLibraryW` and `GetProcAddress` rather than the import address table, so the IAT gives an incomplete picture of capability.
- Registry values are added, modified and deleted during execution, and hive files are written to; the report attributes persistence intent to this activity but does not identify a specific autorun value.
- Collected data is encrypted before transmission, inferred from `rsaenh.dll` and `crypt32.dll` usage and from the altered RNG seed rather than observed on the wire.
- Command and control is a hardcoded address, 94.156.177.41, contacted over HTTP on port 80; the absence of DNS traffic indicates direct connection without domain resolution.
- Anti-analysis behaviour includes a ten-second delay loop, mutex creation to prevent duplicate execution, and the `long-sleeps` and `detect-debug-environment` classifications returned by VirusTotal.

## Detection and response

- Monitor registry writes and DLL loads for stealer-typical combinations of credential, cryptographic and network libraries.
- Block outbound traffic to known malicious addresses and alert on direct-to-IP HTTP connections that bypass name resolution.
- Restrict execution of untrusted binaries and apply endpoint detection to flag unusual outbound sessions.
- Hunt for access to browser credential databases, `Filezilla.xml`, `recentservers.xml` and Credential Manager vault APIs.

Remediation as documented in the report:

1. Isolate the infected machine by disconnecting it from the network.
2. Boot into Safe Mode with Networking to limit active processes.
3. Identify and terminate the malicious processes using Task Manager or Process Explorer.
4. Delete the malicious executable and check browser, FileZilla and Outlook credential stores for theft.
5. Clean the registry, inspecting `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` and `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` for persistence entries.
6. Run a full system scan with Malwarebytes and Microsoft Defender Offline.
7. Change credentials for email, FTP, domain and online service accounts.
8. Enable logging and intrusion detection, and monitor outbound connections for reinfection.

## Key takeaways

- Packer detection alone does not establish that a sample is unprotected; the file is unpacked on disk but decrypts code into RWE memory at runtime.
- A near-empty import table is a stronger signal than a populated one, because dynamic resolution hides the true API surface.
- Absence of DNS traffic paired with direct IP connections is a reliable indicator of hardcoded command and control.
- Registry diffing surfaces persistence and configuration changes that process monitoring alone can miss.
- Credential exposure after an infostealer infection extends beyond the host to every account whose secrets were stored on it.

## Notes

- Academic exercise conducted in an isolated laboratory; the malware sample is not distributed in this repository.
- Indicators quoted here are reproduced from the report as recorded during analysis.
