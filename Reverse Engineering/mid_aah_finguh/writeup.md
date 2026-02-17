# mid_aah_finguh - CTF Writeup

**Category:** Reverse Engineering  
**Artifact:** `middle_fingu.png`  
**Flag:** `root{jjkr3f3r3nc354r3fun}`

## Overview
The challenge hands over a tiny PNG that is far too large for its dimensions. That usually means extra payloads are appended. I treated it like a container and chased embedded artifacts instead of trying to "reverse" the pixels.

## Step 1: Triage
Quick checks showed the PNG is 100x159 but weighs ~63 KB, which is suspicious. A `strings` pass revealed:
- A Base64 blob.
- ELF-related symbols (`/lib64/ld-linux-x86-64.so.2`, `__libc_start_main`).

`binwalk` confirmed the structure:

```text
PNG image @ 0x0
ELF 64-bit executable @ 0x6D0C
PDF document @ 0xCA9C
```

So the PNG hides an ELF and a PDF.

## Step 2: The ELF Rabbit Hole
I carved the ELF out with `dd`, made it executable, and ran it. It has functions that print taunting strings and apply XORs to user input, but no obvious flag path. After a few checks, it was clearly a decoy.

## Step 3: The PDF and the Hint
The Base64 decodes to:

```text
YOU_CAN_ONLY_UNLOCK_ME_WHEN_I_AM_AT_MY_FULL_POWER
```

That’s a JJK hint: Sukuna hits full power at 20 fingers, so the PDF password is `20`. Extracted and opened the PDF using that password.

Inside the page stream, the flag text is stored as hex glyph IDs in a custom font. The glyphs aren’t ASCII, so I parsed the font’s `/ToUnicode` CMap to map glyph IDs back to characters.

Decoded glyph string:

```text
7**1>//.7v#v7v+&vpq7v#0+8
```

## Step 4: XOR Recovery
Assuming the flag starts with `root{`, I XOR’d the first five characters and got a single-byte key `0x45` (`'E'`). Applying XOR `0x45` to the full string yields:

```text
root{jjkr3f3r3nc354r3fun}
```

## Result
**Flag:** `root{jjkr3f3r3nc354r3fun}`

## Tools Used
- `file`, `strings`, `binwalk`: identifying embedded payloads
- `dd`: carving the ELF and PDF
- Python: CMap parsing and XOR decode
