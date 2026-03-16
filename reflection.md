# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?

The game had a sidebar with three difficulty options. When I started with Normal difficulty and guessed 47, the game kept telling me to go lower even though the secret number was 71 — the hints were completely backwards. When I finished a round and switched to Easy and clicked New Game, the secret number was still in the Normal/Hard range (above 20), meaning the difficulty change was not resetting the game properly. The score also never reset between games, carrying over from previous rounds.

- List at least two concrete bugs you noticed at the start:

1. **The hints were backwards** — when my guess was too low, the game said "Go LOWER", and when it was too high, it said "Go HIGHER".
2. **The secret number did not reset when difficulty changed** — switching to Easy still produced secrets above 20, which is outside the Easy range of 1–20.
3. **The submit button recorded the previous guess again** — the history would show the same number repeated across attempts even when a new number was typed.
4. **The score never reset** when starting a new game or changing difficulty.
5. **Invalid input (like letters) wasted an attempt** — the attempt counter incremented before the input was even validated.

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project?

I used Claude Code (claude-sonnet-4.6) as my primary AI tool throughout this project to help identify, explain, and fix the bugs in the app.

- Give one example of an AI suggestion that was correct:

Claude identified that the `check_guess` function had both branches returning `"Too Low"` — one branch said `"Too Low"` with "Go LOWER!" and the other also said `"Too Low"` with "Go HIGHER!". It suggested changing the first branch to return `"Too High"` with "Go LOWER!". I verified this was correct by reading the logic: when `guess > secret`, the guess is too high so the player should go lower.

- Give one example of an AI suggestion that was incorrect or misleading:

The first attempt at fixing the repeated history bug was to use `st.form` with `clear_on_submit=True`. This seemed like a clean solution, but the problem still occurred because Streamlit triggered an additional rerun after clearing the form where the old cached widget value could still be processed. The real fix required also calling `st.rerun()` after processing each guess to force a completely clean render cycle.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?

I manually tested each bug by playing the game after each fix — for example, submitting a guess lower than the secret and verifying the hint said "Go HIGHER", or switching difficulty and confirming the secret number was within the new range. I also watched the Developer Debug Info panel to verify that the secret, attempts, score, and history all updated correctly after each action.

- Describe at least one test you ran:

I tested the hint direction by opening the Developer Debug Info expander to reveal the secret number, then submitting a guess I knew was lower than the secret. Before the fix, the game said "Go LOWER" — after the fix it correctly said "Go HIGHER". I repeated this with a guess higher than the secret to confirm the opposite direction also worked.

- Did AI help you design or understand any tests?

Yes. Claude explained that the type-switching bug (converting the secret to a string on even attempts) was causing the `check_guess` function to fall into the `except TypeError` branch on even attempts, where a separate broken code path ran. This helped me understand why bugs were intermittent — some guesses worked and some didn't depending on whether the attempt count was odd or even.

---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.

In the original app, the secret was generated with `random.randint(low, high)` every time the script ran, with only a weak guard: `if "secret" not in st.session_state`. The problem was that `low` and `high` were derived from the current difficulty setting, so when difficulty changed, the condition `st.session_state.get("difficulty") != difficulty` was never checked — the old secret stayed in session state but now fell outside the new range.

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?

Every time a user clicks a button, types in a box, or changes a widget in Streamlit, the entire Python script runs again from top to bottom — this is called a rerun. Normal Python variables reset to nothing on each rerun, so Streamlit provides `st.session_state`, which is like a notebook that persists between reruns. If you store a value in `st.session_state`, it survives the rerun and you can read it back on the next pass through the script.

- What change finally gave the game a stable secret number?

I added a check at the top of the script that tracked the current difficulty in `st.session_state`. If the stored difficulty did not match the selected difficulty, the game reset everything — a new secret within the correct range, zeroed attempts, cleared history, and zeroed score. This meant the secret was always consistent with whichever difficulty was active.

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?

Using a visible debug panel (like the Developer Debug Info expander) while testing was extremely valuable. Being able to see the secret number, attempt count, score, and history in real time made it much faster to confirm whether a fix actually worked instead of guessing from behavior alone. I want to build debug panels like this into future Streamlit projects from the start.

- What is one thing you would do differently next time you work with AI on a coding task?

I would test each AI-suggested fix immediately and independently before moving on to the next bug. In this project, the repeated history bug required two rounds of fixes because the first suggestion (`clear_on_submit=True`) was not fully verified before the next session. Running the app and manually checking each behavior right after a fix would catch incomplete solutions earlier.

- In one or two sentences, describe how this project changed the way you think about AI generated code.

This project made it clear that AI-generated code can look clean and complete while containing multiple subtle logic bugs that only surface during real use. AI is a fast starting point, but the responsibility for testing and verification still belongs to the developer.
