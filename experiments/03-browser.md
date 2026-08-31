# Experiment 03 — Browser Automation

## Task
Open Google, search for `RTX 4050 laptop GPU`, and return the first three results.

## Intended sequence

1. Open browser.
2. Navigate to Google.
3. Enter the search query.
4. Submit.
5. Inspect results.
6. Extract the first three.
7. Respond with the results.

## Result
PARTIAL/FAIL — the model understood the task, but reliable browser state management and multi-step continuation were not achieved.

## Lesson
Browser automation requires persistent state, observation, action execution, and continuation; textual reasoning alone is insufficient.
