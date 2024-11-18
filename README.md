# Password Strength Analyzer

https://github.com/user-attachments/assets/a0a416c4-3ea8-40a4-b28c-76df107ba9cd

[live demo deployed on github pages](https://eaeo.top/password-evaluator/) (https://eaeo.top/password-evaluator/)
---

## Password Strength Analyzer

This project evaluates the strength of a password based on various metrics and provides feedback to help users create stronger passwords. The system considers password length, character variety, entropy, patterns, and exposure in data breaches to calculate an overall strength score.

---

### Features

1. **Interactive Input**: 
   - Toggle password visibility.
   - Immediate feedback on strength and improvement suggestions.

2. **Scoring System**:
   - Calculates a score between 0 and 100 based on password properties.
   - Factors contributing to the score:
     - Length (up to 20 points).
     - Character diversity (up to 20 points for lowercase, uppercase, numbers, and special characters).
     - Entropy (up to 20 points).
     - Penalties for common patterns and breaches.

3. **Advanced Feedback**:
   - Highlights patterns detected in the password.
   - Shows entropy in bits and potential exposure in breaches.
   - Provides actionable suggestions to improve strength.

4. **Security**:
   - Compares passwords against a list of 10,000 common passwords.
   - Checks for exposure in real-world data breaches using the [Pwned Passwords API](https://haveibeenpwned.com/Passwords).

---

### Scoring Breakdown

| Metric                 | Max Points | Description                                                                                   |
|------------------------|------------|-----------------------------------------------------------------------------------------------|
| **Length**             | 20         | Longer passwords score higher, with diminishing returns after 12 characters.                  |
| **Character Diversity**| 20         | Rewards use of lowercase, uppercase, numbers, and special characters.                         |
| **Entropy**            | 20         | Measures unpredictability based on unique characters and length.                              |
| **Common Password**    | -50        | Deducts points if the password matches a common password.                                     |
| **Data Breaches**      | -50        | Deducts points proportionally if the password has been found in known data breaches.          |

**Final Score**: Sum of all metrics, normalized between 0 (weak) and 100 (strong).

---

### Key Functions and Concepts

#### 1. **Password Entropy**
   - Calculates the randomness of the password using character set size and length.
   - Formula: `log2(character_set_size ^ password_length)`
   - Higher entropy indicates stronger passwords.

#### 2. **Pattern Analysis**
   Detects common weaknesses:
   - Sequential characters (e.g., `1234`, `abcd`).
   - Repeated characters (e.g., `aaaaaa`).
   - Keyboard patterns (e.g., `qwerty`).
   - Repeated substrings (e.g., `abcabc`).
   - Palindromes and alternating characters.

#### 3. **Common Password Detection**
   - Checks if the password exists in a predefined list of common passwords.

#### 4. **Data Breach Check**
   - Uses the Pwned Passwords API to verify if the password has been exposed.
   - SHA1 hash of the password is divided into prefix and suffix to ensure privacy.

---

### User Feedback
1. **Real-Time Suggestions**:
   - Recommendations for length, character variety, and complexity.
2. **Patterns Detected**:
   - Highlights problematic patterns (e.g., `Repeated characters`).

---

Start with `vite preview`

Available on port 4173

At the URL:

[http://localhost:4173/password-evaluator/](http://localhost:4173/password-evaluator/)
