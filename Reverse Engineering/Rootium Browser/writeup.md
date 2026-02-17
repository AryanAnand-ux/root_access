# RootAccess CLI - CTF Writeup

**Category:** Reverse Engineering / Node.js  
**Challenge:** RootAccess CLI (NPM Package)  
**Method:** Runtime Introspection / Module Abuse

## Approach
The target is a published NPM package (`root-access`) with a heavily obfuscated CLI. The flag is split into 10 XOR-encrypted fragments inside the package modules. Instead of reversing RC4-encoded strings and flattened control flow, I loaded the modules directly in Node.js and called the exported methods to extract and recombine the flag.

## Step 1: Initial Analysis
- Installed the package globally: `npm install -g root-access`
- Located the source: `npm root -g` → `node_modules/root-access/dist/lib/`
- `package.json` shows `javascript-obfuscator` with RC4 string encoding, control flow flattening, dead-code injection, and unicode escapes
- Obfuscated modules present: `index.js`, `crypto.js`, `keyforge.js`, `network.js`, `vault.js`
- Key insight: every module uses `module.exports` to expose classes, so `require()` works without deobfuscation

## Step 2: Core Technique
I used dynamic analysis instead of static reversing:
- Load each module in Node.js
- Enumerate class methods with `Object.getOwnPropertyNames()`

This works because the obfuscation makes the source painful to read, but the runtime still exposes clean class interfaces. That means we can call the interesting methods directly.

Key findings:
- `CryptoEngine.getFragment(n)` returns fragment `n`
- `CryptoEngine.__getFullFlag__()` returns the full flag in one shot
- `DataVault.assembleFlag()` rebuilds the flag and reports a SHA256 hash

## Step 3: Implementation
```js
const cryptoMod = require('root-access/dist/lib/crypto.js');
const CE = cryptoMod.CryptoEngine;

// Enumerate methods
console.log(Object.getOwnPropertyNames(CE));

// Extract all fragments
for (let i = 1; i <= CE.getTotalFragments(); i++) {
  console.log(`Fragment ${i}: ${CE.getFragment(i)}`);
}

// Or get the full flag directly
console.log(CE.__getFullFlag__());
```

At runtime, each fragment is decoded by:
1. Base64 decode
2. XOR with the key `SHADOWGATE`

## Step 4: Extraction
Fragments returned by `CryptoEngine.getFragment(i)`:

| Fragment | Value |
| --- | --- |
| 1 | `root` |
| 2 | `{n0_` |
| 3 | `m0r3` |
| 4 | `_34s` |
| 5 | `y_v4` |
| 6 | `ult_` |
| 7 | `3xtr` |
| 8 | `4ct1` |
| 9 | `0n_9` |
| 10 | `9}` |

Concatenating yields the flag. I also verified with `DataVault.assembleFlag()`, which prints the SHA256 hash:
`c8cda0cca11a25de0c8644e9c6e7d1004187514062a374e222dc5b144261dda4`

## Flag
```text
root{n0_m0r3_34sy_v4ult_3xtr4ct10n_99}
```

## Tools Used
- Node.js: runtime execution and method calls
- npm: package install and source discovery
- `Object.getOwnPropertyNames()`: API discovery on obfuscated classes
