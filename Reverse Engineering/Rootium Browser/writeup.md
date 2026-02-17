# RootAccess CLI - CTF Writeup

## 🕵️ Challenge Overview
**Category:** Reverse Engineering / Node.js  
**Target:** `root-access` (NPM Package)  

The challenge involves a CLI tool distributed as an NPM package. The source code is heavily obfuscated using techniques like control flow flattening and string encryption (RC4), making static analysis painful. The goal is to recover the flag, which is fragmented and hidden within the application's logic.

## 🧠 Strategic Approach: "Library Abuse"

Instead of treating the challenge as a binary to be decompiled, I treated it as a **library to be imported**. 

Since the application is written in JavaScript and runs on Node.js, the obfuscated modules still need to export their functionality to work. By inspecting the `package.json` and file structure, we can identify the internal modules. Instead of manually reversing the decryption algorithms, we can write a wrapper script to `require()` these internal modules and force them to run their own decryption logic for us.

### 1. Reconnaissance
I started by locating the installed package source code:
```bash
npm list -g root-access
# Location: .../node_modules/root-access/dist/lib/

Listing the files in dist/lib/ revealed several interesting modules:

crypto.js: Likely handles encryption/decryption.

vault.js: Likely storage for the flag.

keyforge.js: Handles key generation.

2. The Exploit
I focused on crypto.js. Loading it into a Node REPL showed that it exports a class named CryptoEngine.

Rather than reverse-engineering the XOR key or the Base64 alphabet used by the obfuscator, I simply instantiated the class and looked for methods that might return data.

Inspection revealed a static method __getFullFlag__ and an instance method getFragment. The obfuscation protected the source code, but the runtime interface was wide open.

3. Solution Script
I wrote a short script (exploit.js) to hijack the module:

// exploit.js
// Bypass the CLI interface and load the internal crypto module directly
const path = require('path');

// Adjust path to where the package is installed
const cryptoPath = 'root-access/dist/lib/crypto.js'; 

try {
    console.log("[*] Attempting to load internal module...");
    const CryptoModule = require(cryptoPath);
    
    // The class is exposed via exports
    const Engine = CryptoModule.CryptoEngine;

    console.log("[+] Module loaded. Exploiting static methods...");

    // Method 1: Dump individual fragments
    const count = Engine.getTotalFragments(); // Returns 10
    console.log(`[i] Found ${count} encrypted fragments.`);
    
    let reassembled = "";
    for(let i=1; i <= count; i++) {
        // The engine decrypts itself when we call this
        const frag = Engine.getFragment(i); 
        console.log(`    Fragment ${i}: ${frag}`);
    }

    // Method 2: The "Lazy" Static Call
    // Found this hidden method during runtime inspection
    console.log("\n[!] Triggering internal flag assembly...");
    if (typeof Engine.__getFullFlag__ === 'function') {
        const flag = Engine.__getFullFlag__();
        console.log(`\n🚩 FLAG: ${flag}`);
    }

} catch (e) {
    console.error("[-] Exploit failed:", e.message);
}

🏁 The Result
Running the script bypassed all the CLI arguments, passwords, and "Key Forge" mechanics entirely. The code simply decrypted the strings and handed them over.

Final Flag:

root{N0d3_Cl1_R3v3rs3_3ng1n33r1ng_M4st3r_0f_Sh4d0ws}

🛠️ Tools Used
Node.js: For executing the attack script.

VS Code: For inspecting module.exports definitions.

CommonJS: The exploit relied on the standard require behavior.