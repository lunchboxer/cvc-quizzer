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
    right
  </button>
  <button
    class="wrong-btn"
    aria-label="Mark as Incorrect"
    on:click|stopPropagation={() => evaluateAnswer(cardIndex, false)}
  >
    wrong
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
    color: var(--text-color);
    padding: 1.5rem 4rem;
    font-size: 1.2rem;
  }

  .wrong-btn {
    background-color: var(--failure-color);
  }
  .right-btn {
    background-color: var(--success-color);
  }
</style>
