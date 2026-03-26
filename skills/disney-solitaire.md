---
name: disney-solitaire
description: Automates playing Disney Solitaire (TriPeaks style) for Freecash offers, including card gameplay mechanics and reward collection from the gold button in the lower area.
category: productivity
---

# Skill: Disney Solitaire Automation for Freecash

## Description
Automates the process of playing Disney Solitaire (TriPeaks variant) to complete Freecash offer tasks. Uses vision analysis to dynamically locate game elements, handles gameplay mechanics (placing cards +/-1 from current card), manages pop-ups/ads, and collects rewards from the gold button in the lower area upon completion.

## Prerequisites
- Android device connected via USB with USB debugging enabled.
- Device ID known (e.g., RF8M705VHEL).
- Disney Solitaire game launched from Freecash offer.
- ADB available in PATH.
- Access to vision_analyze tool for dynamic element detection.
- **Troubleshooting:** If device not showing in `adb devices`:
  1. Ensure USB debugging is enabled in Developer Options
  2. Check for and authorize the computer's RSA key on device popup
  3. Try different USB cable/port
  4. Kill and restart adb server: `adb kill-server && adb start-server`
  5. For wireless debugging, ensure device and computer are on same network
  6. If connection drops frequently, check USB cable quality and avoid USB hubs
  7. Some devices require toggling USB debugging off/on after reconnecting

## Core Principles
Instead of relying on fixed coordinates (which vary due to screen density, game updates, and UI states), this skill uses:
1. **Vision analysis** to locate elements by appearance/context
2. **State detection** to understand what screen/menu is currently active
3. **Adaptive interaction** based on what's actually visible
4. **Game state tracking** to know current card and available moves

## Steps

### 1. Initial State Assessment & Popup Handling
When uncertain about current state, always check for and handle popups/ads first as they often block interactions.

**Process:**
- Take screenshot: `adb -s <device_id> exec-out screencap -p > /tmp/screen.png`
- Use vision_analyze: "Identify all popup windows, ads, or dialog boxes on screen. Provide approximate center coordinates (x,y) for each close button (X) found."
- Tap each close button coordinate: `adb -s <device_id> shell input tap <x> <y>`
- Repeat until no more popups detected or after 3 attempts (to avoid infinite loops)

**Special Case: System Permission Dialogs**
When automated taps trigger system permission dialogs (e.g., for screen recording):
- Vision prompt: "Locate the 'Zulassen' (Allow) button in the system permission dialog. Provide approximate center coordinates (x,y) for tapping."
- Tap returned coordinates: `adb -s <device_id> shell input tap <x> <y>`
- Wait for dialog to disappear before proceeding

### 2. Locate and Interact with Key Game Elements
Use vision analysis to find elements by their appearance/purpose rather than fixed coordinates.

#### A. Current Card (Top of Discard Pile)
When needing to know what card is currently playable:
- Vision prompt: "Locate the current card at the top of the discard pile (usually face-up card at bottom center or bottom right). Provide approximate center coordinates (x,y) and identify the card's rank and suit if visible."
- Use returned coordinates for reference (don't tap unless gameplay requires)

#### B. Playable Cards (Tableau/Peaks)
When looking for cards to play:
- Vision prompt: "Locate all face-up cards in the tableau/peaks that are visible (not covered). For each card, provide approximate center coordinates (x,y) and identify the card's rank and suit."
- Filter cards that are either rank+1 or rank-1 from current card (Ace can be high or low depending on game rules)

#### C. Draw/Stock Pile
When needing to draw a new current card:
- Vision prompt: "Locate the draw/stock pile (usually face-down cards at bottom left). Provide approximate center coordinates (x,y) for tapping to draw a new card."
- Tap returned coordinates

#### D. Gold Reward Button (Lower Area)
When game appears complete:
- Vision prompt: "Locate the gold button/button in the lower area of the screen that likely collects rewards or claims completion. Provide approximate center coordinates (x,y)."
- Tap returned coordinates to claim reward

#### E. Game Complete Indicators
When checking if solitaire is won:
- Vision prompt: "Look for any text or symbols indicating game completion (like 'Level Complete', 'You Win!', congratulatory animations, or all cards cleared from peaks). Describe what indicates completion and provide coordinates for any interactive elements."

#### F. Undo Button
When needing to revert a mistaken action (like drawing when you shouldn't have):
- Vision prompt: "Locate the undo button (usually a curved arrow pointing left in lower corners or near menu). Provide approximate center coordinates (x,y)."
- Tap returned coordinates to revert last action
- Verify the action was reverted by checking game state
- **Learned Pattern**: For Disney Solitaire on device RF8M705VHEL, the undo button has been found at approximately (1500,900)

### 3. Gameplay Loop
Repeat this cycle until game is complete or no moves remain:

1. **Handle popups/ads** (Step 1) - Always first priority
2. **Check for game completion** (Step 2E) - If complete, go to Step 4
3. **Identify current card** (Step 2A) - Get rank of face-up card at top of discard pile
4. **Find playable cards** (Step 2B) - Locate all face-up cards in peaks that are +/-1 from current card
5. **If playable cards exist:**
   - Select one playable card (prefer uncovering face-down cards when possible)
   - Tap the card coordinates to move it to discard pile
   - Update current card to the moved card's rank
6. **If no playable cards exist:**
   - Tap draw pile (Step 2C) to get new current card
   - If draw pile is empty and no moves, game may be lost
7. **Wait 0.5-1.5 seconds** between actions to simulate natural gameplay
8. **Repeat** from step 1

### 4. Reward Collection
After game completion detected:
1. Handle any final popups/celebrations (Step 1)
2. Locate and tap gold reward button (Step 2D)
3. Wait for reward animation/confirmation
4. Use back button to exit game if needed: `adb -s <device_id> shell input keyevent KEYCODE_BACK`
5. Return to Freecash app and verify reward credited

### 5. Freecash Integration
To complete the offer:
1. Launch Disney Solitaire from Freecash offer wall
2. Play game using Steps 1-3 above
3. Collect reward using Step 4
4. In Freecash app, navigate back to offer
5. Use vision analysis to locate and tap "Claim" button: "Locate the green 'Claim' button for the completed offer. Provide approximate center coordinates (x,y) for tapping."
6. Verify reward has been credited

## Adaptive Strategies for Common Scenarios

### Scenario 1: No Visible Moves but Draw Pile Available
**Detection:** Vision shows face-up cards in peaks but none are +/-1 from current card, and draw pile has cards
**Action:** Tap draw pile to get new current card
**Verification:** New current card allows playable moves

### Scenario 2: Covered Cards Need to be Uncovered
**Detection:** Vision shows face-down cards with face-up cards partially covering them
**Action:** Prioritize playing cards that will uncover face-down cards when possible
**Verification:** More face-up cards become available

### Scenario 3: Winning Condition Detected
**Detection:** Vision shows "Level Complete" text, all peaks cleared, or celebratory animation
**Action:** Proceed directly to reward collection (Step 4)
**Verification: Gold button becomes interactable or reward screen appears

### Scenario 4: Frequent Ads/Popups
**Detection:** Vision shows ad close buttons or popup dialogs every 2-3 actions
**Action:** Handle popups before every gameplay check (integrated into loop)
**Note:** Develop rhythm: check state → handle popups → check for moves → act → repeat

### Scenario 5: Ambiguous Card Values
**Detection:** Vision cannot clearly identify card rank/suit due to image quality or angle
**Action:** 
1. Take multiple screenshots from slightly different angles if possible
2. Use contextual clues (card position, typical solitaire layouts)
3. As last resort, make conservative guess or draw new card
4. Remember: Wrong move is better than no move when stuck

### Scenario 6: Device Connection Issues
**Detection:** `adb devices` shows no device or unauthorized device, or frequent disconnections during automation
**Action:**
  1. Kill and restart adb server: `adb kill-server && adb start-server`
  2. Check USB cable connection and try different ports
  3. On device: Toggle USB debugging off/on in Developer Options
  4. Reauthorize computer if prompted on device screen
  5. Try different USB cable if available
  6. Avoid USB hubs, connect directly to computer
**Verification:** Device appears in `adb devices` as authorized

### Scenario 7: Gameplay Screen Not Launching
**Detection:** After tapping the level/play button, the screen remains on the main menu/lobby instead of transitioning to solitaire gameplay
**Action:**
  1. Wait 2-3 seconds and take a new screenshot to confirm state
  2. Check for and handle any popup ads or dialogs that might be blocking transition
  3. Tap the level button again (sometimes requires multiple attempts)
  4. Look for alternative "Play" or "Start" buttons if the level button isn't working
  5. Use back button to reset and try again if stuck in a loop
**Verification:** Solitaire tableau with cards becomes visible

### Scenario 8: Incorrect Card Identification/Tap
**Detection:** After tapping a card believed to be playable, the card doesn't move or an invalid move error occurs
**Action:**
  1. Take a new screenshot and re-analyze the current card and available moves
  2. Double-check that the tapped card is actually +/-1 from the current card (remember Ace is both high and low)
  3. Verify card coordinates are accurate for the current screen state
  4. Consider that card overlays or partial coverage might affect tap accuracy
**Verification:** Selected card moves to discard pile and current card updates correctly

### Scenario 9: Mistaken Draw or Action
**Detection:** You accidentally drew a card from the deck when you shouldn't have, or performed another incorrect action
**Action:**
  1. Look for an undo button (often in lower corners or near menu)
  2. Use vision analysis to locate undo button if needed: "Locate the undo button (usually a curved arrow pointing left). Provide approximate center coordinates (x,y)."
  3. Tap the undo button coordinates
  4. Verify the action was reverted
**Verification:** Game state returns to before the mistaken action

### Scenario 10: Technical Overlay Interference
**Detection:** Screen shows technical debug information (coordinates, pointers, etc.) at edges that interferes with vision analysis
**Action:**
  1. Crop screenshot to remove known overlay areas if consistent
  2. Focus vision prompts on central game area where cards are located
  3. Wait for overlays to disappear if they're temporary
  4. Increase screenshot quality/resolution if possible

## Game-Specific Notes for Disney Solitaire
- This appears to be a TriPeaks solitaire variant where you clear peaks by selecting cards +/-1 from current card
- Ace typically plays as both high (above King) and low (below 2) unless specified otherwise
- **Gold button in lower area** is the key reward collection point after game completion
- Disney-themed cards may have character images - focus on rank/number in corner for identification
- Game may have multiple levels; completing one level may immediately start another
- Watch for level completion bonuses that might require additional taps
- Some versions have wild cards or special bonuses - adapt vision prompts accordingly

## Pitfalls and Solutions
- **Pitfall:** Rigid coordinate tapping fails due to UI shifts or different screen resolutions
  **Solution:** Always use vision analysis to locate elements before tapping
  
- **Pitfall:** Getting stuck thinking no moves exist when moves are available
  **Solution:** Double-check vision analysis for card ranks; remember Ace can be high/low
  
- **Pitfall:** Missing reward button because it's disguised or animated
  **Solution:** Look for shiny/golden elements, buttons with text like "Collect", "Reward", or triangle/play icons in lower screen area
  
- **Pitfall:** Game logic changes (e.g., different rule variations)
  **Solution:** Start each session by verifying basic mechanics with a few test moves
  
- **Pitfall:** Confusing similar-looking UI elements (ads vs game elements)
  **Solution:** Use context in vision prompts (e.g., "gold button in lower area" not just "golden circle")

## Verification
After completing the game:
1. Return to Freecash app using back buttons
2. Navigate to the Disney Solitaire offer in "Meine Tasks"
3. Verify reward has been credited (check offer status and balance increase)
4. Check that the offer shows as claimed/completed

## Notes
This adaptive approach trades some speed for reliability. Each cycle involves more tool calls (screenshots + vision analysis) but succeeds across varying game states, updates, and popup sequences. The core loop remains: assess state → handle popups → check for moves → make move or draw → repeat.

For best results, pair this skill with careful supervision initially to verify the vision prompts are returning useful coordinates for your specific device/game version. The vision prompts are critical - they may need tuning based on how Disney Solitaire renders on your specific device.

## Working with User-Provided Coordinates
When users provide specific coordinate sequences for known levels or situations (like level 141), the skill can still be valuable:
1. Use vision analysis to first verify the current game state matches expectations
2. Follow the user-provided coordinate sequence for known optimal plays
3. After each coordinate tap, use vision analysis to verify the action had the expected effect
4. If the action didn't work as expected, fall back to the adaptive gameplay loop (Steps 1-3)
5. Always be ready to handle popups/ads between user-provided actions
This hybrid approach combines the efficiency of known sequences with the robustness of vision-based verification.