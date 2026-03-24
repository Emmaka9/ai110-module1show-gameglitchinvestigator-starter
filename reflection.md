# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

When I first ran the app, I could open the "Developer Debug Info" panel, see the secret number, type it exactly, and still lose. The first concrete bug I noticed was that the secret number changed on every Submit click — even when I typed the right answer, by the time I hit the button the secret had already moved to a new value. The second bug was that the hints were completely backwards: if I guessed too high, the game told me to go even higher. There was also a subtler bug where on every even-numbered attempt the secret was silently converted to a string, which broke numeric comparison and made it feel like the game was cheating in a second, different way.

---

## 2. How did you use AI as a teammate?

I used Claude Code (the CLI tool) to help me read through the code, identify the bugs, and plan the fixes. One example of a correct AI suggestion was identifying that the input's `key=f"guess_input_{difficulty}"` was the root cause of the state bug — it explained that Streamlit re-initializes session state whenever a new widget key appears, so changing difficulty (or any rerun) created a fresh key and triggered the `if "secret" not in st.session_state` block again. I verified this by changing the key to a static string and confirming that the secret stayed the same across multiple submits. An example of something I had to push back on was the initial framing — AI wanted to keep the tuple return `("Win", "🎉 Correct!")` from `check_guess` and update the tests to unpack it, but the assignment explicitly says the tests are the authority; I redirected it to change the function instead.

---

## 3. Debugging and testing your fixes

I decided a bug was fixed only when I could reproduce the original failure, apply the fix, and confirm the failure was gone — not just when the code "looked right." For the hint bug, I manually typed a guess I knew was too high (e.g., secret = 42, guess = 99) before and after the fix to confirm the message flipped from "Go HIGHER!" to "Go LOWER!". For the refactoring, I ran `pytest tests/test_game_logic.py` and watched it go from three failures (all `NotImplementedError`) to three passes after filling in `logic_utils.py`. AI helped me understand that the tests expected a single string return value from `check_guess`, not a tuple, which was a mismatch I would have missed just by reading `app.py` in isolation.

---

## 4. What did you learn about Streamlit and state?

The secret number kept changing because Streamlit reruns the entire script from top to bottom every time any widget changes — including clicking a button. The input widget's key was tied to the difficulty selection, so switching difficulty (or triggering any rerun) produced a new key, which Streamlit treated as a brand-new widget and re-ran the `if "secret" not in st.session_state` initialization. I would explain Streamlit reruns to a friend like this: "Imagine every button click reloads the whole page, but `st.session_state` is like a sticky note you tape to the browser — it survives the reload as long as you reference it by the same name." The fix that finally gave the game a stable secret was changing `key=f"guess_input_{difficulty}"` to the static `key="guess_input"`, so Streamlit never saw a new key and never re-initialized the secret.

---

## 5. Looking ahead: your developer habits

One habit I want to reuse is reading the tests *before* touching the code — they told me exactly what interface `check_guess` needed to have (return a plain string) before I wrote a single line of the fix. Next time I work with AI on a coding task, I would give it the test file upfront and ask it to verify its suggestions against the expected interfaces, rather than discovering mismatches after the fact. This project changed the way I think about AI-generated code: it can produce something that looks reasonable and even runs without errors, but "no crashes" is not the same as "correct behavior" — careful reading and targeted tests are still essential.
