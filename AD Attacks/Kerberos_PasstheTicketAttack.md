# **Pass-the-Ticket (PTT) Attack - Kerberos**

## **Understanding Pass-the-Ticket (PTT)**

- **Pass-the-Ticket (PTT)** is an attack in which an attacker uses a **valid Kerberos ticket** (TGT or TGS) from a compromised user or service account to gain unauthorized access to resources. Unlike traditional credential-based attacks, PTT does not require the attacker to use a password or hash but instead leverages an existing, valid Kerberos ticket for authentication.
- In a **Pass-the-Ticket** attack, the attacker does not need to compromise the Kerberos KDC (Key Distribution Center) but can use a stolen ticket obtained through techniques like credential dumping or capturing network traffic.

---

## **How Pass-the-Ticket Works**

1. **Compromise User or Service Account**:

   - The attacker gains access to a user or service account and steals the **Kerberos ticket** (TGT or TGS). This ticket is often found in memory or can be dumped from the system using tools like **Mimikatz**.

2. **Forge or Steal the Ticket**:

   - If the attacker has sufficient privileges, they may dump the **Kerberos ticket** directly from the compromised system's memory or from the **LSASS process**.

3. **Use the Stolen Ticket for Authentication**:

   - The attacker then **forwards** or **passes** the stolen ticket to other systems or services that authenticate using Kerberos. This allows the attacker to authenticate as the original user without needing to know the user's password.

4. **Access Resources**:
   - Once the attacker passes the valid Kerberos ticket to a service (e.g., SMB, RDP), the service will validate the ticket against the KDC. Since the ticket is valid, the attacker is granted access to the targeted service or resource.

---

## **Tools and Techniques**

### **Mimikatz**:

- **Mimikatz** is commonly used to dump Kerberos tickets from memory or steal tickets from the **LSASS** process.

  - **Dumping Kerberos Tickets**:

    ```powershell
    sekurlsa::tickets
    ```

    - This command extracts the tickets from the memory. The attacker can then extract valid tickets (TGT or TGS) and reuse them for PTT attacks.

  - **Extracting Tickets from LSASS**:
    ```powershell
    sekurlsa::logonPasswords /export
    ```
    - This command will dump all cached passwords and tickets from memory, including Kerberos tickets.

---

### **Impacket (Pass-the-Ticket Tool)**:

- **Impacket** provides tools to leverage the stolen Kerberos tickets and authenticate to target services using those tickets.

  - **Passing the Ticket to Target Service**:
    ```bash
    python3 ptt.py -ticket <path_to_ticket> <target_service>
    ```
    - This command will pass the stolen ticket (`.kirbi` file) to a target service (e.g., SMB, RDP, etc.) to authenticate.

---

## **Pass-the-Ticket Attack Workflow**

### **Step 1: Dump Kerberos Tickets (Mimikatz)**

- Once the attacker gains access to the victim machine, they can dump the tickets from memory:
  ```powershell
  sekurlsa::tickets
  ```
- The output shows all Kerberos tickets associated with the logged-in sessions, including TGTs (for domain access) and TGSs (for service-specific access).

### **Step 2: Export Tickets (Mimikatz)**

- If the attacker wants to export tickets for later use:
  ```powershell
  sekurlsa::tickets /export
  ```
- This will export the tickets into `.kirbi` files, which can be reused in a **Pass-the-Ticket** attack.

### **Step 3: Pass the Ticket to Target Service (Impacket)**

- The attacker now uses **Impacket's `ptt.py`** tool to pass the stolen ticket to a target service:
  ```bash
  python3 ptt.py -ticket <path_to_ticket> -target <target_ip_or_hostname>
  ```
  - This command uses the stolen Kerberos ticket to authenticate to a target service without requiring a password.

### **Step 4: Gain Access to Target Resource**

- The attacker can now access the service or resource the ticket was valid for, such as **SMB**, **RDP**, or any other Kerberos-enabled service.

---

## **Real-World Example**

1. **Stealing Tickets (Mimikatz)**:

   - An attacker gains access to a machine (e.g., through phishing, privilege escalation).
   - The attacker runs Mimikatz to dump the Kerberos tickets from memory:
     ```powershell
     sekurlsa::tickets
     ```

2. **Exporting Stolen Ticket**:

   - The attacker exports the ticket to a `.kirbi` file for later use:
     ```powershell
     sekurlsa::tickets /export
     ```

3. **Passing the Ticket to Target Service (Impacket)**:

   - The attacker passes the stolen ticket to authenticate to a **SMB** share on a remote machine:
     ```bash
     python3 ptt.py -ticket stolen_ticket.kirbi -target 192.168.1.100
     ```

4. **Accessing the Target Resource**:
   - The attacker gains access to the SMB share, bypassing the need for a password.

---

## **Detection and Mitigation**

### **Detection Techniques**:

1. **Event Logs**:

   - Monitor **Event ID 4769** (TGS-REQ) and **Event ID 4768** (TGT-REQ) for unusual Kerberos requests.
     - Look for repetitive or abnormal Kerberos ticket requests that may indicate an attacker is using a stolen ticket.

2. **Network Traffic Analysis**:

   - Monitor for unexpected Kerberos traffic (such as TGS requests) in **unusual contexts** (e.g., external IPs or services that shouldn't be accessed).

3. **Kerberos Authentication Failure**:

   - Investigate failed Kerberos authentication attempts (Event ID 4771). Multiple failed attempts to use Kerberos tickets from a compromised system can be a sign of PTT activity.

4. **Ticket Stale/Expired**:
   - Monitor for tickets that have been used after expiration, which could indicate that an attacker is using a replayed or stolen ticket.

---

### **Mitigation Techniques**:

1. **Credential Protection**:

   - Protect Kerberos tickets stored in memory and ensure they are not exposed via memory dumps.
   - Use **Windows Defender Credential Guard** to protect Kerberos tickets from being dumped by attackers.

2. **Enable **Ticket-Expiry Validation\*\*:

   - Enforce short Kerberos ticket lifetimes, so even if a ticket is stolen, it will expire quickly.

3. **Monitor Service Account Usage**:

   - Regularly monitor and audit the use of service accounts, ensuring that **unused** accounts or accounts with high privileges are not used for authentication.

4. **Use Multi-Factor Authentication**:

   - Employ **multi-factor authentication (MFA)** where possible to protect sensitive accounts from being compromised.

5. **Implement Lateral Movement Detection**:
   - Monitor for lateral movement using tools like **Sysmon** and **SIEM systems** to detect abnormal Kerberos authentication patterns.

---

## **Example Scenario (Full)**

1. **Dump Kerberos Tickets**:

   - An attacker dumps the Kerberos tickets using Mimikatz:
     ```powershell
     sekurlsa::tickets
     ```

2. **Export the Ticket**:

   - The attacker exports the ticket to a file:
     ```powershell
     sekurlsa::tickets /export
     ```

3. **Pass the Ticket to Target**:

   - The attacker passes the ticket to a target service (SMB, RDP):
     ```bash
     python3 ptt.py -ticket stolen_ticket.kirbi -target 192.168.1.100
     ```

4. **Access Resource**:
   - The attacker successfully accesses the target resource without the need for credentials.

---

## **⚠️Disclaimer⚠️**:

This guide is for **educational purposes only**. Unauthorized exploitation of Kerberos tickets is illegal and unethical. Always conduct penetration testing with **proper authorization** and adhere to ethical guidelines.
