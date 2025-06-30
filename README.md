# rsa-simulation

import random

def is_prime(n, k=5):
    if n < 2: return False
    for p in [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]:
        if n % p == 0: return n == p
    s, d = 0, n - 1
    while d % 2 == 0:
        s, d = s + 1, d // 2
    for _ in range(k):
        a = random.randint(2, n - 1)
        x = pow(a, d, n)
        if x == 1 or x == n - 1: continue
        for _ in range(s - 1):
            x = pow(x, 2, n)
            if x == n - 1: break
        else: return False
    return True

def generate_prime(bits):
    while True:
        p = random.getrandbits(bits)
        p |= (1 << bits - 1) | 1  # Ensure it's a large odd number
        if is_prime(p): return p

def gcd(a, b):
    while b: a, b = b, a % b
    return a

def extended_gcd(a, b):
    if a == 0: return b, 0, 1
    g, y, x = extended_gcd(b % a, a)
    return g, x - (b // a) * y, y

def mod_inverse(a, m):
    g, x, y = extended_gcd(a, m)
    if g != 1: raise Exception('Modular inverse does not exist')
    return x % m

def generate_keypair(bits=512):
    p = generate_prime(bits)
    q = generate_prime(bits)
    while q == p: q = generate_prime(bits)

    n = p * q
    phi = (p - 1) * (q - 1)

    e = random.randint(2, phi - 1)
    while gcd(e, phi) != 1: e = random.randint(2, phi - 1)

    d = mod_inverse(e, phi)

    return (e, n), (d, n)

def encrypt(public_key, plaintext):
    e, n = public_key
    # Convert plaintext string to integer representation
    # This is a simplified approach for demonstration. In real RSA, padding schemes are used.
    m = int.from_bytes(plaintext.encode('utf-8'), 'big')
    if m >= n: raise ValueError("Plaintext too large for this key size")
    ciphertext = pow(m, e, n)
    return ciphertext

def decrypt(private_key, ciphertext):
    d, n = private_key
    m = pow(ciphertext, d, n)
    # Convert integer back to string
    # This assumes the original plaintext can be recovered directly from the integer.
    # In a real system, the byte length would be known or encoded.
    return m.to_bytes((m.bit_length() + 7) // 8, 'big').decode('utf-8')

if __name__ == '__main__':
    # Generate a key pair (RSA-1024 equivalent, as each prime is 512 bits)
    public_key, private_key = generate_keypair(512)
    print("Public Key (e, n):", public_key)
    print("Private Key (d, n):", private_key)

    # Test with English plaintext
    plaintext_en = "Hello, RSA! This is a test message."
    print("\nOriginal English Plaintext:", plaintext_en)
    ciphertext_en = encrypt(public_key, plaintext_en)
    print("Ciphertext (English):", ciphertext_en)
    decrypted_text_en = decrypt(private_key, ciphertext_en)
    print("Decrypted English Plaintext:", decrypted_text_en)
    assert plaintext_en == decrypted_text_en

    # Test with numeric plaintext
    plaintext_num = "12345678901234567890"
    print("\nOriginal Numeric Plaintext:", plaintext_num)
    ciphertext_num = encrypt(public_key, plaintext_num)
    print("Ciphertext (Numeric):", ciphertext_num)
    decrypted_text_num = decrypt(private_key, ciphertext_num)
    print("Decrypted Numeric Plaintext:", decrypted_text_num)
    assert plaintext_num == decrypted_text_num

    print("\nRSA simulation successful!")



rsa-simulation
