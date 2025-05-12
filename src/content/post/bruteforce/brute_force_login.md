---
title: PortSwigger Lab - Brute Force Login Page
description: Step-by-step walkthrough of bypassing authentication through brute force attacks
publishDate: "2024-11-12T16:00:00Z"
---

![Brute Force Login Interface](/portfolio/brute_force_login.png)
*Figure 1: Burp Suite login page showing POST request parameters*

## Introduction

Authentication systems are a critical security component for web applications. However, when poorly implemented, they can be vulnerable to brute force attacks. In this lab walkthrough, I'll demonstrate how to use Burp Suite's Intruder feature to perform a brute force attack against a login page.

---

## What You'll Learn

✅ **Intercept authentication requests** using Burp Suite  
✅ **Configure Burp Intruder** for multiple parameter attacks  
✅ **Use payload positions** to target both username and password fields  
✅ **Apply Grep matching** to identify successful login attempts  
✅ **Execute cluster bomb attacks** to test multiple credential combinations

By the end of this walkthrough, you'll understand how to identify and exploit vulnerable login pages using automated tools.

---

## Step 1: Capture the Login Request

First, we'll use Burp Suite to intercept the login request:

1. Configure your browser to use Burp Suite as a proxy
2. Navigate to the target login page
3. Enable intercept in the Proxy tab
4. Submit login credentials
5. Examine the captured POST request


This request contains the parameters we'll need to manipulate for our brute force attack.

---

## Step 2: Send to Intruder

After capturing the login request:

1. Right-click on the request
2. Select "Send to Intruder"
3. Go to the Intruder tab

![Intruder Target Configuration](/portfolio/image4.png)
*Figure 3: Send request to the intruder*

---

## Step 3: Configure Attack Type

For this attack, we need to test multiple username and password combinations:

1. Select "Cluster bomb" as the attack type
   - This allows us to use multiple payload sets and test all combinations
2. Clear any default positions by clicking "Clear §"

![Cluster Bomb Attack Selection](/portfolio/image5.png)
*Figure 4: Setting up the Cluster Bomb attack type*

---

## Step 4: Set Payload Positions

Now we need to mark the username and password fields for injection:

1. Highlight the username value
2. Click "Add §" to add it as the first position
3. Highlight the password value
4. Click "Add §" to add it as the second position

![Payload Position Configuration](/portfolio/image6.png)
*Figure 5: Setting payload positions for username and password fields*

---

## Step 5: Configure Payloads

Next, we'll set up our wordlists for both parameters:

1. Go to the Payloads tab
2. For Payload set 1 (usernames):
   - Select "Simple list" as the payload type
   - Add potential usernames to the list
3. For Payload set 2 (passwords):
   - Select "Simple list" as the payload type
   - Add potential passwords to the list

![Payload Configuration](/portfolio/image7.png)
*Figure 6: Configuring payload lists for both parameters*

---

## Step 6: Add Grep Match

To identify successful login attempts:

1. Go to the Options tab
2. Under "Grep - Match", click "Add"
3. Enter "Incorrect Password" as the string to match
   - This will help us identify which responses do NOT contain this error message

![Grep Match Configuration](/portfolio/image8.png)
*Figure 7: Setting up Grep Match to identify successful login attempts*

---

## Step 7: Launch the Attack

Now we're ready to execute our brute force attack:

1. Click "Start attack" to begin
2. The attack window will show all combinations being tested
3. Examine the results, particularly the Grep column

![Attack Results](/portfolio/image9.png)
*Figure 8: Results showing request #70 as a successful login attempt*

Notice in the results that response #70 doesn't have the "Incorrect Password" message flagged (no "1" in that column), indicating a successful login.

---

## Conclusion

In this lab, we successfully demonstrated a brute force attack against a vulnerable login page. By using Burp Suite's Intruder with a Cluster Bomb attack, we were able to test multiple username/password combinations and identify valid credentials.

This highlights the importance of implementing proper brute force protection mechanisms like:

- Account lockout policies
- CAPTCHA or similar challenges
- Rate limiting
- Increasing response times after failed attempts
- Two-factor authentication

Remember that this technique should only be used for security testing on systems you have permission to test, never for unauthorized access.
