---
name: complete-freecash-offer
description: Automates the process of launching a Freecash offer, starting the game, and completing the required in-game tasks to earn rewards.
category: productivity
---

# Skill: Complete a Freecash Offer (e.g., Big Farm Homestead)

## Description
Automates the process of launching a Freecash offer, starting the game, and completing the required in-game tasks to earn rewards. This skill assumes the offer is already visible in the Freecash app or can be found via vision analysis.

## Prerequisites
- Android device connected via USB with USB debugging enabled.
- Device ID known (e.g., RF8M705VHEL).
- Freecash app installed (package: com.freecash.app2).
- ADB available in PATH.

## Steps

### 1. Launch Freecash App
```bash
adb -s <device_id> shell monkey -p com.freecash.app2 -c android.intent.category.LAUNCHER 1
```

### 2. Locate the Offer Tile
If the offer is not immediately visible, use vision analysis to find it.
- Take a screenshot: `adb -s <device_id> exec-out screencap -p > /tmp/screen.png`
- Use vision_analyze to find the offer tile coordinates (e.g., "Big Farm Homestead").
- If not found, swipe the carousel or scroll down and repeat.

### 3. Tap the Offer to Open Details
```bash
adb -s <device_id> shell input tap <x> <y>
```
Replace `<x> <y>` with the coordinates from vision analysis.

### 4. Extract Task Details (Optional)
Use vision_analyze to extract the limited rewards/tasks and their rewards/time limits.
Example prompt: "Extract all text from the 'Begrenzte Rewards' (Limited Rewards) section. List each task with its reward amount and time limit if visible."

### 5. Launch the Game
Tap the green "Play" button (coordinates from vision analysis, e.g., ~890, 395 on 1080x2280 screen).
```bash
adb -s <device_id> shell input tap <play_x> <play_y>
```

### 6. Complete In-Game Tasks
**Note:** Automating gameplay inside the target game is game-specific and may require additional scripting or image recognition.
- For Big Farm Homestead, the tasks are:
  - Build Golden Hills Homestead Level 3 (Reward: 31.82 €, Time Limit: 7 Days)
  - Build Golden Hills Homestead Level 4 (Reward: 64.55 €, Time Limit: 14 Days)
- You must manually complete these tasks within the game, or create a separate game-specific automation skill.

### 7. Return to Freecash and Claim Rewards
After completing the in-game tasks, return to Freecash (usually via the game's exit button or home button) and navigate back to the offer to claim the reward.
- Press BACK until you return to Freecash: `adb -s <device_id> shell input keyevent KEYCODE_BACK` (may need multiple times).
- Once back in Freecash, the offer should show a "Claim" button. Tap it to receive the reward.

### 8. Verify Reward Credited
Check the balance in Freecash to ensure the reward has been added.

## Pitfalls
- The Freecash app may open a Chrome Custom Tab for the offerwall; if so, uiautomator may not see the content. Use vision analysis and coordinate tapping.
- Game UI varies widely; automating in-game progress is complex and may violate game terms of service.
- Ensure the device stays awake during long-running tasks: `adb -s <device_id> shell svc power stayon usb`.

## Verification
After completing the steps, the reward amount should be added to your Freecash balance. You can verify by checking the balance displayed in the app.

## Notes
This skill is a template. For each specific offer, you will need to:
- Identify the offer tile coordinates.
- Extract the specific tasks.
- Potentially create a sub-skill for automating the target game if desired.