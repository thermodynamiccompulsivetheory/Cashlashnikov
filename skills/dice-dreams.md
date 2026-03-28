---
name: dice-dreams
description: "Automates Dice Dreams game: logs in as guest, dismisses popups, starts rolling dice via tap at (550,1990), and when out of dice, starts improving buildings."
category: gaming
---

# Skill: Dice Dreams Automation

## Description
Automates gameplay in Dice Dreams: launches the game, logs in as guest, dismisses pop-up ads and dialogs, taps the roll dice button at coordinates (550,1990) to roll, and when out of dice, switches to building improvement mode to progress.

## Prerequisites
- Android device connected via USB with USB debugging enabled.
- Device ID known (e.g., RF8M705VHEL or cb39e2e9).
- Dice Dreams game installed and accessible from app launcher.
- ADB available in PATH.
- Access to vision_analyze tool for dynamic element detection.
- Game updated to a version where the roll button is reliably at approximately (550,1990) in portrait mode (adjust if needed).

## Core Principles
Instead of relying solely on fixed coordinates, this skill uses:
1. **Vision analysis** to detect and dismiss pop-up ads (look for red X icons, close buttons).
2. **State detection** to understand if we are on the home screen, in a popup, or out of dice.
3. **Adaptive interaction** based on what's actually visible on screen.
4. **Fallback to building improvement** when dice are depleted.

## Steps

### 1. Ensure Game is Launched and on Home Screen
If uncertain about current state, launch Dice Dreams via launcher and wait for home screen.

**Process:**
- Launch game: `adb -s <device_id> shell monkey -p com.joydice.dreams -c android.intent.category.LAUNCHER 1`
- Wait 2-3 seconds for loading.
- Use vision analysis to confirm we are on the Dice Dreams home screen:
  - Prompt: "Is this the Dice Dreams home screen showing the main board, roll button, and guest login prompt? Describe what you see."
- If not on home screen, press back button repeatedly (`adb -s <device_id> shell input keyevent KEYCODE_BACK`) up to 3 times to exit any menus, then re-launch if needed.

### 2. Dismiss Pop-up Ads and Dialogs
Pop-ups frequently appear after launching or during gameplay. Always clear them before attempting to roll.

**Process:**
- Take screenshot: `adb -s <device_id> exec-out screencap -p > /tmp/popup_check.png`
- Use vision_analyze: "Look for any advertisement close buttons, 'X' icons, skip buttons, or modal dialogs that overlay the game screen. If present, describe their appearance and provide approximate center coordinates (x,y) for tapping to dismiss them. If none present, state that no dismissible ads are visible."
- If coordinates are returned, tap each: `adb -s <device_id> shell input tap <x> <y>`
- Repeat until no more dismissible ads are detected or after 3 attempts (to avoid infinite loops).

### 3. Locate and Tap Roll Button
The primary action is rolling dice.

**Process:**
- After dismissing pop-ups, use vision analysis to locate the roll button if coordinates may vary:
  - Prompt: "Locate the dice roll button (often a large circular button with dice icons or a 'Roll' label). Provide approximate center coordinates (x,y) for tapping."
- If vision returns coordinates, use those; otherwise fall back to the default coordinate (550,1990) for portrait mode.
- Tap the roll button: `adb -s <device_id> shell input tap <x> <y>`
- Wait a short delay (1-2 seconds) for the roll animation to resolve.

### 4. Detect Out of Dice State
After each roll (or periodically), check if we are out of dice.

**Process:**
- Take screenshot after roll attempt.
- Use vision_analyze: "Is there any indication that we are out of dice or cannot roll? Look for text like 'Out of dice', 'No dice', or a disabled/greyed out roll button. Also check if the roll button is unresponsive. Describe what you see."
- If the response indicates out of dice, proceed to Step 5 (Building Improvement).
- If we still have dice, loop back to Step 2 (dismiss any new pop-ups) and then Step 3 (roll again).

### 5. Building Improvement Mode (When Out of Dice)
When dice are depleted, use resources to build/upgrade buildings to progress and earn more dice.

**Process:**
- Dismiss any pop-ups (Step 2).
- Locate the build/menu button:
  - Prompt: "Locate the build menu button (often a hammer, wrench, or building icon). Provide approximate center coordinates (x,y) for tapping to open the build/upgrade menu."
- Tap to open build menu.
- Inside the build menu, look for upgradeable buildings:
  - Prompt: "Locate any upgrade buttons (green arrows, '+', or 'Upgrade' labels) on buildings that can be improved with current resources. Provide approximate center coordinates (x,y) for each."
- Tap each upgradeable building to improve it.
- After upgrading, dismiss any pop-ups that may appear.
- Optionally, check if we have earned more dice (look for dice count increase or a notification).
- If dice are still out, consider repeating building improvements or wait for dice regeneration.
- If we have dice again, return to Step 3 to resume rolling.

### 6. Progress Checking Loop (Heartbeat Cycle)
Repeat the following cycle with a variable heartbeat of 3-8 seconds until desired progress is made:
1. **Handle pop-up ads** (Step 2) - Always first priority
2. **Check dice status** (Step 4) - If we have dice, roll (Step 3); if out of dice, improve buildings (Step 5)
3. Wait variable time between 3-8 seconds (randomized) to simulate natural gameplay rhythm

## Adaptive Strategies for Common Scenarios

### Scenario 1: Guest Login Prompt
**Detection:** Vision shows a button or text prompting to 'Play as Guest', 'Continue as Guest', or similar.
**Action:** Tap the guest login button to start the game without social login.
**Verification:** Home screen with board and roll button appears.

### Scenario 2: Full-Screen Ad Interstitial
**Detection:** Vision shows a full-screen ad, often with a small 'X' in corner or 'Close' button after a delay.
**Action:** Wait for skip button to appear, then tap it; or if immediate close button is visible, tap it.
**Verification:** Returns to game home screen.

### Scenario 3: Roll Button Temporarily Disabled During Animation
**Detection:** After tapping roll, the button may appear greyed out or unresponsive for a second.
**Action:** Do not repeatedly tap; wait for the roll animation to complete before checking state again.
**Solution:** Include a short delay after each tap before proceeding to next iteration.

### Scenario 4: Building Menu Navigation
**Detection:** After tapping build menu, may need to scroll or tab to see all buildings.
**Action:** If vision shows scrollable list, use swipe gestures to reveal more buildings before locating upgrade buttons.
**Alternative:** Use coordinates for known building positions if consistent.

### Scenario 5: Energy/Life System (if applicable)
Some versions have an energy system limiting rolls.
**Detection:** Vision shows a heart or energy counter with a timer.
**Action:** If energy is out, either wait for regeneration or use in-game items to refill; otherwise proceed with rolling if energy available.

## Verification
To confirm skill is working:
- Observe that pop-up ads are being dismissed promptly.
- See the roll button being tapped and dice rolling animation.
- When out of dice, observe transition to building menu and upgrades being applied.
- Check in-game dice count increases after building improvements or over time.

## Notes
- This adaptive approach trades some speed for reliability. Each cycle involves more tool calls (screenshots + vision analysis) but succeeds across varying game states, ad frequencies, and UI updates.
- The coordinate (550,1990) is a starting point for portrait mode on many devices; adjust based on vision analysis if the button is consistently elsewhere.
- For best results, pair this skill with careful supervision initially to verify the vision prompts are returning useful coordinates for your specific device/game version.
- Consider integrating with a cron job for periodic execution if long-term automation is desired.