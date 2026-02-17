# The Library Code

**Category:** Cryptography  
**Artifact:** `encrypted_note.txt`  
**Flag:** `root{ii3st5_library}`

## Overview
This is a Caesar cipher with a hidden shift. The trick is that the key is embedded in the challenge description (the IP address), and the text keeps digits unchanged, which matches a simple alphabet-only shift.

## Step 1: Read the Clue
The description references “IIEST Library Question Bank” and the IP `10.11.1.6`, plus the hint “Sum all the essence within.” That strongly implies summing the digits:

```
1 + 0 + 1 + 1 + 1 + 6 = 10
```

So the Caesar shift is 10.

## Step 2: Decrypt
Encrypted message:

```
Dro pvkq sc ss3cd5_vslbkbi
```

Shift letters back by 10 (leave digits as-is):

```
The flag is ii3st5_library
```

## Flag
```
root{ii3st5_library}
```

## Tools Used
- Python: Caesar shift decryption
