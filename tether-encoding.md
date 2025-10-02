# Tether Encoding Reference

Tether is a binary protocol for remote communication between Caffeine
object memories. Each object is encoded as a tag, which is a 32-bit
word (4 bytes, big-endian), followed by zero or more payload words.

## Tag Space Layout

The 32-bit tag space is divided into ranges:

| Range | Base Value | Purpose |
|-------|-----------|---------|
| `0x70000000` - `0x7FFFFFFF` | `0x70000000` | instructions |
| `0x60000001` - `0x6FFFFFFF` | `0x60000001` | object proxies |
| `0x40000000` - `0x5FFFFFFF` | `0x40000000` | SmallIntegers |
| `0x20000000` - `0x3FFFFFFF` | `0x20000000` | class tags and special literal values |

## Decoding Algorithm

Read the first 32-bit word. Determine its type by comparing against
the base values:

```
if (tag >= 0x70000000) → instruction
else if (tag >= 0x60000001) → object proxy
else if (tag >= 0x40000000) → a SmallInteger
else if (tag >= 0x20000000) → class tag or special value
else → Error: unsupported encoding
```

## Small Integers

**Encoding:** `value + 0x40000000` (non-negative integers only in
current implementation)

**Decoding:** `tag - 0x40000000`

**Examples:**
- `3` → `0x40000003`
- `4` → `0x40000004`
- `7` → `0x40000007`
- `0` → `0x40000000`

**Note:** Negative integers require a `negativeNumberTag` (not yet
implemented in tether.js).

## Special Values

Fixed tags for common objects:

| Value | Tag |
|-------|-----|
| `true` | `0x20000001` |
| `false` | `0x20000002` |
| `nil` | `0x20000003` |

## Class Tags (Structured Objects)

When `tag >= 0x20000000` and `tag < 0x40000000`, the tag indicates the
object's class. The tag is followed by additional data specific to
that class.

### String (`0x20000005`)

Format:
```
[stringTag: 4 bytes][length: 4 bytes][characters: length bytes]
```

**Decoding:**
1. Read tag `0x20000005`
2. Read length (32-bit word)
3. Read `length` bytes as UTF-8 characters

### Array (`0x20000008`)

Format:
```
[arrayTag: 4 bytes][length: 4 bytes][element1][element2]...[elementN]
```

**Decoding:**
1. Read tag `0x20000008`
2. Read length (32-bit word)
3. Recursively decode `length` elements

### ByteArray (`0x2000001B`)

Format:
```
[byteArrayTag: 4 bytes][length: 4 bytes][bytes: length bytes]
```

**Decoding:**
1. Read tag `0x2000001B`
2. Read length (32-bit word)
3. Read `length` raw bytes

### Answer (`0x2000001D`)

Marks a response to a message send. Followed by the encoded result
object.

## Proxy Markers (Remote Object References)

**Encoding:** `exposureHash + 0x60000001`

**Decoding:** `tag - 0x60000001` gives the exposure hash

These represent objects in the remote object memory. The exposure hash
is the object's identity hash in its home image.

## Answer Format

**All MCP tool responses are wrapped in an answer tag.**

Format:
```
[answerTag: 4 bytes][result encoding...]
```

The `answerTag` is `0x2000001D`. After reading this tag, decode the
following bytes as the actual result.

## Practical Decoding Example

Given the result of `3 + 4`:

```
Bytes: [0x20, 0x00, 0x03, 0xBD] [0x40, 0x00, 0x00, 0x07]
       ↑ answerTag (0x2000001D)   ↑ smallInteger 7 (0x40000007)

Step 1: Read first word → 0x2000001D (answerTag)
Step 2: Read next word → 0x40000007
Step 3: Check: 0x40000007 >= 0x40000000 → a SmallInteger
Step 4: Decode: 0x40000007 - 0x40000000 = 7
```

Given a string result `"hello"`:

```
Bytes: [0x20, 0x00, 0x03, 0xBD] [0x20, 0x00, 0x00, 0x05] [0x00, 0x00, 0x00, 0x05] [0x68, 0x65, 0x6C, 0x6C, 0x6F]
       ↑ answerTag              ↑ stringTag (0x20000005)  ↑ length: 5            ↑ "hello"

Step 1: Read answerTag (0x2000001D)
Step 2: Read stringTag (0x20000005)
Step 3: Read length (0x5)
Step 4: Read 5 bytes as characters: "hello"
```

## Usage in MCP Tool

The `send` tool returns Tether-encoded bytes. To interpret the result:

1. Read the first 4 bytes as a 32-bit big-endian word (should be `answerTag` or `0x2000001D`)
2. Read the next 4 bytes to determine the result type
3. Apply the decoding algorithm based on the result type
4. If it's a class tag, read additional payload data
5. If it's a SmallInteger, subtract the base to get the value
6. If it's a proxy marker, it's a remote object reference (identity hash)

**Note:** Exception handling and other error conditions are not yet
supported.

## Encoding for Agents

When calling the `send` tool, you must construct Tether-encoded hex
strings. This requires careful attention to the encoding format.

### Critical Rule: 8 Hex Characters Per Word

**Every 32-bit word must be exactly 8 hexadecimal characters.** Use
leading zeros to pad.

### Encoding Small Integers

**Formula:** `(value + 0x40000000).toString(16).padStart(8, '0')`

**Step-by-step for encoding `3`:**
1. Add base: `3 + 0x40000000 = 0x40000003`
2. Format as 8 chars: `40000003` ✓

**Step-by-step for encoding `4`:**
1. Add base: `4 + 0x40000000 = 0x40000004`
2. Format as 8 chars: `40000004` ✓

**Common mistake:** Writing `400000004` (9 characters) - this is wrong!

### Encoding Arrays

**Format:** `[arrayTag][length][element1][element2]...[elementN]`

**Step-by-step for encoding `[4]` (array with one element):**
1. Array tag: `0x20000008` (8 chars)
2. Length: `1` → `0x00000001` (8 chars)
3. Element (SmallInteger 4): `40000004` (8 chars, from above)
4. Concatenate: `20000008` + `00000001` + `40000004` = `200000080000000140000004`
5. Verify: 24 characters = 3 words × 8 chars ✓

### Validation Checklist

Before sending a Tether-encoded parameter:

1. ☐ Count total hex characters - must be divisible by 8
2. ☐ Verify each word is exactly 8 characters
3. ☐ Check for accidental extra/missing zeros
4. ☐ Confirm leading zeros are present where needed

### Example: Encoding `3 + 4`

**Receiver (SmallInteger 3):**
```
Value: 3
Tagged: 40000003
Length: 8 characters ✓
```

**Arguments (Array containing SmallInteger 4):**
```
Array tag:    20000008  (8 chars)
Length:       00000001  (8 chars)
Element:      40000004  (8 chars)
Concatenated: 200000080000000140000004
Total length: 24 characters = 3 words ✓
```

**Selector:** `+` (plain string, no encoding needed)

**Result:** For this example the tool returns `{"result": "40000007"}` - the Smalltalk side decodes the Tether, performs the computation, and returns the result as a Tether-encoded hex string.
