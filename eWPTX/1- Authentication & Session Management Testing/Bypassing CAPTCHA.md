---
date: 2025-12-29 19:12
status:
tags:
tags2: "[[Authentication]]"
Tag3: "[[eWPTX]]"
---

# Bypassing CAPTCHA

## Overview
This test focuses on identifying **weak lockout mechanisms**, specifically **CAPTCHA implementations** that aim to prevent automated login attempts but are poorly designed or improperly enforced.

---

## What is CAPTCHA?
**CAPTCHA** (Completely Automated Public Turing test to tell Computers and Humans Apart) is a security control that requires a user to solve a challenge to prove they are human, thereby preventing automated bot attacks.

---

## Prerequisites
- Understanding of authentication workflows
- Familiarity with brute-force attacks
- Basic scripting or automation knowledge
- Awareness of OCR and ML-based attacks

---

## Types of CAPTCHA

---

## 1. Arithmetic-Based CAPTCHA (Weak)

**Description:**
- Asks the user to solve a simple arithmetic problem (e.g., *“What is 3 + 7?”*).

**Strength:**
- Very low

**Bypass Techniques:**
- Brute-force automation
- CAPTCHA-solving services (e.g., 2Captcha)

**Why It Fails:**
- Extremely low complexity
- Easily solvable by scripts or bots
- No real human verification


---

## 2. Text-Based CAPTCHA (Basic)

**Description:**
- Displays distorted or jumbled text that the user must type correctly.

**Strength:**
- Basic

**Observations:**
- Slightly more complex than arithmetic CAPTCHAs
- Often includes distorted letters or background noise

**Vulnerabilities:**
- Bots using OCR or pre-trained ML models can recognize text
- Simple distortions are trivial for modern automated tools
- Frequently bypassed at scale


---

## 3. Image-Based CAPTCHA (Moderate)

**Description:**
- Users must identify or select images matching a condition  
  (e.g., *“Select all images with traffic lights”*).

**Strength:**
- Moderate

**Why It’s Better:**
- Requires more advanced AI models
- Harder to automate than text-based CAPTCHAs

**Vulnerabilities:**
- Still susceptible to machine learning
- Object-recognition models can bypass with sufficient training
- Not fully bot-proof


---

## 4. reCAPTCHA v2 (Moderate → Strong)

**Description:**
- Developed by Google
- Typically involves clicking **“I’m not a robot”**
- May trigger image-based challenges depending on risk

**Security Notes:**
- Combines user interaction with behavioral analysis
- Much harder to bypass than custom CAPTCHAs

---

## 5. reCAPTCHA v3 (Very Strong)

**Description:**
- Fully invisible CAPTCHA
- Runs in the background without user interaction
- Assigns a **risk score** based on user behavior

**Why It’s Strong:**
- No challenge to solve
- Difficult to automate convincingly
- Relies on behavioral signals rather than puzzles

---

## Key Takeaway
> CAPTCHA alone is **not sufficient** as a lockout mechanism.  
> Weak implementations can be bypassed easily, especially without:
- Rate limiting
- Account lockout policies
- Behavioral analysis
- Proper server-side validation

---
# Lab Documentation
file:///Users/abdalrahmanmajed/Downloads/walkthrough-2172.pdf
