<script>
  export let cardIndex
  export let evaluateAnswer
</script>

<div class="answer-buttons" role="group" aria-label="Answer Evaluation">
  <button
    class="right-btn"
    aria-label="Mark as Correct"
    on:click|stopPropagation={() => evaluateAnswer(cardIndex, true)}
  >
    Right
  </button>
  <button
    class="wrong-btn"
    aria-label="Mark as Incorrect"
    on:click|stopPropagation={() => evaluateAnswer(cardIndex, false)}
  >
    Wrong
  </button>
</div>

<style>
  .answer-buttons {
    display: flex;
    justify-content: center;
    padding: 1rem;
    gap: 1rem;
  }
  .answer-buttons button {
    color: var(--answer-button-text);
    padding: 1rem 3rem;
  }

  .answer-buttons button:hover {
    background-color: var(--button-hover-bg);
    color: var(--button-text);
  }
  .wrong-btn {
    background-color: var(--failure-color);
    border-color: var(--failure-color);
  }
  .right-btn {
    background-color: var(--success-color);
    color: var(--button-text);
    border-color: var(--success-color);
  }
</style>
