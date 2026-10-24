# Timestamped Secrets
Category: Cryptography

Author: Yahaya Meddy

# Challenge Description:

Someone encrypted a message using AES in ECB mode, but they were not careful with the key. The key is made from something simple: the current time. Find the key and decrypt the flag.

Hint: encryption.py is a redacted example of the program.

(Challenge page on picoCTF)

# Files:

| File | Description |
|------|-------------|
| message.txt | the approximate time of encryption and the ciphertext in hex |
| encryption.py | a redacted copy of the program that encrypted the message |

[encryption.py](https://github.com/user-attachments/files/33907464/encryption.py)
[message.txt](https://github.com/user-attachments/files/33907463/message.txt)

# Analysis:

First I opened message.txt. It had two things:

<img width="729" height="45" alt="Image" src="https://github.com/user-attachments/assets/4f36912c-54f0-4f77-804e-0a9ea7b0d719" />

The hint is a Unix timestamp. It is the number of seconds since 1 January 1970. It goes up by 1 every second, so 1790147508 means one exact second.

Then I opened encryption.py. These were the important lines:

<img width="472" height="105" alt="Image" src="https://github.com/user-attachments/assets/94cd6d09-1058-402d-9213-f6949f661cf2" />

Reading it line by line:

#### time.time(): gets the current time in seconds
#### sha256: turns that number into 32 bytes
#### [:16]: keeps the first 16 bytes, and this is the AES key
#### AES.MODE_ECB: encrypts the message with that key
#### pad: fills the message up to a multiple of 16 bytes

The script is redacted, so it does not run as it is. But it shows everything I needed.

### Why this is a problem?

The key is not random. It is made from the time, and the hint tells us roughly what the time was.
SHA-256 looks random, but the same input always gives the same output. So if I guess the right second, I get the right key.
Normally AES-128 has 2^128 possible keys. Here the key can only be one of a few thousand values, one for each second near the hint.
A safe key must come from a secure random generator, not from the clock.

# **Conceptually:**

Hint (approximate time)

```
    ↓
```

Try every second near the hint

```
    ↓
```

SHA-256 of the second, keep the first 16 bytes = candidate key

```
    ↓
```

Decrypt the ciphertext with AES-ECB

```
    ↓
```

Check if the result is readable text with valid padding

```
    ↓
```

Right key found = flag

### How do I know the key is correct?

A wrong key gives random-looking bytes. The right key gives readable letters.

There is also padding. The ciphertext is 32 bytes, which is 2 blocks. The message is padded to fill the last block, so the last bytes must be a valid padding. For example, if 5 bytes are missing, the last 5 bytes are all 05.

So I accepted a key only if the padding was valid and every other byte was printable text.

# Solving:

I wrote a Python script that tries every second around the hint, starting from the hint and moving outward:
(took help from AI for writing script)

<img width="758" height="500" alt="Image" src="https://github.com/user-attachments/assets/ac1aec5c-2b50-4852-a657-51b5bf88873d" />

What the script does:

Takes a second near the hint
Makes the key from it with SHA-256
Decrypts the ciphertext
Checks the padding and the text
If it fails(gives gibberish/random key), goes to the next second

The script found the key at timestamp 1790147508, which is 0 seconds away from the hint. The decrypted text was the flag.

# 🚩 Flag:

academy{sa3S_sEc9t_65bbf411}

# Concepts Used:

-->Unix timestamps

-->SHA-256 hashing

-->AES-128 in ECB mode

-->PKCS#7 padding

-->Brute force on a very small key space
