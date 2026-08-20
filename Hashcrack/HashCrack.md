# HashCrack

## 1. Title & Metadata
- **Challenge:** HashCrack
- **Category:** Cryptography
- **Difficulty:** Easy
- **Platform:** picoCTF

## 2. TL;DR / Key Takeaways
The challenge served three hashes of increasing algorithm strength (MD5 → SHA-1 → SHA-256), each protecting an extremely weak, common password. Rather than any cryptographic weakness in the algorithms themselves, the vulnerability exploited was **weak password selection** — all three plaintexts existed in public precomputed lookup tables, so each hash was reversed via a simple online lookup with no brute-forcing required.
<img width="1810" height="945" alt="image" src="https://github.com/user-attachments/assets/69b821d8-70ac-4109-aa10-ba230497de19" />
<img width="1113" height="420" alt="image" src="https://github.com/user-attachments/assets/3bf3adee-7a14-4aa5-86fc-525a40424a0d" />


## 3. Reconnaissance & Enumeration
- **Tool used:** `nc` (netcat) to connect to the challenge service, and [CrackStation](https://crackstation.net/) (free online hash-lookup tool) to reverse each hash.
- Connected to the remote service:
  ```
  nc verbal-sleep.picoctf.net 55006
  ```
- Initial output showed the service would present a hash, prompt for its plaintext password, and — if correct — reveal the next (harder) hash. This indicated a sequential, three-stage challenge rather than a single crack.
- No hash-type identification was needed manually; CrackStation auto-detects the algorithm and reports it alongside the result.

## 4. Exploitation / Solution

**Stage 1 — MD5**
```
482c811da5d5b4bc6d497ffa98491e38
```
Submitted to CrackStation → returned plaintext `password123`. Entered at the prompt, confirmed correct. No secret revealed at this stage.

**Stage 2 — SHA-1**
```
b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3
```
Submitted to CrackStation → returned plaintext `letmein`. Entered at the prompt, confirmed correct. Still no secret.

**Stage 3 — SHA-256**
```
916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745
```
Submitted to CrackStation → identified as SHA-256, returned plaintext `qwerty098` (exact/green match). Entered at the prompt, confirmed correct — this stage revealed the flag.

## 5. Flag
```
picoCTF{UseStr0nG_h@shEs_&PaSswDs!_6965e43b}
```

## 6. Remediation / Lessons Learned
- Hash algorithm strength (MD5 vs. SHA-1 vs. SHA-256) is irrelevant if the underlying plaintext is weak or common — all three passwords here were reversible via a free lookup table with zero compute cost.
- Real-world systems should enforce **salting** (unique per-password random values) and use **slow, purpose-built hashing functions** (bcrypt, scrypt, Argon2) rather than fast general-purpose hashes like MD5/SHA-family, which are trivial to brute-force or look up at scale.
- Password policies should reject common/breached passwords (e.g., via a check against Have I Been Pwned's Pwned Passwords list) to prevent this exact class of attack.
