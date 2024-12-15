<script>
  import { flip } from 'svelte/animate'
  import { quintOut } from 'svelte/easing'

  export let cards = []
  export let toggleCard
</script>

<div class="card-grid">
  {#each cards as card, index (card.word)}
    <div
      class="card {card.isCorrect ? 'correct-animation' : ''}"
      role="gridcell"
      aria-label={`Card ${index + 1}: Number ${card.number}`}
      tabindex="0"
      class:flipped={card.isFlipped}
      class:correct={card.isCorrect === true}
      class:incorrect={card.isCorrect === false}
      on:click={() => toggleCard(index)}
      on:keydown={(event) => {
        if (event.key === 'Enter' || event.key === ' ') {
          toggleCard(index)
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

<style>
  .card-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 0.6rem;
  }

  .card {
    aspect-ratio: 1;
    perspective: 1000px;
    cursor: pointer;
    font-size: 2rem;
    width: 6.4rem;
    height: 6.4rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s;
    color: var(--text-color);
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
    backface-visibility: hidden;
  }

  .card-front {
    background-color: var(--card-bg);
    color: var(--text-color);
  }

  .card-back {
    background-color: var(--card-bg-alt);
    color: var(--text-color);
    transform: rotateY(180deg);
  }

  .card.correct .card-back {
    background-color: var(--success-color);
    color: var(--answer-button-text);
  }

  .card.incorrect .card-back {
    background-color: var(--failure-color);
    color: var(--answer-button-text);
  }

  .correct-animation {
    animation: flip 0.6s ease;
  }

  @keyframes flip {
    0% {
      transform: rotateY(0);
    }
    25% {
      transform: rotateY(180deg);
    }
    50% {
      transform: rotateY(0);
    }
    75% {
      transform: rotateY(180deg);
    }
    100% {
      transform: rotateY(0);
    }
  }

  @media (min-width: 640px) {
    .card-grid {
      gap: 1rem;
    }
    .card {
      width: 10rem;
      height: 10rem;
      font-size: 3rem;
    }
  }
  @media (min-width: 768px) {
    .card {
      width: 13rem;
      height: 13rem;
      font-size: 4rem;
    }
    .card-grid {
      gap: 1.5rem;
    }
  }
</style>
