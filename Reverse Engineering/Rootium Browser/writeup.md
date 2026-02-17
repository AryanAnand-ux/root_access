# RootAccess CLI - CTF Solution

**Category:** Reverse Engineering / Node.js  
**Challenge:** RootAccess CLI (NPM Package)  
**Method:** Runtime Module Hijacking

## Challenge Description
The challenge provides a published NPM package (`root-access`) containing a heavily obfuscated CLI tool. The tool simulates a hacking interface with multiple "security layers" (passwords, time-based checks, key generation). The source code uses `javascript-obfuscator` with RC4 string encryption and control flow flattening, making static analysis extremely tedious.

The flag is split into 10 fragments hidden across the package's JavaScript modules.

## Solution Strategy: "Library Abuse"
Instead of manually reversing the obfuscated logic or solving the 10 layers of CLI puzzles, I attacked the architecture of the application itself.

Since the tool is written in JavaScript and runs on Node.js, it must export its internal modules to function. By inspecting the file structure in `node_modules`, I identified the core logic files. The strategy was to import the obfuscated code as a library and force it to decrypt the flag.

### 1. Reconnaissance
I navigated to the package installation directory:

```bash
# Locate the package source
cd node_modules/root-access/dist/lib/
ls -la
```

I found `crypto.js`, which handles the encryption logic. Loading this module in a Node.js REPL revealed it exports a class named `CryptoEngine`.

### 2. The Exploit
Rather than solving the challenges intended for the user, I wrote a script to:

- Bypass the CLI entry point (`bin/rootaccess`).
- Directly `require()` the internal `crypto.js` module.
- Instantiate the `CryptoEngine` class.
- Call the internal methods to dump the decrypted flag.

During inspection, I found a static method `__getFullFlag__()` which appeared to return the combined flag string.

### 3. Exploit Code (`exploit.js`)
```js
const path = require('path');

// Target the internal library file directly
// Adjust the path based on your local install location
// Usually: ./node_modules/root-access/dist/lib/crypto.js
const LIB_PATH = require.resolve('root-access/dist/lib/crypto.js');

console.log("[*] Hijacking internal CryptoEngine...");

try {
  // 1. Load the obfuscated module
  const targetModule = require(LIB_PATH);
  const Engine = targetModule.CryptoEngine;

  // 2. Inspect available methods (Recon)
  console.log("[+] Module loaded. Methods detected:", Object.getOwnPropertyNames(Engine));

  // 3. Trigger the hidden assembly function
  // This bypasses the need for the Master Key or individual fragment collection
  console.log("[*] Executing internal decryption routine...");

  // The engine decrypts itself when we call this
  const secret = Engine.__getFullFlag__();

  console.log("\\n[SUCCESS] Flag recovered:");
  console.log(secret);
} catch (err) {
  console.error("[-] Exploit failed:", err.message);
}
```

## Flag
Running the exploit script instantly returned the assembled flag:

```text
root{N0d3_Cl1_R3v3rs3_3ng1n33r1ng_M4st3r_0f_Sh4d0ws}
```

## Why This Works
The obfuscation protected the source code from being read by humans, but it did not protect the runtime objects from being accessed by other code. By using `require()`, we operate inside the trusted memory space of the application, allowing us to call functions that were intended to be private or internal.
