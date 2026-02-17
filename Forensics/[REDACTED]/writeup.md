# [REDACTED] - CTF Writeup

**Category:** Misc / Steganography  
**Artifact:** `[REDACTED].pdf`  
**Flag:** `root{EASY_2_UNREDACT}`

## Summary
The PDF was visually redacted with black rectangles, but the underlying text was still present. Removing the overlay objects exposes the hidden flag.

## Step 1: Inspection
- Opened the PDF in Inkscape.
- Noticed the black bars were separate vector objects layered on top of text.
- This indicates visual masking, not true redaction.

## Step 2: Unredaction
- Selected each black rectangle.
- Deleted the overlay objects.
- The covered text became visible immediately.

## Result
**Flag:** `root{EASY_2_UNREDACT}`

## Tools Used
- Inkscape: PDF editing and object deletion
