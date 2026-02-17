# Root of Trust - Boot Integrity

**Category:** Reverse Engineering  
**Artifact:** `custom_root_access.iso`  
**Flag:** `root{Ch41n_0f_Tru5t_Br0k3n}`

## Summary
The ISO boots into a locked verification UI with no shell, so the only viable path is to extract the boot artifacts and reverse the validation binary offline. The check routine applies a ROT13 transform to letters and then compares bytes via XOR masks, which can be inverted to recover the expected payload.

## Step 1: ISO Recon
- Unpacked the ISO (7-Zip) and found `vmlinuz` plus `initramfs.cpio.gz` under `boot/`.
- Decompressed the initramfs and located a single statically linked ELF named `init`.
- `strings` showed prompts like `Enter Authentication Credentials (Flag)` and an embedded `root{` prefix in `.rodata`.

## Step 2: Static Reversal
I disassembled the `init` binary (Capstone + Python) and traced the references to the prompt strings. The validation flow is:
- Read user input.
- Enforce length: 21 chars inside `root{...}`.
- Apply ROT13 to alphabetic characters.
- XOR-compare against constants (masks `0xAA` / `0x55`) stored as `movabs` immediates.

Once the transform chain was clear, the solution was to recover the comparison bytes from the immediates and reverse the XOR and ROT13 steps.

## Step 3: Extraction
The recovered payload is:

```
Ch41n_0f_Tru5t_Br0k3n
```

Wrapping with the required format produces the final flag:

```
root{Ch41n_0f_Tru5t_Br0k3n}
```

## Tools Used
- 7-Zip: ISO and initramfs extraction
- Capstone (Python): disassembly and control-flow tracing
- Python: constant extraction and transform inversion
