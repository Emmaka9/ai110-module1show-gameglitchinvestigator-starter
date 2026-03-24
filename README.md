# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable.

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the broken app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.** - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

### Game Purpose
This is a number guessing game built with Streamlit. The player picks a difficulty (Easy, Normal, or Hard), which determines the range of the secret number and how many attempts they get. On each turn the player types a guess and receives a "Too High" or "Too Low" hint until they either guess correctly or run out of attempts. A running score rewards faster wins.

### Bugs Found

| # | Bug | Where |
|---|-----|-------|
| 1 | **State bug** – the text input used `key=f"guess_input_{difficulty}"`, so switching difficulty (or any rerun) generated a new widget key and cleared the stored value, which also caused the session-state initialization block to re-run and pick a new secret number on every submit. | `app.py` line 123 |
| 2 | **Type-comparison bug** – on every even attempt the secret was silently cast to a string (`secret = str(st.session_state.secret)`). String comparison (`"7" > "60"`) gives lexicographic, not numeric, results, making it nearly impossible to win on those turns. | `app.py` lines 158–161 |
| 3 | **Inverted hints** – `check_guess` returned `"📈 Go HIGHER!"` when `guess > secret` and `"📉 Go LOWER!"` when `guess < secret` — exactly backwards. | `app.py` lines 37–40 |
| 4 | **Refactoring gap** – `logic_utils.py` contained only `NotImplementedError` stubs; the tests imported from it, so `pytest` failed immediately. | `logic_utils.py` |

### Fixes Applied

1. **State bug** – Changed the input key from `f"guess_input_{difficulty}"` to the static string `"guess_input"`. Streamlit only initializes `session_state` when a key is absent; a stable key means the secret is generated once and kept for the whole game.
2. **Type-comparison bug** – Removed the `if attempts % 2 == 0` branch entirely. `check_guess` now always receives two integers and uses plain `>` / `<` comparison.
3. **Inverted hints** – Fixed the logic so `guess > secret` → `"Too High"` / `"📉 Go LOWER!"` and `guess < secret` → `"Too Low"` / `"📈 Go HIGHER!"`. Also moved display messages out of `check_guess` (which now returns only the outcome string) into a `hint_messages` dict in `app.py`.
4. **Refactoring** – Moved all four functions (`get_range_for_difficulty`, `parse_guess`, `check_guess`, `update_score`) into `logic_utils.py` and updated `app.py` to import them from there. All three pytest tests now pass.

## 📸 Demo
<img width="1902" height="827" alt="image" src="https://github.com/user-attachments/assets/1a13cd68-ad52-4770-957a-004f403552e1" />

![Winning game screenshot](screenshot.png)

## 🚀 Stretch Features

- [ ] [If you choose to complete Challenge 4, insert a screenshot of your Enhanced Game UI here]
