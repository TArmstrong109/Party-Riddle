<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Riddle Challenge</title>
  <style>
    :root { color-scheme: light dark; }
    body {
      font-family: system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial, "Noto Sans", "Liberation Sans", sans-serif;
      margin: 0; padding: 2rem;
      display: grid; place-items: start center;
      min-height: 100vh; background: #0f172a; color: #e2e8f0;
    }
    .card {
      max-width: 780px; width: 100%;
      background: #111827; border: 1px solid #1f2937; border-radius: 12px;
      padding: 1.5rem; box-shadow: 0 10px 30px rgba(0,0,0,0.35);
    }
    h1 { margin: 0 0 0.5rem; font-weight: 700; font-size: 1.5rem; }
    .sub { color: #94a3b8; margin-bottom: 1rem; }
    .riddle {
      white-space: pre-line; /* preserve line breaks */
      background: #0b1020; border: 1px solid #1f2937; border-radius: 10px;
      padding: 1rem; line-height: 1.6; margin-bottom: 1rem; color: #cbd5e1;
    }
    label { display: block; margin-bottom: 0.5rem; color: #cbd5e1; }
    .row { display: flex; gap: 0.75rem; }
    input[type="text"] {
      flex: 1; padding: 0.75rem 0.9rem; border-radius: 10px;
      border: 1px solid #334155; background: #0b1220; color: #e2e8f0;
    }
    button {
      padding: 0.75rem 1rem; border-radius: 10px; border: none;
      background: #22c55e; color: #0b1020; font-weight: 700; cursor: pointer;
    }
    button:hover { filter: brightness(1.1); }
    .msg { margin-top: 0.75rem; min-height: 1.25rem; }
    .msg.error { color: #fca5a5; }
    .msg.ok { color: #86efac; }
    .hint { margin-top: 0.5rem; color: #60a5fa; }
    footer { margin-top: 1rem; color: #64748b; font-size: 0.85rem; }

    /* === Modal (pop-up) styles === */
    .modal-backdrop {
      position: fixed; inset: 0;
      background: rgba(2,6,23,0.65);
      display: none; /* hidden by default */
      align-items: center; justify-content: center;
      z-index: 1000;
    }
    .modal {
      width: min(92vw, 560px);
      background: #111827; border: 1px solid #1f2937; border-radius: 12px;
      box-shadow: 0 25px 60px rgba(0,0,0,0.45);
      padding: 1.25rem; color: #e2e8f0;
    }
    .modal header {
      display: flex; justify-content: space-between; align-items: center;
      margin-bottom: 0.75rem;
    }
    .modal h2 { margin: 0; font-size: 1.25rem; }
    .close-btn {
      background: transparent; border: none; color: #94a3b8; cursor: pointer;
      font-size: 1.1rem; line-height: 1; padding: 0.25rem 0.5rem; border-radius: 8px;
    }
    .close-btn:hover { color: #e2e8f0; background: #0b1020; }
    .modal .content { line-height: 1.6; }
    .modal .address { margin-top: 0.5rem; font-weight: 700; color: #86efac; }
    .modal footer { margin-top: 1rem; text-align: right; }

    /* class that shows the modal */
    .show { display: flex; }
  </style>
</head>
<body>
  <main class="card">
    <h1>Riddle Challenge</h1>
    <p class="sub">Answer correctly to reveal the destination in a pop‑up.</p>

    <div class="riddle" id="riddleText">
I’ve chased a light across a bay,
In gilded halls where secrets sway.
I’ve frozen deep where lovers weep,
Beneath the waves where dreams don’t keep.

I’ve wandered lands both harsh and wild,
With nature’s wrath and fate reviled.
I’ve built a world inside a dream,
Where time is fluid, not what it seems.

My name’s a blend of art and lore,
A painter’s touch, an actor’s core.
Who am I, whose fame does grow,
From roaring twenties to icy snow?
    </div>

    <label for="answer">Your answer</label>
    <div class="row">
      <input id="answer" type="text" placeholder="Type your answer…" autocomplete="off" />
      <button id="submitBtn">Submit</button>
    </div>

    <div class="msg" id="feedback" aria-live="polite"></div>
    <div class="hint" id="hint" style="display:none;"></div>

    <footer>Attempts: <span id="attempts">0</span></footer>
  </main>

  <!-- === Modal (pop-up) === -->
  <div id="modalBackdrop" class="modal-backdrop" role="dialog" aria-modal="true" aria-labelledby="modalTitle">
    <div class="modal" role="document">
      <header>
        <h2 id="modalTitle">Destination Unlocked</h2>
        <button class="close-btn" id="modalCloseBtn" aria-label="Close pop-up">✕</button>
      </header>
      <div class="content">
        ✅ Correct! Here’s the address:
        <div id="modalAddress" class="address">43 Nannatee Way, Wanneroo</div>
      </div>
      <footer>
        <button class="close-btn" id="modalCloseBtn2">Close</button>
      </footer>
    </div>
  </div>

  <!-- Place script at end so DOM is ready -->
  <script>
    // === Config ===
    const ACCEPTED_ANSWERS = [
      "leonardo dicaprio",
      "leo dicaprio",
      "leonardo wilhelm dicaprio",
      "dicaprio",
      "leonardo",
      "leonardo di caprio"
    ];
    const HINTS = [
      "Hint #1: Green light across the bay in the Roaring Twenties.",
      "Hint #2: Titanic’s icy waters & The Revenant’s wilderness.",
      "Hint #3: Shares a name with a Renaissance painter."
    ];
    const HINT_AFTER_ATTEMPTS = 3;

    // ✅ Updated address
    const ADDRESS_TEXT = "43 Nannatee Way, Wanneroo";

    // === Elements ===
    const answerInput = document.getElementById('answer');
    const submitBtn   = document.getElementById('submitBtn');
    const feedback    = document.getElementById('feedback');
    const hintBox     = document.getElementById('hint');
    const attemptsEl  = document.getElementById('attempts');

    const modalBackdrop = document.getElementById('modalBackdrop');
    const modalAddress  = document.getElementById('modalAddress');
    const modalCloseBtn = document.getElementById('modalCloseBtn');
    const modalCloseBtn2= document.getElementById('modalCloseBtn2');

    let attempts = 0;
    let lastFocus = null;

    // Robust normalization: lower-case, trim, collapse spaces, remove punctuation
    function normalize(str) {
      return (str || '')
        .toLowerCase()
        .trim()
        .replace(/\s+/g, ' ')
        .replace(/[^\p{L}\p{N}\s]/gu, ''); // remove punctuation safely
    }

    function openModal(addressText) {
      modalAddress.textContent = addressText;
      modalBackdrop.classList.add('show');
      lastFocus = document.activeElement;
      modalCloseBtn.focus();

      modalBackdrop.addEventListener('click', backdropHandler);
      document.addEventListener('keydown', escHandler);
    }

    function closeModal() {
      modalBackdrop.classList.remove('show');
      modalBackdrop.removeEventListener('click', backdropHandler);
      document.removeEventListener('keydown', escHandler);
      if (lastFocus) lastFocus.focus();
    }

    function backdropHandler(e) {
      if (e.target === modalBackdrop) closeModal();
    }
    function escHandler(e) {
      if (e.key === 'Escape') closeModal();
    }

    modalCloseBtn.addEventListener('click', closeModal);
    modalCloseBtn2.addEventListener('click', closeModal);

    function checkAnswer() {
      const user = normalize(answerInput.value);
      attempts++;
      attemptsEl.textContent = attempts;

      const ok = ACCEPTED_ANSWERS.map(normalize).includes(user);

      if (ok) {
        feedback.textContent = "Nice! That’s correct.";
        feedback.className = "msg ok";
        openModal(ADDRESS_TEXT);
        submitBtn.disabled = true;
        answerInput.disabled = true;
        hintBox.style.display = "none";
      } else {
        feedback.textContent = "Not quite. Try again!";
        feedback.className = "msg error";
        if (attempts >= HINT_AFTER_ATTEMPTS) {
          const hintIndex = Math.min(attempts - HINT_AFTER_ATTEMPTS, HINTS.length - 1);
          hintBox.textContent = HINTS[hintIndex];
          hintBox.style.display = "block";
        }
      }
    }

    // Submit on Enter
    answerInput.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') checkAnswer();
    });
    submitBtn.addEventListener('click', checkAnswer);
  </script>
</body>
</html>
