
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
      max-width: 680px; width: 100%;
      background: #111827; border: 1px solid #1f2937; border-radius: 12px;
      padding: 1.5rem; box-shadow: 0 10px 30px rgba(0,0,0,0.35);
    }
    h1 { margin: 0 0 0.5rem; font-weight: 700; font-size: 1.5rem; }
    .sub { color: #94a3b8; margin-bottom: 1rem; }
    .riddle {
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
      background: rgba(2,6,23,0.65); /* dark translucent */
      display: none;
      align-items: center; justify-content: center;
      z-index: 1000;
    }
    .modal {
      width: min(92vw, 520px);
      background: #111827; border: 1px solid #1f2937; border-radius: 12px;
      box-shadow: 0 25px 60px rgba(0,0,0,0.45);
      padding: 1.25rem;
      color: #e2e8f0;
    }
    .modal header {
      display: flex; justify-content: space-between; align-items: center;
      margin-bottom: 0.75rem;
    }
    .modal h2 { margin: 0; font-size: 1.25rem; }
    .close-btn {
      background: transparent; border: none; color: #94a3b8; cursor: pointer;
      font-size: 1.25rem; line-height: 1; padding: 0.25rem 0.5rem; border-radius: 8px;
    }
    .close-btn:hover { color: #e2e8f0; background: #0b1020; }
    .modal .content { line-height: 1.6; }
    .modal .address {
      margin-top: 0.5rem; font-weight: 700; color: #86efac;
    }
    .modal footer {
      margin-top: 1rem; text-align: right;
    }
    .modal-show { display: flex; }
  </style>
</head>
<body>
  <main class="card">
    <h1>Riddle Challenge</h1>
    <p class="sub">Answer correctly to reveal the destination in a pop‑up.</p>

    <div class="riddle" id="riddleText">
      I speak without a mouth and hear without ears. I have no body, but I come alive with wind. What am I?
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
        <div id="modalAddress" class="address">123 Example Street, Maddington WA 6109</div>
      </div>
      <footer>
        <button class="close-btn" id="modalCloseBtn2">Close</button>
      </footer>
    </div>
  </div>

  <script>
    // === Configuration ===
    const ACCEPTED_ANSWERS = [
      "echo",
      "an echo",
      "the echo"
    ];
    const HINTS = [
      "Hint #1: You hear it in caves.",
      "Hint #2: It repeats what you say."
    ];
    const HINT_AFTER_ATTEMPTS = 3;
    const ADDRESS_TEXT = "123 Example Street, Maddington WA 6109";

    // === Elements ===
    const answerInput = document.getElementById('answer');
    const submitBtn  = document.getElementById('submitBtn');
    const feedback   = document.getElementById('feedback');
    const hintBox    = document.getElementById('hint');
    const attemptsEl = document.getElementById('attempts');

    const modalBackdrop = document.getElementById('modalBackdrop');
    const modalAddress  = document.getElementById('modalAddress');
    const modalCloseBtn = document.getElementById('modalCloseBtn');
    const modalCloseBtn2= document.getElementById('modalCloseBtn2');

    let attempts = 0;
    let lastFocus = null;

    function normalize(str) {
      return (str || '')
        .toLowerCase()
        .trim()
        .replace(/[^\p{L}\p{N}\s-]/gu, ''); // remove punctuation safely (unicode-aware)
    }

    function openModal(addressText) {
      // Set address text
      modalAddress.textContent = addressText;
      // Show modal
      modalBackdrop.classList.add('modal-show');
      // Basic focus management
      lastFocus = document.activeElement;
      modalCloseBtn.focus();

      // Close when clicking outside the dialog
      modalBackdrop.addEventListener('click', backdropHandler);
      // Close on Esc
      document.addEventListener('keydown', escHandler);
    }

    function closeModal() {
      modalBackdrop.classList.remove('modal-show');
      modalBackdrop.removeEventListener('click', backdropHandler);
      document.removeEventListener('keydown', escHandler);
      if (lastFocus) { lastFocus.focus(); }
    }

    function backdropHandler(e) {
      // If user clicked directly on the backdrop (not inside modal box), close
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
        // Progressive hints
        if (attempts >= HINT_AFTER_ATTEMPTS) {
          const hintIndex = Math.min(attempts - HINT_AFTER_ATTEMPTS, HINTS.length - 1);
          hintBox.textContent = HINTS[hintIndex];
          hintBox.style.display = "block";
        }
      }
    }

    // Allow Enter key
    answerInput.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') { checkAnswer(); }
    });
    submitBtn.addEventListener('click', checkAnswer);
  </script>
</body>
</html>
