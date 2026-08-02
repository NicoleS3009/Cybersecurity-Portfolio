# Hash Cracking Analysis

⚠️ Academic laboratory conducted within a controlled environment.  
Hashes and passwords have been anonymized.

## Context
Cryptography course laboratory focused on identifying and evaluating 
the security of various hashing algorithms.

## Objective
Understand the resistance of different hashing algorithms against 
cracking and dictionary attacks.

## Analyzed Algorithms
- MD4
- MD5
- SHA1
- SHA256
- NTLM
- bcrypt
- sha512crypt

## Methodology
- Hash type identification
- Evaluation of resistance
- Use of cracking tools (online and local)
- Comparison between fast hashes and hashes designed for passwords

## Results
- Fast algorithms (MD4, MD5, SHA1) proved to be highly vulnerable
- NTLM exhibited weaknesses against dictionary attacks
- bcrypt and sha512crypt showed greater resistance due to the use of salting 
  and key derivation functions

## Mitigation
- Use of resistant algorithms such as bcrypt, Argon2, or scrypt
- Implementation of strong password policies
- Use of salting and attempt rate limiting

## Key Takeaways
Practical understanding of why weak hashing algorithms 
must not be used for storing passwords.