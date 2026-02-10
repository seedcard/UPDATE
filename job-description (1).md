# Job Description: SeedHammer Engraver GUI Modification

## Project Overview
Modify the open-source SeedHammer engraver software to require manual fingerprint input and update fixed metadata fields on engraved plates.

## Project Details
- **Software:** SeedHammer Engraver (https://github.com/seedcard/engraver)
- **Language:** Go (Golang)
- **Scope:** GUI modifications + backend engraving changes
- **Timeline:** 1-2 weeks estimated
- **Budget:** [Your budget here]

## Required Modifications

### 1. Add Manual Fingerprint Input Screen
- Create a new GUI screen for entering an 8-character hexadecimal fingerprint
- Screen should appear after wallet confirmation, before engraving begins
- User navigates with directional buttons (up/down to change character, left/right to move cursor)
- Fingerprint entry is **required** - no skip or auto-derive option
- No validation needed - fingerprint can be any 8 hex characters (0-9, a-f)

### 2. Update Fixed Metadata Fields
Modify the engraving template to use constant values:
- **Version field:** Change from "V1" to "SC"
- **Page number field:** Change from dynamic (e.g., "1/2", "2/3") to fixed "1/1"
- **Fingerprint field:** Use the manually entered value (not auto-derived from seed)

### 3. Technical Requirements
- Modify GUI to add fingerprint input screen with character selection interface
- Update backup.go to use fixed metadata values
- Ensure manual fingerprint flows through from GUI to engraving output
- Test on actual SeedHammer hardware (or provide simulation instructions)

## Deliverables
1. **Ready-to-flash IMG file** - Complete Raspberry Pi image that can be written directly to SD card
2. Modified source code (GitHub repository or patch files for reference)
3. Build instructions (how the IMG was created, for future modifications)
4. Flashing instructions (simple steps to write IMG to SD card)
5. Testing confirmation that modifications work as specified on actual hardware

## Source Material Provided
- Current working SeedHammer IMG file (4GB) from SD card
- Complete technical specification documents with exact code modifications needed
- Access to original SeedHammer repository (https://github.com/seedcard/engraver)

## Build Requirements
- Developer must be able to build complete Raspberry Pi Zero IMG from modified code
- IMG must be bootable and ready to flash to SD card (no additional setup required)
- All dependencies and configurations must be included in the IMG
- Final IMG should be similar size to original (~4GB)

## Required Skills
- Go (Golang) programming experience
- GUI development (ideally with the framework used by SeedHammer)
- **Raspberry Pi development** - ability to build bootable IMG files
- Experience with Raspberry Pi Zero and embedded Linux
- Ability to read and modify existing codebases
- Git/GitHub familiarity
- Experience with IMG building tools (e.g., Buildroot, Yocto, or custom scripts)

## Working Process
1. You provide current working IMG file (4GB) via cloud storage link
2. Developer extracts and modifies the SeedHammer code per specifications
3. Developer rebuilds complete bootable IMG with modifications
4. You flash IMG to SD card and test on actual SeedHammer hardware
5. Developer makes any needed adjustments based on testing
6. Final IMG delivered

## Documentation Provided
Complete technical specification documents will be provided including:
- Exact code modifications needed for GUI
- Exact code modifications needed for backup/engraving
- Sample code snippets
- Expected behavior and workflow

## How to Apply
Please include:
- Your experience with Go and GUI development
- Estimated timeline for completion
- Your rate or fixed project quote
- GitHub profile or code samples (if available)
- Any questions about the project scope

---

**Note:** This is a straightforward modification to an existing open-source project. All necessary technical details and code examples are documented and ready to share with the selected developer.
