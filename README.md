# Freecash Skill Family

This repository contains a collection of skills for automating various Freecash offers and related tasks.

## Skills Included

1. **Big Farm Homestead Automation** (`big-farm-homestead-automation`)
   - Automates progression in Big Farm Homestead game to complete Freecash offer tasks
   - Builds Golden Hills Homestead Level 3 & 4, continues to Level 50
   - Uses vision-based adaptive techniques for reliable automation

2. **Complete Freecash Offer Template** (`complete-freecash-offer`)
   - Template for launching Freecash offers and completing required in-game tasks
   - Generic framework adaptable to different offer types

3. **Disney Solitaire Automation** (`disney-solitaire`)
   - Automates playing Disney Solitaire (TriPeaks variant) for Freecash rewards
   - Includes card gameplay mechanics (±1 from current card) and reward collection
   - Handles popups/ads and locates gold reward button in lower area

4. **Dice Faucet Automation** (`dice-faucet`)
   - Automates sicodice.com faucet claiming and betting strategies
   - Uses Reverse Martingale to reach $33 withdrawal, then Normal Mode with stoploss
   - Integrates with 2Captcha and Browserbase for robust automation

5. **Bitvest Dice Automation** (`bitvest-dice`)
   - Automates login to bitvest.io, faucet claiming, and dice betting strategies
   - Uses credentials: MidasTouch / TouchTapGold
   - Similar Reverse Martingale → Normal Mode progression with stoploss

## Usage

Each skill is stored as a markdown file in the `skills/` directory. To use these skills with Hermes Agent:

1. Copy the desired skill file to your Hermes skills directory:
   ```bash
   cp skills/big-farm-homestead-automation.md ~/.hermes/skills/productivity/
   ```

2. Reload skills in Hermes:
   ```
   /skills reload
   ```

3. Use the skill in your automation workflows.

## Requirements

- Android device with USB debugging enabled
- ADB available in PATH
- Hermes Agent with vision_analyze tool access
- For dice skills: 2Captcha API key and Browserbase proxy configuration
- Specific app installations (Big Farm Homestead, Disney Solitaire, etc.)

## Notes

These skills use vision analysis instead of fixed coordinates for reliable automation across different devices, screen sizes, and game versions. They adapt to changing UI elements by analyzing screenshots and locating elements by appearance/context.

Each skill includes detailed instructions, prerequisites, core principles, step-by-step procedures, adaptive strategies for common scenarios, pitfalls and solutions, and verification steps.
