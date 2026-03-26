---
name: dice-faucet
category: gaming
description: Automates sicodice.com faucet claiming and betting strategies.
---

# Dice-Faucet Skill

## Description
Automates logging into sicodice.com, claiming the faucet, and performing a reverse martingale betting strategy to reach the $33 minimum withdrawal limit, then switching to normal mode with a stoploss strategy.

## Trigger Conditions
- User wants to automate sicodice.com faucet claiming and betting.
- Credentials for sicodice.com are available in environment variables `SICODICE_USER` and `SICODICE_PASS`.
- 2Captcha API key available in `2CAPTCHA_KEY`.
- Browserbase proxy configured via environment variables `BROWSERBASE_PROXY_HOST`, `BROWSERBASE_PROXY_PORT`, `BROWSERBASE_PROXY_USER`, `BROWSERBASE_PROXY_PASS`.

## Steps
1. **Setup Browser with Proxy**
   - Launch browser using Browserbase with proxy settings.
   - Navigate to `https://sicodice.com`.

2. **Solve Login CAPTCHA (if any) via 2Captcha**
   - Detect if a CAPTCHA appears on login form.
   - If present, send image to 2Captcha using API key, wait for solution, fill in.

3. **Log In**
   - Fill username field with `SICODICE_USER`.
   - Fill password field with `SICODICE_PASS`.
   - Click login button.

4. **Navigate to Faucet**
   - After login, locate and click the Faucet button/link.
   - Claim faucet reward (usually a small amount of cryptocurrency).

5. **Reverse Martingale Betting Loop (to reach $33)**
   - Set initial bet amount = faucet reward (or a small base like $0.01).
   - Set multiplier = 5x (i.e., payout odds 5:1).
   - Set win bet increase = 400% (i.e., after a win, multiply bet by 5).
   - Set loss bet reset = base bet (reverse martingale: after loss, reset to base).
   - Loop up to 7 iterations or until balance >= $33:
     - Place bet with current amount at 5x multiplier.
     - Wait for roll result.
     - If win:
       - Increase bet = bet * 5 (400% increase).
       - Add winnings to balance.
     - If loss:
       - Reset bet to base amount.
     - Continue loop.

6. **Transfer to Normal Mode**
   - Once balance >= $33, withdraw to internal wallet (if needed) or keep balance.
   - Switch to normal betting mode.

7. **Normal Mode Stoploss Strategy**
   - Set base bet = small fraction of balance (e.g., balance / 100).
   - Set target win chance = 72% (payout 1.375x).
   - Set martingale loss limit = 18 consecutive losses.
   - Loop:
     - Place bet with current amount at 1.375x multiplier.
     - If win:
       - Reset bet to base bet.
       - Add winnings.
     - If loss:
       - Double bet (martingale) up to 18 times; if reached, stop and withdraw remaining balance.
   - Continue until user stops or withdrawal limit reached again.

8. **Withdrawal**
   - Navigate to withdrawal page.
   - Enter withdrawal amount (balance) and submit.
   - Confirm transaction.

## Pitfalls & Solutions
- **CAPTCHA on login**: Use 2Captcha; if fails, retry up to 3 times.
- **Proxy authentication errors**: Ensure Browserbase proxy credentials are correct; fallback to direct connection if proxy fails.
- **Element selectors change**: Use fallback selectors (by text, aria-label) and retry with slight waits.
- **Insufficient balance for bet**: Adjust base bet to minimum allowed by site.
- **Withdrawal minimum**: Ensure balance >= $33 before attempting withdrawal; otherwise continue betting.

## Verification
- After each major step (login, faucet claim, betting loop, withdrawal), snapshot page and check for expected text (e.g., "Welcome, username", "Faucet claimed", "Balance: $XX", "Withdrawal successful").
- Log each bet outcome to a temporary file for debugging.
- If any step fails after 3 retries, abort and return error message.

## Notes
- This skill assumes the user has set required environment variables.
- The skill uses browser automation via `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_type`, and optionally `browser_vision` for CAPTCHA detection.
- 2Captcha integration can be done via HTTP API calls using `terminal` with `curl`.
- All amounts are in USD equivalent; adjust if site uses different currency.