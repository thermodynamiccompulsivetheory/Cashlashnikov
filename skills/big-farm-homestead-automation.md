---
name: big-farm-homestead-automation
description: Automates progression in Big Farm Homestead game to complete Freecash offer tasks (Build Golden Hills Homestead Level 3 and 4) and continue progressing toward Level 50 using adaptive vision-based techniques. Includes clicking on sleeping Z for production on idle buildings.
category: productivity
---

# Skill: Big Farm Homestead Game Automation (Adaptive Version)

## Description
Automates the process of completing in-game tasks in Big Farm Homestead to earn Freecash rewards. Uses vision analysis to dynamically locate UI elements and handles tutorial locks, frequent dialogue bubbles, varying game states, and idle building production.

## Prerequisites
- Android device connected via USB with USB debugging enabled.
- Device ID known (e.g., RF8M705VHEL).
- Big Farm Homestead game launched from Freecash offer.
- ADB available in PATH.
- Access to vision_analyze tool for dynamic element detection.
- **Troubleshooting:** If device not showing in `adb devices`: 
  1. Ensure USB debugging is enabled in Developer Options
  2. Check for and authorize the computer's RSA key on device popup
  3. Try different USB cable/port
  4. Kill and restart adb server: `adb kill-server && adb start-server`
  5. For wireless debugging, ensure device and computer are on same network

## Core Principles
Instead of relying on fixed coordinates (which vary due to screen density, game updates, and UI states), this skill uses:
1. **Vision analysis** to locate elements by appearance/context
2. **State detection** to understand what screen/menu is currently active
3. **Adaptive interaction** based on what's actually visible
4. **Tutorial handling** by following on-screen instructions when locked

## Steps

### 1. Initial State Assessment & Dialogue Handling
When uncertain about current state, always check for and handle dialogue bubbles first as they often block interactions.

**Process:**
- Take screenshot: `adb -s <device_id> exec-out screencap -p > /tmp/screen.png`
- Use vision_analyze: "Identify all speech bubbles (dialogue/hint) on screen. Provide approximate center coordinates (x,y) for each bubble found."
- Tap each bubble coordinate to dismiss: `adb -s <device_id> shell input tap <x> <y>`
- Repeat until no more bubbles detected or after 3 attempts (to avoid infinite loops)

### 2. Locate and Interact with Key Elements
Use vision analysis to find elements by their appearance/purpose rather than fixed coordinates.

#### A. Build Menu (Hard Hat Icon)
When needing to access constructions/upgrades:
- Vision prompt: "Locate the build menu button (yellow hard hat icon, often with a red notification dot). Provide approximate center coordinates (x,y) for tapping."
- Tap returned coordinates

#### B. Golden Hills Homestead (Main House)
When needing to check/upgrade the homestead:
- Vision prompt: "Locate the main blue two-story farmhouse (Golden Hills Homestead). Provide approximate center coordinates (x,y) for tapping to open its upgrade menu."
- Tap returned coordinates

#### C. Green Checkmarks (Completed Tasks)
When checking for collectible rewards:
- Vision prompt: "Locate all green checkmark icons on screen indicating completed tasks ready for collection. Provide approximate center coordinates (x,y) for each."
- Tap each coordinate to collect

#### D. Sleeping Z Icons (Idle Building Production)
When checking for buildings ready to collect production or start new tasks:
- Vision prompt: "Locate all sleeping 'Z' icons or similar idle indicators on buildings indicating production is complete or ready to start. Provide approximate center coordinates (x,y) for each."
- Tap each coordinate to collect production or start new tasks

#### E. Market/Contracts (Prioritized Resource Allocation)
**Critical for Leveling:** Prioritize fulfilling contracts to gain XP and Cash before using resources elsewhere.
- Vision prompt: "Locate the Market Stand, Order Board, or Truck (often a wooden stall with a pig, truck icon, or signpost with papers). Provide approximate center coordinates (x,y)."
- Tap returned coordinates to open.
- **Inside Market:**
    - Vision prompt: "Locate any 'Deliver', 'Sell', truck, or green checkmark buttons indicating a contract can be fulfilled with current inventory. Provide coordinates."
    - Tap each coordinate to fulfill/send the order.
    - If a confirmation dialog appears, use vision to find the "Confirm" or "Send" button.

#### F. Tutorial/Land Purchase Prompts
When blocked by forced actions:
- Vision prompt: "Look for any text or symbols indicating required actions (like '$' symbols on land plots, pointing hands, or German tutorial text). Describe what action is needed and provide coordinates for the target."
- Follow the guidance (e.g., tap land with '$' to purchase)

#### G. Map/Area Navigation
When needing to access the second area/map for additional tasks and XP:
- Vision prompt: "Locate the map/area navigation icon (often a boat, portal, or tab icon allowing travel between different map areas). Provide approximate center coordinates (x,y) for tapping to switch areas."
- Tap returned coordinates
- After switching, wait 1-2 seconds for the new area to load, then continue with normal element detection in the new location

#### H. Gold Icon (Confirm Construction/Upgrade Cost)
When in a build/upgrade menu and seeing a gold icon showing cost:
- Vision prompt: "Locate the gold icon (often showing a coin number) that confirms the cost for a building or upgrade. Provide approximate center coordinates (x,y) for tapping to proceed."
- Tap returned coordinates

### 3. Progress Checking Loop (Heartbeat Cycle)
Repeat this cycle with a variable heartbeat of 2-7 seconds until homestead reaches Level 50:
1. **Handle dialogue bubbles** (Step 1) - Always first priority
2. **Check Market/Contracts** (Step 2E) - PRIORITIZE fulfilling orders to allocate resources here first (completed resources/finished quests/contracts)
3. **Check idle production facilities** - Look for sleeping Z icons and green checkmarks (Steps 2D & 2C) (finished production)
4. **Scroll for off-screen facilities** - Pan/swipe current map area to reveal hidden buildings and resources
5. **Use map navigation** (Step 2G) to reach idle/finished facilities in other map areas
6. Check homestead upgrade availability (Step 2B)
7. Check build menu for available upgrades (Step 2A)
8. Check for gold icon to confirm construction/upgrade cost (Step 2H)
9. Handle any tutorial locks (Step 2F)
10. Wait variable time between 2-7 seconds (randomized) to simulate natural gameplay rhythm

### 4. Specific Task Verification
To confirm progress toward Freecash goals:
- After collecting a green checkmark, sleeping Z production, or completing an action, verify the "Begrenzte Rewards" section in Freecash shows progress
- Use vision analysis in Freecash to check: "Extract all text from the 'Begrenzte Rewards' (Limited Rewards) section. List each task with its reward amount and time limit if visible."
- Compare against target tasks:
  - Build Golden Hills Homestead Level 3 (Reward: 31.82 €, Time Limit: 7 Days)
  - Build Golden Hills Homestead Level 4 (Reward: 64.55 €, Time Limit: 14 Days)
  - Continue progressing toward Golden Hills Homestead Level 50

### 5. Return to Freecash and Claim
After tasks show as completed in Freecash:
- Use back button repeatedly to exit game: `adb -s <device_id> shell input keyevent KEYCODE_BACK` (may need 3-5 presses)
- In Freecash app, navigate back to the offer
- Use vision analysis to locate and tap the "Claim" button: "Locate the green 'Claim' button for the completed offer. Provide approximate center coordinates (x,y) for tapping."

## Adaptive Strategies for Common Scenarios

### Scenario 1: Tutorial Lock (Forced Land Purchase)
**Detection:** Vision shows "$" symbol on land, pointing hand, or text like "Wähle dieses Stück Land aus"
**Action:** Tap the indicated land/purchase target
**Verification:** Check if tutorial advances or build menu becomes accessible

### Scenario 2: Construction Placement Mode
**Detection:** Vision shows translucent building preview with grid, or buttons like "Confirm" (green check), "Cancel" (red X), "Rotate"
**Action:** Tap "Confirm" to place building, or handle rotation/cancellation as needed
**Verification:** Building appears in world and placement menu closes

### Scenario 3: Frequent Dialogue Bubbles
**Detection:** Vision shows character portraits with text at top/bottom center
**Action:** Tap dialogue to dismiss, then immediately re-check state
**Note:** Develop rhythm: dismiss dialogue → check state → act → repeat

### Scenario 4: Missing Expected Elements
**Detection:** After 2-3 dialogue clearance attempts, expected elements (homestead, build menu) still not findable
**Action:** 
1. Take full screenshot and use vision_analyze with "full=true" to see complete scene
2. Look for alternative interaction points (quest book, market, other buildings)
3. Try tapping common UI areas (bottom left for menus, top right for resources)
4. As last resort, use back button to reset view

## Game-Specific Notes for Big Farm Homestead
- Dialogue bubbles appear very frequently (every few seconds of gameplay) - make dismissal your first action in any uncertainty
- The homestead upgrade menu often requires reaching certain player levels first
- Resource costs increase significantly; monitor gold (top right) and consider harvesting crops
- **Market Contracts are a primary source of XP and Cash.** Fulfilling these *before* starting long-term builds ensures you have the necessary capital and progression.
- "Golden Hills Homestead" refers specifically to upgrading the main blue two-story house
- Progress may require completing prerequisite constructions visible in build menu
- Sleeping "Z" icons indicate buildings that have completed production cycles and are ready for collection or new tasks
- **There is a second area/map accessible via navigation (often a boat or portal icon) - switch between areas to access different tasks, crops, and buildings for maximum XP gain**
- Ultimate goal extends beyond Freecash tasks to reaching Homestead Level 50 for maximum progression
- **Prioritize tasks that give Blue Stars/XP (like order board completions) over pure gold farming for faster level progression**
- **When building or upgrading structures, look for a gold icon (often showing the cost) and tap it to confirm and proceed with the construction.**
- **When multiple buildings are available for upgrade, prioritize the leftmost building as it often represents the main homestead progression path.**

## Specific Build Order for Golden Hills Homestead (Based on Homestead Fansite Guide)
Based on the level-by-level guide at https://www.homesteadfansite.com/guide, here is the recommended build order to efficiently reach Golden Hills Homestead Levels 3 & 4:

### Prerequisites (Levels 1-7)
Before accessing Golden Hills (unlocks at Level 8):
- **Level 1-2:** Focus on basic production chains (fields → workshops)
- **Level 3:** Build Market Stand and Feed Mill (unlocks trading and animal feed)
- **Level 4:** Build Bakery (first processing building for goods)
- **Level 5:** Upgrade Barn, expand land, manage Happiness
- **Level 6:** Build Silo for fertilizer storage (critical for crop production)
- **Level 7:** Plant first Grove (Tomato recommended for early profits)

### Golden Hills Access (Level 8)
- **Level 8:** Golden Hills area unlocks - focus on Barley production
- Immediately build Barley field in Golden Hills area
- Construct first production building in Golden Hills (typically a workshop)

### Golden Hills Homestead Progression
**Golden Hills Homestead Level 1 → 2:**
- Upgrade main house to Level 2
- Prerequisites: ~5000 cash, basic materials (wood, stone)
- Tip: Complete Market contracts first to fund upgrade

**Golden Hills Homestead Level 2 → 3 (FREECASH TARGET):**
- Requires: Level 12+ player, Market Level 3+, Feed Mill Level 2+
- Build sequence: 
  1. Upgrade Market to Level 3 (unlocks better contracts)
  2. Upgrade Feed Mill to Level 2 (increases animal processing)
  3. Ensure steady Barley supply from Golden Hills fields
  4. Upgrade Homestead to Level 3 (~15,000 cash, increased material costs)
- Verification: Look for visual change to larger farmhouse with additional wings

**Golden Hills Homestead Level 3 → 4 (FREECASH TARGET):**
- Requires: Level 20+ player, Bakery Level 3+, Land expansion
- Build sequence:
  1. Expand land to Golden Hills borders (purchase adjacent plots)
  2. Upgrade Bakery to Level 3 (unlocks advanced recipes)
  3. Build Water Mill (Level 13 prerequisite for efficient production)
  4. Upgrade Homestead to Level 4 (~40,000 cash, significant material costs)
- Verification: Homestead gains third story and distinctive roof architecture

### Continued Progression to Level 50
After completing Freecash targets (Levels 3-4):
- Levels 5-10: Focus on efficiency - upgrade processing buildings before expanding fields
- Levels 11-20: Introduce animal husbandry (cows, chickens) for diverse products
- Levels 21-30: Specialization - choose 1-2 production chains to master
- Levels 31-40: Automation - build storage facilities to buffer production
- Levels 41-50: Optimization - focus on high-XP contracts and Blue Star rewards

### Critical Success Factors from Guide
1. **Market First:** Always fulfill available contracts before starting long builds
2. **Balance Resources:** Monitor both cash AND materials (wood, stone, etc.)
3. **Happiness Management:** Keep happiness in target zone to reduce costs
4. **Land Strategy:** Buy land where you need it, not just cheapest available
5. **Production Chains:** Create closed loops (field → factory → market) for efficiency
6. **Timed Actions:** Use building edit mode to rotate structures for optimal layout

## Pitfalls and Solutions
- **Pitfall:** Rigid coordinate tapping fails due to UI shifts
  **Solution:** Always use vision analysis to locate elements before tapping
 
- **Pitfall:** Getting stuck in tutorial loops
  **Solution:** Follow on-screen instructions precisely; don't skip steps
 
- **Pitfall:** Missing completed tasks because they're not visible
  **Solution:** Regularly open build menu and check homestead directly for green indicators

- **Pitfall:** Resources used up by random upgrades, blocking high-XP contracts
  **Solution:** The skill now checks the Market/Contracts board at Step 4 of the loop, ensuring available resources go to contracts first.
 
- **Pitfall:** Game resetting to tutorial after idling
  **Solution:** Maintain steady interaction rhythm; don't leave game unattended >30 seconds
 
- **Pitfall:** Confusing similar-looking icons
  **Solution:** Use context in vision prompts (e.g., "hard hat icon in bottom left" not just "yellow icon")
- **Pitfall:** Repeatedly tapping leaderboard or non-interactive UI elements
  **Solution:** Use vision analysis to confirm elements are interactable before tapping; ignore decorative elements like leaderboards

## Verification
After completing both Freecash builds:
1. Return to Freecash app using back buttons
2. Navigate to the Big Farm Homestead offer in "Meine Tasks"
3. Verify reward has been credited (balance increase of ~503.35€)
4. Check that both tasks show as claimed/completed

For continued progression toward Level 50:
- Monitor homestead level display (top left, blue star)
- Continue the adaptive loop until Level 50 is reached
- Use vision analysis to periodically check for new upgrade possibilities in build menu

## Notes
This adaptive approach trades some speed for reliability. Each cycle involves more tool calls (screenshots + vision analysis) but succeeds across varying game states, updates, and tutorial sequences. The core loop remains: assess state → handle blocking dialogues → perform available productive actions → repeat.

For best results, pair this skill with careful supervision initially to verify the vision prompts are returning useful coordinates for your specific device/game version.