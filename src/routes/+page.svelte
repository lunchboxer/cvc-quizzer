<script>
  import { cvcWords } from "$lib/words.js";
  import { flip } from "svelte/animate";
  import { quintOut } from "svelte/easing";

  // Create an array of integers from 11-20
  const numbers = Array.from({ length: 10 }, (_, i) => i + 11).sort(
    () => 0.5 - Math.random(),
  );

  // Shuffle the CVC words and numbers, taking the first 9 of each
  let cards = cvcWords
    .sort(() => 0.5 - Math.random())
    .slice(0, 9)
    .map((word, index) => ({
      word,
      number: numbers[index],
      isFlipped: false,
      isAnswered: false,
      isCorrect: null,
    }));

  let score = 0;
  let cardIndex;

  function toggleCard(index) {
    if (cardIndex !== undefined) return;
    cards[index].isFlipped = !cards[index].isFlipped;
    cardIndex = index;
    cards = cards;
  }

  function evaluateAnswer(index, isCorrect) {
    if (cards[index].isAnswered) return;

    cards[index].isAnswered = true;
    cards[index].isCorrect = isCorrect;

    if (isCorrect) {
      score += 1;
    }

    cards = cards;
    cardIndex = undefined;
  }
</script>

<main>
  <h1>CVC Quizzer</h1>
  <div class="game-container">
    <div class="score">Score: {score}</div>
    <div
      class="card-grid"
      role="grid"
      aria-label="CVC Word Game Grid"
      aria-describedby="game-instructions"
    >
      {#each cards as card, index (card.word)}
        <div
          class="card"
          role="gridcell"
          aria-label={`Card ${index + 1}: Number ${card.number}`}
          tabindex="0"
          class:flipped={card.isFlipped}
          class:correct={card.isCorrect === true}
          class:incorrect={cards[index].isCorrect === false}
          on:click={() => toggleCard(index)}
          on:keydown={(e) => {
            if (e.key === "Enter" || e.key === " ") {
              toggleCard(index);
            }
          }}
          animate:flip={{ duration: 300, easing: quintOut }}
        >
          <div class="card-inner">
            <div class="card-front">{card.number}</div>
            <div class="card-back">
              {card.word}
            </div>
          </div>
        </div>
      {/each}
    </div>
    {#if cardIndex !== undefined}
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
    {/if}
    <div id="game-instructions" class="sr-only">
      Flip cards to reveal CVC words. Evaluate each word as right or wrong to
      score points.
    </div>
  </div>
</main>

<style>
  :root {
    --success-color: #689f38;
    --failure-color: #d32f2f;
  }
  h1 {
    font-weight: inherit;
  }
  main {
    padding: 2rem;
  }
  .game-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .score {
    font-size: 1.5rem;
    margin-bottom: 1rem;
  }

  .card-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
    max-width: 600px;
    margin: 0 auto;
  }

  .card {
    aspect-ratio: 1;
    perspective: 1000px;
    width: 10rem;
    cursor: pointer;
    height: 10rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    border: 2px solid #e0e0e0;
    border-radius: 12px;
    transition: transform 0.2s;
  }

  .card:hover {
    transform: scale(1.05);
  }

  .card-inner {
    position: relative;
    width: 100%;
    height: 100%;
    text-align: center;
    transition: transform 0.6s;
    transform-style: preserve-3d;
  }

  .card.flipped .card-inner {
    transform: rotateY(180deg);
  }

  .card-front,
  .card-back {
    position: absolute;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    border-radius: 10px;
    backface-visibility: hidden;
  }

  .card-front {
    background-color: #444;
    color: white;
    font-size: 3rem;
  }

  .card-back {
    background-color: #999;
    color: black;
    font-size: 3rem;
    transform: rotateY(180deg);
  }

  .card.correct .card-back {
    background-color: var(--success-color);
  }

  .card.incorrect .card-back {
    background-color: var(--failure-color);
  }

  .answer-buttons {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-top: 1rem;
  }

  .right-btn,
  .wrong-btn {
    padding: 0.5rem 1rem;
    font-size: 1rem;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .right-btn {
    background-color: lightgreen;
  }

  .wrong-btn {
    background-color: lightcoral;
  }

  .right-btn:hover {
    background-color: var(--success-color);
  }

  .wrong-btn:hover {
    background-color: var(--failure-color);
  }

  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }
</style>
