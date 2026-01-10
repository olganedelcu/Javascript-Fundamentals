### What is debouncing?

**function debouncing** is useful whenever you are handling some sort of events, for example: imagine you’re building a website with a **search bar.**
as users type, you want to **fetch** suggestions from the **backend**.
so each time the user types a letter, your code might try to make a request —
but that would be **way too many requests** (e.g. one per keystroke).
that’s where **debouncing** comes in.
it delays the function execution by a fixed interval (e.g. 500ms).
if the function gets called again during that interval, it resets the timer.
🚫 this prevents **multiple unnecessary calls** and only **runs the function when** the user **stops typing** for a moment.

super useful for:

🖱️ search input bars
📏 window resize events
📝 form validations on input
🔍 filters with real-time data
