# **NTLM Relay Attack**

## **Understanding NTLM Relay**

- **NTLM** (New Technology LAN Manager) is a legacy authentication protocol used in Windows environments.
- **NTLM Relay** is an attack where an attacker intercepts NTLM authentication traffic and relays it to a malicious server. This allows the attacker to gain access to resources that the legitimate user has access to.

## **Tools and Techniques**

### **Impacket:**

A powerful Python library for working with Windows networks. It provides tools for various attacks, including NTLM Relay.

**Basic NTLM Relay with Impacket (Conceptual Example)**

1. **Prerequisites:**

   - **Victim Machine:** A Windows machine joined to the domain.
   - **Attack Machine:** Kali Linux with Impacket installed.
   - **Target Server:** A server that requires NTLM authentication (e.g., SMB, HTTP).

2. **Install Impacket:**

   ```bash
   pip install impacket
   ```

3. **Capture NTLM Hash (Conceptual):**

   - **Method 1 (SMB Relay):**

     - Use `smbclient` or other tools to connect to a file share on the victim machine.
     - Capture the NTLM hash during the authentication process.

   - **Method 2 (HTTP Relay):**
     - Use tools like Wireshark or Burp Suite to capture NTLM traffic over HTTP.

4. **Relay NTLM Hash to Target Server (Conceptual):**

   - Use the `ntlmrelayx` tool from Impacket:

     ```bash
     ntlmrelayx -t <target_server_ip> -u <username> -H <captured_ntlm_hash>
     ```

     - Replace `<target_server_ip>`, `<username>`, and `<captured_ntlm_hash>` with the actual values.

### **Responder**

Responder is a versatile tool that can perform various network attacks, including NTLM relay, LLMNR/NBT-NS poisoning, and more.

- **Installation:**
  ```bash
  git clone https://github.com/lgandx/Responder.git
  cd Responder
  sudo python3 setup.py install
  ```
- **Basic NTLM Relay with Responder (Conceptual Example):**

  ```bash
  sudo responder -I eth0 -wAdv
  ```

  - `-I eth0`: Specifies the network interface to listen on.
  - `-A`: Analyze mode. This option allows you to see NBT-NS,BROWSER, LLMNR requests without responding.
  - `-w`: Start the WPAD rogue proxy server. Default value is False
  - `-v`: Enables verbose output.

**Key Points:**

- **LLMNR/NBT-NS Poisoning:** Responder can poison LLMNR and NBT-NS name resolution requests, tricking victims into authenticating to the attacker's machine.
- **Flexibility:** Responder offers various options for configuring the attack, such as specifying target services, controlling the level of verbosity, and more.

**Important Notes:**

- **This is a simplified example.** NTLM Relay attacks often involve more complex scenarios, such as double hop attacks.
- **Ethical Considerations:** Always perform penetration testing within legal and ethical boundaries. Obtain proper authorization before testing any systems.
- **Mitigation:**
  - **Disable NTLM:** If possible, disable NTLM and use stronger authentication protocols like Kerberos.
  - **Implement MFA:** Use multi-factor authentication to enhance security.
  - **Network Segmentation:** Isolate critical systems on separate networks.
  - **Regular Security Audits:** Conduct regular security audits to identify and address vulnerabilities.

**⚠️Disclaimer⚠️**
This information is for educational purposes only. I do not condone any illegal or unethical activities. Always perform penetration testing within legal and ethical boundaries. Obtain proper authorization before testing any systems.

Remember to adapt these commands and techniques to your specific environment and target.

**Remember:**

- NTLM Relay attacks can be complex.
- Thoroughly research and understand the techniques and potential risks before attempting any attacks.
- Always prioritize security best practices like disabling NTLM and implementing strong authentication methods.
