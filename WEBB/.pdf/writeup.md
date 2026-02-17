# .pdf - CTF Writeup

**Category:** Web / Forensics  
**Flag:** `root{easy_if_u_follow}`

## Summary
This challenge hides the path to the flag across several layers: page source, Git history, Base64, ROT13, and a remote font file. The trick is to follow the hints that point you toward commit metadata rather than the working tree.

## Step 1: Source Recon
- Inspected the HTML source of the challenge page.
- Found a GitHub repo reference (shown as a hint in the page source).
- The README contains a clue about “ignoring commit history before push --force,” which strongly suggests looking at commits instead of files.

## Step 2: Git Forensics
Cloned the repo and reviewed commit history:

```bash
git clone https://github.com/Asif-Tanvir-2006/rootaccess_web_chal_pdf.git
git log --oneline
```

Two commits contained a suspicious Base64 blob:

```text
1725e55 dWdnY2Y6Ly9lbmoudHZndWhvaGZyZXBiYWcucGJ6L05mdnMtR25haXZlLTIwMDYvc2JhZ2Yvem52YS9zYmFnLmdncw==
8622249 dWdnY2Y6Ly9lbmoudHZndWhvaGZyZXBiYWcucGJ6L05mdnMtR25haXZlLTIwMDYvc2JhZ2Yvem52YS9zYmFnLmdncw==
```

## Step 3: Decode Chain
Decode Base64, then apply ROT13:

```text
Base64 ->
uggcf://enj.tvguhohfrepbagrag.pbz/Nfvs-Gnaive-2006/sbagf/znva/sbag.ggs

ROT13 ->
https://raw.githubusercontent.com/Asif-Tanvir-2006/fonts/main/font.ttf
```

## Step 4: Extract Flag
Downloaded the font and ran `strings` on it:

```bash
strings font.ttf
```

The flag is embedded in the font metadata:

```text
root{easy_if_u_follow}
```

## Tools Used
- Git: commit history inspection
- CyberChef: Base64 + ROT13 decoding
- `strings`: metadata extraction from the font file
