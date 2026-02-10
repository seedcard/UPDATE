# SeedHammer GUI - Manual Fingerprint Input Modification

## Overview
Modify the SeedHammer GUI to **require** manual fingerprint entry before engraving. The fingerprint is never auto-derived. Additionally, set fixed metadata fields:
- **Page number:** Always "1/1" (instead of dynamic)
- **Version:** Always "SC" (instead of "V1")

## Files to Modify

### 1. Add FingerprintInputScreen struct (add to gui.go)

Add this new screen after the `ConfirmWarningScreen` struct (around line 600):

```go
type FingerprintInputScreen struct {
	fingerprint [8]rune
	cursorPos   int
	scroll      int
}

func NewFingerprintInputScreen() *FingerprintInputScreen {
	return &FingerprintInputScreen{
		fingerprint: [8]rune{'0', '0', '0', '0', '0', '0', '0', '0'},
		cursorPos:   0,
	}
}

func (s *FingerprintInputScreen) Update(ctx *Context) (string, Result) {
	for {
		e, ok := ctx.Next(Button1, Button3, Up, Down, Left, Right)
		if !ok {
			break
		}
		
		switch e.Button {
		case Button1: // Back/Cancel
			if e.Click {
				return "", ResultCancelled
			}
		case Button3: // Confirm manual fingerprint (REQUIRED)
			if e.Click {
				fp := string(s.fingerprint[:])
				return fp, ResultComplete
			}
		case Left:
			if e.Pressed && s.cursorPos > 0 {
				s.cursorPos--
			}
		case Right:
			if e.Pressed && s.cursorPos < 7 {
				s.cursorPos++
			}
		case Up:
			if e.Pressed {
				s.incrementChar(1)
			}
		case Down:
			if e.Pressed {
				s.incrementChar(-1)
			}
		}
	}
	return "", ResultNone
}

// incrementChar changes the current character up or down through valid hex chars
func (s *FingerprintInputScreen) incrementChar(delta int) {
	validChars := "0123456789abcdef"
	current := s.fingerprint[s.cursorPos]
	
	// Find current position in valid chars
	currentIdx := strings.IndexRune(validChars, unicode.ToLower(current))
	if currentIdx == -1 {
		currentIdx = 0
	}
	
	// Increment/decrement with wrapping
	newIdx := (currentIdx + delta + len(validChars)) % len(validChars)
	s.fingerprint[s.cursorPos] = rune(validChars[newIdx])
}

func (s *FingerprintInputScreen) Layout(ctx *Context, ops op.Ctx, dims image.Point) {
	th := &descriptorTheme
	op.ColorOp(ops, th.Background)
	
	r := layout.Rectangle{Max: dims}
	
	// Title
	layoutTitle(ctx, ops, dims.X, th.Text, "Master Fingerprint")
	
	// Instructions
	btnw := assets.NavBtnPrimary.Bounds().Dx()
	body := r.Shrink(leadingSize, btnw, 0, btnw)
	
	var bodytxt richText
	bodyst := ctx.Styles.body
	subst := ctx.Styles.subtitle
	
	bodytxt.Add(ops, subst, body.Dx(), th.Text, "Enter 8-character fingerprint")
	bodytxt.Add(ops, bodyst, body.Dx(), th.Text, "Use ↑↓ to change character")
	bodytxt.Add(ops, bodyst, body.Dx(), th.Text, "Use ←→ to move cursor")
	bodytxt.Y += infoSpacing * 2
	
	// Draw fingerprint input with cursor
	fpText := string(s.fingerprint[:])
	
	// Create monospaced display with cursor indicator
	var fpDisplay strings.Builder
	for i, char := range fpText {
		if i == s.cursorPos {
			fpDisplay.WriteString("[")
			fpDisplay.WriteRune(char)
			fpDisplay.WriteString("]")
		} else {
			fpDisplay.WriteRune(char)
		}
		if i == 3 {
			fpDisplay.WriteString(" ") // Space in middle for readability
		}
	}
	
	// Use larger font for fingerprint display
	leadst := ctx.Styles.lead
	bodytxt.Add(ops, leadst, body.Dx(), th.Text, fpDisplay.String())
	
	// Position text
	ops.Begin()
	for _, l := range bodytxt.Lines {
		l.W.Add(ops)
	}
	op.Position(ops, ops.End(), body.Min.Add(image.Pt(0, scrollFadeDist)))
	
	// Navigation buttons - NO SKIP BUTTON
	layoutNavigation(ctx, ops, th, dims,
		NavButton{Button: Button1, Style: StyleSecondary, Icon: assets.IconBack},
		NavButton{Button: Button3, Style: StylePrimary, Icon: assets.IconCheckmark},
	)
}
```

### 2. Modify DescriptorScreen to include fingerprint screen

In the `DescriptorScreen` struct, add a new field:

```go
type DescriptorScreen struct {
	Descriptor urtypes.OutputDescriptor
	Mnemonic   bip39.Mnemonic
	addresses  *AddressesScreen
	confirm    *ConfirmWarningScreen
	warning    *ErrorScreen
	fingerprint *FingerprintInputScreen  // ADD THIS LINE
}
```

### 3. Modify DescriptorScreen.Layout to always show fingerprint screen

In the `DescriptorScreen.Layout` method, modify to always require fingerprint input before completing:

```go
func (s *DescriptorScreen) Layout(ctx *Context, ops op.Ctx, dims image.Point) (int, Result, string) {
	th := &descriptorTheme
	for {
		switch {
		case s.addresses != nil:
			done := s.addresses.Layout(ctx, ops.Begin(), dims)
			dialog := ops.End()
			if !done {
				dialog.Add(ops)
				return 0, ResultNone, ""
			}
			s.addresses = nil
			continue
		case s.fingerprint != nil:
			fp, result := s.fingerprint.Update(ctx)
			s.fingerprint.Layout(ctx, ops.Begin(), dims)
			dialog := ops.End()
			dialog.Add(ops)
			switch result {
			case ResultComplete:
				s.fingerprint = nil
				keyIdx := 0 // Always use first key
				return keyIdx, ResultComplete, fp // Return manual fingerprint
			case ResultCancelled:
				s.fingerprint = nil
				continue
			}
			return 0, ResultNone, ""
		case s.confirm != nil:
			result := s.confirm.Update(ctx)
			switch result {
			case ConfirmYes:
				s.confirm = nil
				// After confirmation, ALWAYS show fingerprint screen
				s.fingerprint = NewFingerprintInputScreen()
				continue
			case ConfirmNo:
				s.confirm = nil
				return 0, ResultCancelled, ""
			}
		case s.warning != nil:
			dismissed := s.warning.Update(ctx)
			if dismissed {
				s.warning = nil
				continue
			}
		}
		e, ok := ctx.Next(Button1, Button2, Button3)
		if !ok {
			break
		}
		switch e.Button {
		case Button1:
			if e.Click {
				return 0, ResultCancelled, ""
			}
		case Button2:
			if !e.Click {
				break
			}
			s.addresses = NewAddressesScreen(s.Descriptor)
		case Button3:
			if !e.Click {
				break
			}
			if err := validateDescriptor(s.Descriptor); err != nil {
				s.warning = NewErrorScreen(err)
				continue
			}
			keyIdx, ok := descriptorKeyIdx(s.Descriptor, s.Mnemonic, "")
			if !ok {
				if len(s.Descriptor.Keys) == 1 {
					s.confirm = &ConfirmWarningScreen{
						Title: "Unknown Wallet",
						Body:  "The wallet does not match the seed.\n\nIf it is passphrase protected, long press to confirm.",
						Icon:  assets.IconCheckmark,
					}
				} else {
					s.warning = &ErrorScreen{
						Title: "Unknown Wallet",
						Body:  "The wallet does not match the seed or is passphrase protected.",
					}
				}
				continue
			}
			// ALWAYS show fingerprint input screen (never auto-derive)
			s.fingerprint = NewFingerprintInputScreen()
		}
	}
	
	// ... rest of layout code stays the same ...
	
	return 0, ResultNone, ""
}
```

### 4. Update callers to handle the fingerprint return value

Wherever `DescriptorScreen.Layout` is called, update to capture the manual fingerprint:

```go
// Example - find where DescriptorScreen is used and update:
keyIdx, result, fingerprint := descriptorScreen.Layout(ctx, ops, dims)

// fingerprint will ALWAYS contain the manually entered value
// Never empty since fingerprint input is now required
```

### 5. Modify backup.go to use manual fingerprint and fixed metadata

In `backup.go`, modify the `frontSideSeed` function to:
1. Use the manual fingerprint (passed in)
2. Set page number to always "1/1"
3. Set version to always "SC"

**Find this section (around line 280):**
```go
// Engrave version, mfp and page.
const version = "V1"
margin := scale(outerMargin)
innerMargin := scale(innerMargin)
metaMargin := scale(4)
page := fmt.Sprintf("%d/%d", plate.KeyIdx+1, plate.Keys)
mfp := strings.ToUpper(fmt.Sprintf("%.8x", plate.MasterFingerprint))
```

**Replace with:**
```go
// Engrave version, mfp and page - ALL FIXED VALUES
const version = "SC"  // Changed from "V1" to "SC"
margin := scale(outerMargin)
innerMargin := scale(innerMargin)
metaMargin := scale(4)
page := "1/1"  // Always "1/1" instead of dynamic
mfp := strings.ToUpper(fmt.Sprintf("%.8x", plate.MasterFingerprint))
// Note: plate.MasterFingerprint now contains the MANUAL fingerprint
// passed from the GUI, not auto-derived
```

### 6. Ensure manual fingerprint flows through to backup

When creating the `Seed` struct for engraving, make sure to use the manual fingerprint:

```go
// Parse manual fingerprint from GUI
var mfp uint32
if fingerprint != "" {
	fpBytes, err := hex.DecodeString(fingerprint)
	if err == nil && len(fpBytes) == 4 {
		mfp = uint32(fpBytes[0])<<24 | uint32(fpBytes[1])<<16 | 
		      uint32(fpBytes[2])<<8 | uint32(fpBytes[3])
	}
}

// Create seed plate with manual fingerprint
seedPlate := backup.Seed{
	Title:             desc.Title,
	KeyIdx:            keyIdx,
	Mnemonic:          mnemonic,
	Keys:              len(desc.Keys),
	MasterFingerprint: mfp,  // Use manual fingerprint
	Font:              constant.Font,
	Size:              plateSize,
}
```

## Testing Steps

1. Build and flash modified firmware
2. Scan a seed QR code
3. After confirming wallet, fingerprint input screen should appear
4. Test character input:
   - Up/Down buttons cycle through 0-9, a-f
   - Left/Right buttons move cursor
5. Test "Skip" (Button2) to use auto-derived fingerprint
6. Test "Confirm" (Button3) with manual fingerprint
7. Verify engraved plate shows the entered fingerprint

## UI Flow

```
Scan Seed QR
    ↓
Confirm Wallet
    ↓
Enter Fingerprint (REQUIRED - cannot skip)
    ├─ Manual Entry (8 hex chars)
    └─ No auto-derive option
    ↓
Engrave with fixed metadata:
    - Version: "SC"
    - Page: "1/1"
    - MFP: [manual input]
```

## Key Behavior Changes

1. **Fingerprint is ALWAYS manual** - No auto-derivation, no skip option
2. **Version is ALWAYS "SC"** - Not "V1", not dynamic
3. **Page is ALWAYS "1/1"** - Not "1/2", "2/3", etc.
4. **No validation** - Fingerprint doesn't need to match seed

## Engraved Metadata Fields

Every plate will show:
- **Top:** Version "SC"
- **Middle Top:** Page "1/1"
- **Middle Center:** Manual fingerprint (e.g., "ABCD1234")
- **Body Left Column:** Seed words 1-12 (or 1-16 for 24-word seeds)
- **Body Center:** QR code of complete seed
- **Body Right Column:** Words 17-24 (if 24-word seed)
- **Bottom:** Title (if provided)

**Full engraving layout restored** - not the 6-word modified version.

## Notes

### Fingerprint Input:
- Format: 8 hexadecimal characters (4 bytes)
- Display format: `abcd 5678` (space in middle for readability)
- Current character shown in brackets: `[a]bcd 5678`
- Valid characters: 0-9, a-f
- **Always required** - cannot proceed without entering fingerprint
- **No validation** - fingerprint doesn't need to match seed

### Fixed Metadata:
- **Version:** Always "SC" (hardcoded in backup.go)
- **Page:** Always "1/1" (hardcoded in backup.go)
- These are constants and never change regardless of wallet configuration

### Breaking Changes from Original:
- Removes auto-fingerprint derivation completely
- Removes skip/auto option from fingerprint screen
- Changes version identifier from "V1" to "SC"
- Removes dynamic page numbering (always 1/1)
- Fingerprint validation against seed is bypassed
