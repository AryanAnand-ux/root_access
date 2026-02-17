# RootAccess CLI

**Category:** Reverse Engineering / Node.js  
**Artifact:** `root-access` (NPM package)  
**Flag:** `root{N0d3_Cl1_R3v3rs3_3ng1n33r1ng_M4st3r_0f_Sh4d0ws}`

## Overview
The CLI is heavily obfuscated, but it still exposes internal classes at runtime. Instead of untangling RC4 strings and flattened control flow, I loaded the package as a library and called the methods that already assemble the flag.

## Step 1: Locate the Payload
- Installed globally: `npm install -g root-access`
- Found the code at: `npm root -g` → `node_modules/root-access/dist/lib/`
- `package.json` shows `javascript-obfuscator` with RC4 string encoding, control flow flattening, dead-code injection, and unicode escapes
- Modules present: `index.js`, `crypto.js`, `keyforge.js`, `network.js`, `vault.js`

Key observation: each module uses `module.exports`, which means we can `require()` the classes directly.

## Step 2: Runtime Introspection
Instead of static reversing, I used dynamic discovery:
- Load modules in Node.js
- Enumerate class methods via `Object.getOwnPropertyNames()`

This revealed:
- `CryptoEngine.getTotalFragments()`
- `CryptoEngine.getFragment(n)`
- `CryptoEngine.__getFullFlag__()` (hidden static helper)
- `DataVault.assembleFlag()` (rebuild + hash)

## Step 3: Scripted Extraction
```js
const cryptoMod = require('root-access/dist/lib/crypto.js');
const CE = cryptoMod.CryptoEngine;

console.log(Object.getOwnPropertyNames(CE));

for (let i = 1; i <= CE.getTotalFragments(); i++) {
  console.log(`Fragment ${i}: ${CE.getFragment(i)}`);
}

console.log(CE.__getFullFlag__());
```

Each fragment is decoded at runtime as:
1. Base64 decode
2. XOR with key `SHADOWGATE`

## Step 4: Flag Assembly
Fragments from `CryptoEngine.getFragment(i)`:

| Fragment | Value |
| --- | --- |
| 1 | `root{N0` |
| 2 | `d3_Cl1_` |
| 3 | `R3v3rs3` |
| 4 | `_3ng1n3` |
| 5 | `3r1ng_` |
| 6 | `M4st3r` |
| 7 | `_0f_` |
| 8 | `Sh4d` |
| 9 | `0ws` |
| 10 | `}` |

`DataVault.assembleFlag()` also reports the SHA256 hash:
`0c950c774d2f7bbbb74c06d8abeba59e9679ee543d7928c320e90ebe6368aee6`

## Flag
```text
root{N0d3_Cl1_R3v3rs3_3ng1n33r1ng_M4st3r_0f_Sh4d0ws}
```

## Tools Used
- Node.js: runtime loading and method calls
- npm: package install and source discovery
- `Object.getOwnPropertyNames()`: API discovery on obfuscated classes
