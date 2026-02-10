# backup.go Fixed Metadata Changes

## IMPORTANT: Use Original Code, Not Modified Version

Your backup.go file contains a **modified version** that only engraves words 4-9. 
**This must be replaced with the original code** that engraves all 12 words.

Look for the commented section labeled "Good code / original code" - that's what should be used.

**HOWEVER:** Keep the QR code disabled as it is in your current modified version.

## Quick Reference

### Location: backup.go - frontSideSeed function

## Changes Required to ORIGINAL CODE

### 1. Change Version from "V1" to "SC"

**FIND (in original code):**
```go
const version = "V1"
```

**REPLACE WITH:**
```go
const version = "SC"
```

### 2. Change Page Number to Fixed "1/1"

**FIND (in original code):**
```go
page := fmt.Sprintf("%d/%d", plate.KeyIdx+1, plate.Keys)
```

**REPLACE WITH:**
```go
page := "1/1"  // Fixed value instead of dynamic
```

### 3. Fingerprint Already Uses plate.MasterFingerprint

**EXISTING CODE (no change needed here):**
```go
mfp := strings.ToUpper(fmt.Sprintf("%.8x", plate.MasterFingerprint))
```

This line doesn't need to change - it already uses `plate.MasterFingerprint`.
The change is in HOW that value gets set (from GUI manual input instead of auto-derivation).

### 4. DISABLE the QR Code Engraving

**FIND (in original code):**
```go
// Engrave seed QR.
qrCmd, err := engrave.ConstantQR(strokeWidth, 3, qr.Q, seedqr.CompactQR(plate.Mnemonic))
if err != nil {
	return nil, err
}
qr, sz := dims(qrCmd)
cmd(engrave.Offset(scale(60)-sz.X/2, (plateDims.Y-sz.Y)/2, qr))
```

**REPLACE WITH (comment it out):**
```go
// QR code disabled
// qrCmd, err := engrave.ConstantQR(strokeWidth, 3, qr.Q, seedqr.CompactQR(plate.Mnemonic))
// if err != nil {
// 	return nil, err
// }
// qr, sz := dims(qrCmd)
// cmd(engrave.Offset(scale(60)-sz.X/2, (plateDims.Y-sz.Y)/2, qr))
```

### 5. KEEP All Word Engraving Logic

**DO NOT use the modified version that has:**
```go
startWord := 3  // Words 4-9 only - WRONG
endWord := 9
```

**DO use the original that has:**
```go
maxCol1 := 16   // Column 1: words 1-16
maxCol2 := 4    // Column 2: words 17-20 (top)
// Plus bottom of column 2 for words 21-24
// NO QR code (disabled)
```

### 1. Change Version from "V1" to "SC"

**FIND:**
```go
const version = "V1"
```

**REPLACE WITH:**
```go
const version = "SC"
```

### 2. Change Page Number to Fixed "1/1"

**FIND:**
```go
page := fmt.Sprintf("%d/%d", plate.KeyIdx+1, plate.Keys)
```

**REPLACE WITH:**
```go
page := "1/1"  // Fixed value instead of dynamic
```

### 3. Fingerprint Already Uses plate.MasterFingerprint

**EXISTING CODE (no change needed here):**
```go
mfp := strings.ToUpper(fmt.Sprintf("%.8x", plate.MasterFingerprint))
```

This line doesn't need to change - it already uses `plate.MasterFingerprint`.
The change is in HOW that value gets set (from GUI manual input instead of auto-derivation).

## Complete Modified Section

Here's what the section should look like after changes:

```go
// Engrave version, mfp and page.
const version = "SC"  // CHANGED: was "V1"
margin := scale(outerMargin)
innerMargin := scale(innerMargin)
metaMargin := scale(4)
page := "1/1"  // CHANGED: was fmt.Sprintf("%d/%d", plate.KeyIdx+1, plate.Keys)
mfp := strings.ToUpper(fmt.Sprintf("%.8x", plate.MasterFingerprint))
// plate.MasterFingerprint now contains manual fingerprint from GUI
```

## What Gets Engraved

After these changes, every plate will show:

| Field       | Old Behavior          | New Behavior      |
|-------------|----------------------|-------------------|
| Version     | "V1"                 | "SC"              |
| Page        | "1/2", "2/3", etc.   | "1/1"             |
| Fingerprint | Auto-derived from seed | Manual user input |
| Seed Words  | All 12 (or 24)       | All 12 (or 24)    |
| QR Code     | ✓ Included           | ✗ Disabled        |
| Title       | ✓ Included           | ✓ Included        |

**CRITICAL:** Must use original code that engraves ALL words, but keep QR code disabled.

## Testing

1. After making changes, rebuild the firmware
2. Scan a test seed QR
3. Enter a test fingerprint (e.g., "12345678")
4. Verify engraved plate shows:
   - Version: "SC"
   - Page: "1/1"
   - MFP: "12345678"
