<script>
    import { cvcWords } from "$lib/words.js";
    import { flip } from "svelte/animate";
    import { quintOut } from "svelte/easing";
    import { onMount } from "svelte";
    import "../app.css";

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
    let isDarkMode = false;

    // Check system preference on mount
    onMount(() => {
        isDarkMode = window.matchMedia("(prefers-color-scheme: dark)").matches;
        updateTheme();
    });

    function toggleTheme() {
        isDarkMode = !isDarkMode;
        updateTheme();
    }

    function updateTheme() {
        document.documentElement.classList.toggle("dark-mode", isDarkMode);
    }

    function toggleCard(index) {
        // Prevent flipping if the card is already answered or another card is being evaluated
        if (cards[index].isAnswered || cardIndex !== undefined) return;

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

<header>
    <h1>CVC Quizzer</h1>
    <div class="score">Score: {score}</div>
    <div class="theme-toggle">
        <button on:click={toggleTheme}>
            {isDarkMode ? "Light Mode" : "Dark Mode"}
        </button>
    </div>
</header>

<main>
    <div class="grid-container">
        {#if cardIndex !== undefined}
            <div
                class="answer-buttons"
                role="group"
                aria-label="Answer Evaluation"
            >
                <button
                    class="right-btn"
                    aria-label="Mark as Correct"
                    on:click|stopPropagation={() =>
                        evaluateAnswer(cardIndex, true)}
                >
                    Right
                </button>
                <button
                    class="wrong-btn"
                    aria-label="Mark as Incorrect"
                    on:click|stopPropagation={() =>
                        evaluateAnswer(cardIndex, false)}
                >
                    Wrong
                </button>
            </div>
        {/if}
        <div class="card-grid">
            {#each cards as card, index (card.word)}
                <div
                    class="card"
                    role="gridcell"
                    aria-label={`Card ${index + 1}: Number ${card.number}`}
                    tabindex="0"
                    class:flipped={card.isFlipped}
                    class:correct={card.isCorrect === true}
                    class:incorrect={card.isCorrect === false}
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
        <div id="game-instructions" class="sr-only">
            Flip cards to reveal CVC words. Evaluate each word as right or wrong
            to score points.
        </div>
    </div>
</main>

<style>
    h1 {
        font-weight: inherit;
    }

    main {
        padding: 1rem;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .grid-container {
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;
        flex-direction: column;
        align-items: center;
        position: fixed;
        bottom: 1rem;
        width: calc(100% - 6rem);
    }

    .score {
        font-size: 1.5rem;
        margin-bottom: 1rem;
    }

    .card-grid {
        display: grid;
        grid-template-columns: repeat(3, 1fr); /* Fixed 3 columns */
        gap: 0.6rem; /* Default gap */
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

    .answer-buttons {
        display: flex;
        justify-content: center;
        padding: 2rem;
        gap: 1rem;
    }

    .answer-buttons button {
        padding: 0.5rem 1rem;
        font-size: 1rem;
        cursor: pointer;
        color: var(--answer-button-text);
        border-radius: 4px;
    }

    .answer-buttons button:hover {
        background-color: var(--button-hover-bg);
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

    header {
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;
        align-items: baseline;
        padding: 1rem 2rem;
        margin-bottom: 1rem;
    }
    .theme-toggle button {
        background-color: var(--button-bg);
        color: var(--button-text);
        border: 1px solid var(--card-border);
        padding: 10px 15px;
        border-radius: 5px;
        cursor: pointer;
        transition:
            background-color 0.3s,
            color 0.3s;
    }

    .theme-toggle button:hover {
        background-color: var(--button-hover-bg);
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
    @media (min-width: 640px) {
        main {
            padding-bottom: 2rem;
        }
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
