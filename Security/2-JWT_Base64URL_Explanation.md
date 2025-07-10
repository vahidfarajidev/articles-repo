# Understanding JWT and Base64URL

## What Is a JWT?

A JWT (JSON Web Token) consists of three parts:

```
HEADER.PAYLOAD.SIGNATURE
```

Each part is Base64URL-encoded — not encrypted — and separated by dots. This structure makes it easy to transmit in HTTP headers and URLs.

---

## Is Base64URL Secure?

**No, Base64URL is not secure on its own.** It is just an encoding method to make data URL-safe and compact.

Let’s break down this example JWT:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 🔹 Header (Base64URL-decoded):

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 🔹 Payload (Base64URL-decoded):

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

Both of these parts can be decoded by anyone — even without the secret key.

---

## Where’s the Security Then?

Security comes from the **signature** part:

```
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

This is created using:

- Header + Payload
- A secret key
- A hashing algorithm (e.g., HMAC SHA-256)

When the server receives a JWT, it re-generates the signature using the same secret key and compares it with the token's signature. If they don’t match, the token is **rejected**.

---

## Key Points

- ✅ JWT content is **readable** via Base64URL.
- ❌ JWT is **not encrypted** by default.
- 🔐 JWT security relies on the **signature** and **secret key**.
- ⚠️ Do **not** put sensitive data (e.g., passwords) in the payload.
- ✅ Always use HTTPS to prevent token theft.
- 🔐 For confidentiality, use **JWE (JSON Web Encryption)** in addition to JWT.

---


---

## How Is the Signature Created?

The signature in a JWT is what guarantees that the token hasn't been tampered with. It's generated using the following components:

1. **Header + Payload**: These are encoded using Base64URL and concatenated with a dot:
   ```
   base64UrlEncode(header) + "." + base64UrlEncode(payload)
   ```

2. **A Secret Key**: A private key known only to the server. This is used to sign the token.

3. **A Hashing Algorithm**: Such as `HMAC SHA-256`. It generates a unique signature (like a fingerprint) based on the header, payload, and secret.

If any part of the token is modified (even a single character), the signature validation will fail when checked by the server, and the token will be rejected.

### Analogy

Think of the signature like a wax seal on a letter. Everyone can read the letter (it's not encrypted), but only the person with the original seal (secret key) can produce a valid one. If someone tampers with the contents and reseals it without the correct seal, it's obvious it's been tampered with.



---

## How Does HMAC SHA-256 Work in JWT?

When creating the signature part of a JWT, we use an algorithm like `HMAC SHA-256`. This process ensures the token hasn’t been altered. Let’s break it down step-by-step.

### 🔢 Step-by-Step Signature Creation

#### 1. Prepare the Header and Payload

Convert both to JSON and then Base64URL-encode them:

```json
Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

After encoding:

```text
encodedHeader  = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
encodedPayload = eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
```

#### 2. Create the String to Sign

Join the two encoded strings with a dot (`.`):

```text
dataToSign = encodedHeader + "." + encodedPayload
```

Example:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
```

#### 3. Apply HMAC SHA-256

Now we generate a signature using:

```text
signature = HMAC_SHA256(dataToSign, secretKey)
```

- **dataToSign** is the string above
- **secretKey** is a private key that only the server knows (e.g., `"my_super_secret_key"`)

This results in a binary signature, which is then Base64URL-encoded to create the final signature string.

```text
encodedSignature = Base64UrlEncode(signature)
```

Example (simplified):

```text
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### 4. Final JWT Token

Now you can build the complete JWT:

```text
encodedHeader + "." + encodedPayload + "." + encodedSignature
```

---

## 🔐 What Is the Role of `secretKey`?

The `secretKey` is critical. It ensures that only the server (who knows the key) can create or validate a valid signature.

If someone changes the payload, they can’t generate a matching signature without knowing the secret key.

### Analogy

Think of the `secretKey` like a stamp only the server has. If someone tampers with the document, they can’t reproduce the original stamp — and the forgery is detected immediately.

---

## 🧪 Summary Table

| Component     | Purpose                                          | Example                                                  |
|---------------|--------------------------------------------------|----------------------------------------------------------|
| Header        | Metadata about algorithm and token type         | `{ "alg": "HS256", "typ": "JWT" }`                      |
| Payload       | User data or claims                             | `{ "sub": "1234567890", "name": "John Doe" }`           |
| Secret Key    | Used in signature generation                     | `"my_super_secret_key"`                                 |
| Signature     | Ensures integrity and authenticity               | `HMAC_SHA256(header.payload, secretKey)` → base64url    |

---


## Conclusion

JWTs are not meant to hide information — they are meant to **prove authenticity**. While the contents are readable, the signature ensures they haven’t been tampered with.

Always handle JWTs securely and understand what they are — and are not — designed to do.

---

**Resources:**

- [https://jwt.io](https://jwt.io)
- [RFC 7519 - JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
