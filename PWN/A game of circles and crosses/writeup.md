# A Game of Circles and Crosses

**Category:** PWN / Web  
**Artifact:** `bugs.py` (Tkinter client)  
**Flag:** `root{M@yb3_4he_r3@!_tr3@5ur3_w@$_th3_bug$_w3_m@d3_@l0ng_4h3_w@y}`

## Summary
The client is a broken Tic-Tac-Toe implementation. Rather than wrestling with faulty win checks, the clean path is to understand the server verification protocol and submit a winning move sequence directly to the API.

## Step 1: Initial Analysis
- Reviewed `bugs.py` to understand the flow and data structures.
- The client sends proofs to `https://rootaccess.pythonanywhere.com/verify`.
- Board uses mixed types (`"3"` for empty, `0` for player, `1` for AI), which breaks validation logic.
- Win detection and occupancy checks are inconsistent.
- Moves are recorded as `{ "x": col, "y": row }` coordinates.

## Step 2: Core Technique (API Bypass)
The server is authoritative. If it validates a move sequence as a win, it returns the flag regardless of client-side bugs. The strategy:
- Craft a valid sequence of human moves.
- Encode them using the same proof format as the client.
- POST directly to the verification endpoint.

## Step 3: Implementation
Payload structure:

```text
payload = {
  "v": 1,
  "nonce": <random_hex>,
  "human": [ {x,y}, {x,y}, {x,y} ]
}
proof = base64_urlsafe(json.dumps(payload))
```

Winning example (first column):

```text
(0,0), (0,1), (0,2)
```

## Step 4: Extraction
Sending the proof returns:

```json
{"ok": true, "result": "player", "flag": "..."}
```

Rows, columns, and diagonals all work as long as the sequence is a valid win.

## Flag
```text
root{M@yb3_4he_r3@!_tr3@5ur3_w@$_th3_bug$_w3_m@d3_@l0ng_4h3_w@y}
```

## Tools Used
- Python 3: crafting and sending the verification payload
