# System Design Concepts

## Hashing vs Encryption

### Hashing

**Definition**: Hashing is a one-way cryptographic function that converts input data of any size into a fixed-size string of characters (hash value or digest).

**Key Characteristics:**
- **One-way function**: Cannot be reversed to get original data
- **Deterministic**: Same input always produces same hash
- **Fixed output size**: Regardless of input size
- **Avalanche effect**: Small input change causes dramatically different hash
- **Fast computation**: Quick to calculate

**Common Hash Functions:**
- MD5 (128-bit) - Deprecated due to vulnerabilities
- SHA-1 (160-bit) - Deprecated for cryptographic use
- SHA-256 (256-bit) - Widely used
- SHA-3 (variable) - Latest standard
- bcrypt - Designed for password hashing
- Argon2 - Modern password hashing winner

**Use Cases:**
- Password storage (with salt)
- Data integrity verification
- Digital signatures
- Blockchain/cryptocurrency
- Hash tables/maps
- Checksums
- Deduplication

**Example:**
```
Input: "hello"
SHA-256: 2cf24dba4f21d4288094e5878138e9b21d5e1d7e2c10d9b86c0db49e7e6e59d3

Input: "hello!"
SHA-256: ce06092fb948d9ffac7d1a376e404b26b7575bcc11ee05a4615fef4fec3a308b
```

### Encryption

**Definition**: Encryption is a two-way process that transforms plaintext into ciphertext using an algorithm and key, with the ability to decrypt back to original data.

**Key Characteristics:**
- **Reversible**: Can decrypt to get original data
- **Requires key(s)**: Uses cryptographic keys
- **Variable output size**: Often similar to input size
- **Confidentiality**: Protects data from unauthorized access

**Types of Encryption:**

#### Symmetric Encryption
- **Same key** for encryption and decryption
- **Faster** than asymmetric
- **Key distribution problem**: How to securely share the key

**Algorithms:**
- AES (Advanced Encryption Standard) - Most common
- DES (Data Encryption Standard) - Deprecated
- 3DES (Triple DES) - Legacy
- ChaCha20 - Stream cipher

#### Asymmetric Encryption
- **Key pair**: Public and private keys
- **Public key** encrypts, **private key** decrypts
- **Slower** than symmetric
- **Solves key distribution** problem

**Algorithms:**
- RSA - Most common
- ECC (Elliptic Curve Cryptography)
- DSA (Digital Signature Algorithm)

**Use Cases:**
- Data transmission security (HTTPS/TLS)
- File/disk encryption
- Email encryption (PGP/GPG)
- Secure messaging
- VPN connections
- Database encryption

### Key Differences

| Aspect | Hashing | Encryption |
|--------|---------|------------|
| **Purpose** | Data integrity, verification | Data confidentiality |
| **Reversibility** | One-way (irreversible) | Two-way (reversible) |
| **Keys** | No keys required | Requires cryptographic keys |
| **Output Size** | Fixed size | Variable (usually similar to input) |
| **Speed** | Very fast | Slower (especially asymmetric) |
| **Use Case** | Passwords, checksums, integrity | Secure communication, storage |

### Security Considerations

**Hashing:**
- Use cryptographically secure hash functions
- Always salt passwords before hashing
- Use slow hash functions for passwords (bcrypt, Argon2)
- Avoid MD5 and SHA-1 for security purposes

**Encryption:**
- Use strong, well-tested algorithms (AES, RSA)
- Proper key management is crucial
- Use appropriate key sizes (AES-256, RSA-2048+)
- Implement proper initialization vectors (IV)
- Consider authenticated encryption (AES-GCM)

### Common Misconceptions

1. **"Hashing is encryption"** - No, hashing is one-way
2. **"Encrypted passwords"** - Passwords should be hashed, not encrypted
3. **"MD5 is secure"** - MD5 is cryptographically broken
4. **"Longer hash = more secure"** - Not always true; algorithm matters more

### Real-World Examples

**Password Storage:**
```
// Wrong: Storing plain text
user_password = "mypassword123"

// Wrong: Using encryption
encrypted_password = encrypt("mypassword123", key)

// Correct: Using hashing with salt
salt = generate_random_salt()
hashed_password = bcrypt("mypassword123" + salt)
```

**Data Transmission:**
```
// HTTPS uses both:
1. Asymmetric encryption (RSA/ECDH) for key exchange
2. Symmetric encryption (AES) for data transmission
3. Hashing (SHA) for message authentication
```

### When to Use Which

**Use Hashing When:**
- Storing passwords
- Verifying data integrity
- Creating checksums
- Building hash tables
- Digital signatures

**Use Encryption When:**
- Protecting data in transit
- Securing stored sensitive data
- Implementing secure communication
- Building VPNs or secure channels
- Need to retrieve original data

Understanding these fundamental differences is crucial for system design interviews and building secure systems.