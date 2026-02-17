# Echoes of the Future

**Category:** Cryptography  
**Artifacts:** `vault.docx`, `vaultkey.txt`, `lmao`  
**Flag:** `root{Emmanuel_Goldstein}`

## Summary
This puzzle chains three layers: a riddle that yields the document password, a Huffman tree that decodes a huge bitstream, and a monoalphabetic substitution that converts the decoded text into readable English. The final clue is the author named in the recovered quote.

## Step 1: Unlock the Vault
`vaultkey.txt` points to “D Albert’s tree” and “DOH, the Firefox,” which hints at two David A. Huffmans. The “third film” clue resolves to *The Onion Field*, which is the password for `vault.docx`.

## Step 2: Rebuild the Huffman Tree
Inside the document:
- A 66,080-bit string.
- A riddle describing a prefix-free code with fixed leaf order.
- Constraints like space encoded as `111`.

Those constraints define a mostly fixed tree with a small ambiguous branch. I brute-forced the few valid variants, selected the tree that fully decodes the bitstream, and produced a monoalphabetic ciphertext.

## Step 3: Solve the Substitution
Frequency analysis and word patterns (`kya` → `the`, `fs` → `of`, `wul` → `and`) yield a clean substitution map. Applying it reveals a passage from *Nineteen Eighty-Four* (the fictional book inside it).

## Extraction
The passage is attributed to **Emmanuel Goldstein**, which is the flag:

```
root{Emmanuel_Goldstein}
```

## Tools Used
- Python: Huffman reconstruction, decoding, substitution solve
- `msoffcrypto-tool`: decrypting the DOCX
- `olefile`: OLE inspection / extraction
