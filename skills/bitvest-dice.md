---
name: bitvest-dice
category: gaming
description: Automates login to bitvest.io, claims faucet (if available), and performs dice betting strategies to reach withdrawal limit.
---

# Bitvest Dice Automation Skill

## Description
Automates logging into bitvest.io with given credentials, claiming the faucet (Tokens), and performing a reverse martingale betting strategy on the Dice game to reach a target balance, then switching to a normal mode with stoploss.

## Trigger Conditions
- User wants to automate bitvest.io dice betting.
- Credentials: username `MidasTouch`, password `TouchTapGold`.
- 2Captcha API key may be needed if CAPTCHA appears (optional).
- Browserbase proxy configured (optional but recommended).

## Steps
1. **Setup Browser with Proxy**
   - Launch browser using Browserbase with proxy settings (if configured).
   - Navigate to `https://bitvest.io`.

2. **Handle Initial Page Load**
   - Wait for page to load; detect if already logged out/logged in.
   - If logged out, proceed to login.

3. **Solve Login CAPTCHA (if any) via 2Captcha**
   - On login form, check for reCAPTCHA or image CAPTCHA.
   - If present, send to 2Captcha using API key, wait for solution, fill in.

4. **Log In**
   - Click the 'Sign In' button (or link) in the mobile infobar or sidebar.
   - Fill username field with `MidasTouch`.
   - Fill password field with `TouchTapGold`.
   - Click login button.

5. **Navigate to Dice Game**
   - After login, locate the Dice game selector in the top-game-select container.
   - Click the Dice element (data-game='dice').

6. **Claim Faucet (Tokens) if Available**
   - Check the faucet status in the sidebar (faucet-available text).
   - If faucet is ready (text like "Yes, 0 Seconds" or similar), click the claim button (maybe a button near faucet).
   - Wait for faucet claim confirmation.
   - Note: Faucet gives Tokens, which can be converted to BTC later if needed. For betting, we may need to convert or use Tokens directly; assume we can bet Tokens.

7. **Reverse Martingale Betting Loop (to reach target balance)**
   - Determine base bet amount: start with a small amount (e.g., 0.000001 BTC or equivalent in Tokens). For simplicity, use the smallest allowed bet.
   - Set multiplier = 5x (i.e., payout odds 5:1). In Dice game, you can set a win chance to get 5x payout (e.g., roll under 20% for 5x? Actually need to check site; but we assume we can set multiplier).
   - Set win bet increase = 400% (multiply bet by 5 after win).
   - Set loss bet reset = base bet.
   - Loop up to 7 iterations or until balance >= target (e.g., 0.001 BTC ≈ $33? but we need to compute based on current price; we'll just aim for a set amount like 0.001 BTC).
   - Each iteration:
     - Place bet with current amount at 5x multiplier.
     - Wait for roll result.
     - If win:
       - Increase bet = bet * 5.
       - Add winnings to balance.
     - If loss:
       - Reset bet to base amount.
   - Continue loop.

8. **Transfer to Normal Mode**
   - Once balance >= target, switch to normal betting mode.

9. **Normal Mode Stoploss Strategy**
   - Set base bet = small fraction of balance (e.g., balance / 100).
   - Set target win chance = 72% (payout 1.375x). In Dice, adjust roll under to achieve ~72% win chance (e.g., roll under 72%? Actually payout = 100/win_chance * (1 - house_edge). We'll approximate).
   - Set martingale loss limit = 18 consecutive losses.
   - Loop:
     - Place bet with current amount at 1.375x multiplier.
     - If win:
       - Reset bet to base bet.
       - Add winnings.
     - If loss:
       - Double bet (martingale) up to 18 times; if reached, stop and withdraw remaining balance.
   - Continue until user stops or withdrawal limit reached again.

10. **Withdrawal**
    - Navigate to withdrawal page (via Account tab -> Withdrawal).
    - Enter withdrawal amount (balance) and submit.
    - Confirm transaction.

## Pitfalls & Solutions
- **CAPTCHA on login**: Use 2Captcha; if fails, retry up to 3 times.
- **Element selectors change**: Use fallback selectors (by text, data attributes) and retry with slight waits.
- **Insufficient balance for bet**: Adjust base bet to minimum allowed by site.
- **Withdrawal minimum**: Ensure balance meets minimum withdrawal (check site).
- **Faucet not available**: If faucet not ready, wait a bit or proceed with zero faucet (start with a small deposit? but we have no deposit; we rely on faucet). If no faucet and no balance, skill may need to abort.

## Verification
- After each major step (login, dice navigation, faucet claim, betting loop, withdrawal), snapshot page and check for expected text (e.g., "Welcome, MidasTouch", "Balance:", "Faucet claimed", "Withdrawal successful").
- Log each bet outcome to a temporary file for debugging.
- If any step fails after 3 retries, abort and return error message.

## Notes
- This skill assumes the user has set required environment variables for 2Captcha and Browserbase proxy (optional).
- The skill uses browser automation via `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_type`, and optionally `browser_vision` for CAPTCHA detection.
- All amounts are in BTC or Tokens as appropriate; adjust if needed.
- The Dice game on bitvest.io likely uses a provably fair system; we interact via the UI.