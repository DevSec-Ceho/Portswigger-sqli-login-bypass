# PortSwigger Web Security Academy: SQL Injection Login Bypass

<p align="center">
  <img src="https://img.shields.io/badge/Platform-PortSwigger-orange?style=for-the-badge" alt="PortSwigger">
  <img src="https://img.shields.io/badge/Category-SQL%20Injection-blue?style=for-the-badge" alt="SQL Injection">
  <img src="https://img.shields.io/badge/Difficulty-Apprentice-brightgreen?style=for-the-badge" alt="Apprentice">
  <img src="https://img.shields.io/badge/Status-Solved-success?style=for-the-badge" alt="Solved">
</p>

<h2>📌 Lab Information</h2>

<table>
  <tr><th>Information</th><th>Details</th></tr>
  <tr><td>Platform</td><td>PortSwigger Web Security Academy</td></tr>
  <tr><td>Lab Name</td><td>SQL injection vulnerability allowing login bypass</td></tr>
  <tr><td>Category</td><td>SQL Injection</td></tr>
  <tr><td>Difficulty</td><td>Apprentice</td></tr>
  <tr><td>Vulnerability</td><td>SQL Injection / Authentication Bypass</td></tr>
  <tr><td>Injection Point</td><td>Username parameter</td></tr>
  <tr><td>Target Account</td><td>Administrator</td></tr>
  <tr><td>Tool Used</td><td>Burp Suite Community Edition</td></tr>
  <tr><td>Status</td><td>Successfully Solved</td></tr>
</table>

<h2>📝 Introduction</h2>

<p>This write-up documents my successful completion of the <strong>SQL injection vulnerability allowing login bypass</strong> lab on PortSwigger Web Security Academy.</p>

<p>The objective was to exploit an SQL injection vulnerability in the application's login functionality to bypass authentication and gain access to the administrator account without knowing the legitimate password.</p>

<p>By manipulating the username parameter and using an SQL comment sequence, I was able to alter the authentication query and neutralize the password verification condition.</p>

<p>This exercise demonstrates how improper handling of user input in authentication queries can compromise an application's access control mechanism.</p>

<h2>🔍 Step 1: Initial Reconnaissance</h2>

<p>After launching the lab instance, I explored the web application and identified its login functionality.</p>

<p>The application presented a login form containing two input fields:</p>

<ul>
  <li><strong>Username:</strong> Used to identify the account.</li>
  <li><strong>Password:</strong> Used to authenticate the account.</li>
</ul>

<p>Since authentication systems commonly interact with databases, I decided to investigate whether the login functionality was vulnerable to SQL injection.</p>

<h3>Screenshot 1: Initial Login Page</h3>

<p align="center">
<img width="1335" height="481" alt="sql2" src="https://github.com/user-attachments/assets/0f028f74-97ac-4ee6-9065-594641088067" />
</p>

<p align="center"><em>Figure 1: The initial login interface presented by the lab.</em></p>

<h2>🔎 Step 2: Intercepting the Login Request</h2>

<p>I entered a username and password into the login form and intercepted the resulting HTTP request using <strong>Burp Suite Community Edition</strong>.</p>

<p>I forwarded the intercepted request to Burp Suite Repeater to modify the submitted parameters and investigate the application's authentication behavior.</p>

<p>The request contained the following important parameters:</p>

<pre><code>POST /login HTTP/2

csrf=...
username=admin
password=...</code></pre>

<p>The username parameter was particularly interesting because it appeared to be used in the application's database authentication process.</p>

<h2>💉 Step 3: Exploiting the SQL Injection Vulnerability</h2>

<p>To test the login functionality, I modified the username parameter with the following SQL injection payload:</p>

<pre><code class="language-sql">administrator'--</code></pre>

<p>The payload was designed to close the original SQL string and introduce a comment sequence that would neutralize the remaining portion of the authentication query.</p>

<p>A vulnerable authentication query might conceptually resemble:</p>

<pre><code class="language-sql">SELECT *
FROM users
WHERE username = 'administrator'
AND password = 'user_password';</code></pre>

<p>After injecting the payload, the resulting query could resemble:</p>

<pre><code class="language-sql">SELECT *
FROM users
WHERE username = 'administrator'--'
AND password = 'user_password';</code></pre>

<p>The <code>--</code> sequence starts an SQL comment in the relevant database syntax.</p>

<p>Consequently, the password verification condition becomes part of the comment and is no longer evaluated as part of the SQL statement.</p>

<p>The application may therefore authenticate the administrator account without checking the supplied password.</p>

<h3>Screenshot 2: Burp Suite Repeater Request and Response</h3>

<p align="center">
 <img width="1546" height="742" alt="sql1" src="https://github.com/user-attachments/assets/50c3cb0a-94de-436d-86a7-095a1cc2983d" />
</p>

<p align="center"><em>Figure 2: The modified login request and the server response inspected through Burp Suite Repeater.</em></p>

<h2>⚠️ Step 4: Understanding the Password Field Requirement</h2>

<p>One interesting observation during this exploitation was that the password field remained required even after the SQL comment was introduced.</p>

<p>Although the injected SQL comment was intended to remove the password verification condition from the backend query, the login form still expected a password value to be submitted.</p>

<p>This happens because client-side form validation and server-side SQL query processing are two different mechanisms.</p>

<ul>
  <li><strong>Client-side validation:</strong> The browser may enforce a required password field before submitting the form.</li>
  <li><strong>Server-side processing:</strong> The application processes the submitted username and password parameters.</li>
  <li><strong>SQL comment:</strong> The injected comment can neutralize the password condition in the vulnerable SQL query.</li>
</ul>

<p>Therefore, I supplied a non-empty password value to satisfy the login form while the injected username payload targeted the backend authentication query.</p>

<p>The important distinction is that satisfying the form's input requirement does not mean the application is actually verifying the supplied password.</p>

<h2>🔓 Step 5: Gaining Access to the Administrator Account</h2>

<p>After submitting the modified login request, the application authenticated me as the administrator.</p>

<p>I was redirected to the account page, where the application displayed the following information:</p>

<pre><code>Your username is: administrator</code></pre>

<p>This confirmed that the authentication bypass was successful.</p>

<h3>Screenshot 3: Successful Administrator Login</h3>

<p align="center">
 <img width="1898" height="656" alt="sql3" src="https://github.com/user-attachments/assets/0ca67105-d87d-4166-956c-ab92278eebcb" />
</p>

<p align="center"><em>Figure 3: Successful access to the administrator account after exploiting the SQL injection vulnerability.</em></p>

<h2>✅ Step 6: Lab Completion</h2>

<p>After successfully accessing the administrator account, PortSwigger recognized that the lab objective had been achieved.</p>

<p>The platform displayed the confirmation message:</p>

<blockquote>
  <p><strong>Congratulations, you solved the lab!</strong></p>
</blockquote>

<p>This confirmed successful completion of the exercise.</p>

<h2>🛡️ Vulnerability Analysis</h2>

<p>The root cause of this vulnerability was the unsafe incorporation of user-controlled input into an SQL authentication query.</p>

<p>Because the application failed to properly separate SQL instructions from user-supplied data, an attacker could manipulate the authentication query and bypass password verification.</p>

<h3>Security Impact</h3>

<ul>
  <li>Unauthorized access to user accounts.</li>
  <li>Authentication bypass without legitimate credentials.</li>
  <li>Potential compromise of administrator privileges.</li>
  <li>Possible unauthorized access to sensitive application functionality and information.</li>
</ul>

<h2>🔐 Recommended Mitigations</h2>

<p>Developers can prevent this vulnerability through the following security practices:</p>

<ol>
  <li><strong>Parameterized Queries:</strong> Use prepared statements to ensure user input is treated as data rather than executable SQL.</li>
  <li><strong>Secure Authentication:</strong> Retrieve user records safely and verify passwords using secure password-hashing functions.</li>
  <li><strong>Input Handling:</strong> Never concatenate untrusted input directly into SQL statements.</li>
  <li><strong>Least Privilege:</strong> Restrict database account permissions to only the operations required by the application.</li>
  <li><strong>Security Testing:</strong> Regularly test authentication functionality for SQL injection vulnerabilities.</li>
</ol>

<h2>💡 Lessons Learned</h2>

<p>This lab reinforced several important web application security concepts:</p>

<ul>
  <li>How SQL injection can compromise authentication mechanisms.</li>
  <li>How SQL comments can neutralize portions of vulnerable queries.</li>
  <li>The difference between client-side form validation and server-side authentication.</li>
  <li>How Burp Suite Repeater can be used to manipulate HTTP requests.</li>
  <li>Why secure database queries are essential for protecting user accounts.</li>
</ul>

<h2>🧰 Tools Used</h2>

<ul>
  <li>Kali Linux</li>
  <li>Mozilla Firefox</li>
  <li>Burp Suite Community Edition</li>
  <li>PortSwigger Web Security Academy</li>
</ul>

<h2>🏁 Conclusion</h2>

<p>The <strong>SQL injection vulnerability allowing login bypass</strong> lab provided practical experience in exploiting SQL injection vulnerabilities within authentication mechanisms.</p>

<p>By injecting <code>administrator'--</code> into the username parameter, I was able to neutralize the password verification portion of the vulnerable authentication query and successfully access the administrator account.</p>

<p>The exercise also highlighted an important distinction between browser-side form validation and backend authentication logic.</p>

<p>It strengthened my understanding of SQL injection, authentication bypass, SQL comment syntax, and HTTP request manipulation.</p>

<p><strong>Lab Status: Successfully Completed! 🚀</strong></p>

<hr>

<p align="center">
  <strong>Hack. Learn. Improve. Repeat.</strong>
  <br>
  <em>Documenting my journey in cybersecurity and ethical hacking.</em>
</p>
