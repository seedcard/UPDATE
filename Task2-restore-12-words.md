# Task#2 - Restore Full 12-Word Engraving

## Current Issue

The `backup.go` file has been modified to only engrave **words 4-9** (6 words). This modified version needs to be **removed** and replaced with the original code that engraves all 12 words.

## What Needs to Happen

### REMOVE this modified code (around line 240 in backup.go):

```go
// REMOVE THIS SECTION - Modified version that only engraves 6 words
func frontSideSeed(scale func(float32) int, strokeWidth int, plate Seed, plateDims image.Point) (engrave.Command, error) {
	constant := engrave.NewConstantStringer(plate.Font, scale(plateFontSize), bip39.ShortestWord, bip39.LongestWord)
	var cmds engrave.Commands
	cmd := func(c engrave.Command) {
		cmds = append(cmds, c)
	}

	// Define the range of words to engrave: 4 to 9
	startWord := 3 // zero-based index (4th word)
	endWord := 9   // zero-based index (10th word)
	// ... rest of modified code
}
```

### RESTORE the original code:

The original `frontSideSeed` function should engrave:
- **Column 1:** Words 1-16 (or all 12 words for 12-word seeds)
- **Column 2:** Words 17-24 (if 24-word seed)
- **QR Code:** In the middle
- **Metadata:** Version, fingerprint, page number

The commented-out "Good code / original code" in your backup.go file shows the correct version.

## Original Layout (What You Want - but NO QR code)

```
┌─────────────────────────────┐
│ SC    [Fingerprint]    1/1  │  ← Metadata (changed to SC and 1/1)
├─────────────────────────────┤
│  1 WORD1                    │
│  2 WORD2                    │
│  3 WORD3                    │
│  4 WORD4      (NO QR)       │  ← QR disabled
│  ...                        │
│ 12 WORD12                   │
├─────────────────────────────┤
│         [Title]             │  ← Card name on right or bottom
└─────────────────────────────┘
```

## Modified Layout (Current - NOT WANTED)

```
┌─────────────────────────────┐
│ V1    [Fingerprint]    1/1  │
├─────────────────────────────┤
│  4 WORD4                    │
│  5 WORD5                    │
│  6 WORD6      (NO QR)       │
│  7 WORD7                    │
│  8 WORD8                    │
│  9 WORD9                    │
├─────────────────────────────┤
│         [Title]             │
└─────────────────────────────┘
```

## Instructions for Developer

1. **Use the original/commented "Good code" version** of `frontSideSeed` in backup.go
2. **Make these changes** to the original:
   - Change `const version = "V1"` to `const version = "SC"`
   - Change `page := fmt.Sprintf("%d/%d", plate.KeyIdx+1, plate.Keys)` to `page := "1/1"`
   - Keep `mfp := strings.ToUpper(fmt.Sprintf("%.8x", plate.MasterFingerprint))` (will use manual input)
   - **DISABLE the QR code** by commenting out the QR engraving section
3. **Keep all the word column engraving** - DO NOT remove any word engraving logic
4. **Result:** All 12 words engraved + NO QR code + manual fingerprint + "SC" + "1/1" + title

## Seed QR Scanning

**CONFIRMATION:** The seed QR scanning functionality is already working correctly:
- Scans standard SeedQR format
- Automatically detects 12-word or 24-word seeds
- Handles UR-encoded seeds
- No changes needed to scanning functionality

The `ScanScreen` in gui.go handles all of this automatically.

## Summary of What Gets Engraved (Final Version)

✓ **All 12 seed words** (or 24 if scanned)
✗ **QR code** - DISABLED (commented out)
✓ **Manual fingerprint** (from GUI input)
✓ **Version:** "SC" (not "V1")
✓ **Page:** "1/1" (not dynamic)
✓ **Title/Card Name** (if provided) - right side or bottom depending on plate size
