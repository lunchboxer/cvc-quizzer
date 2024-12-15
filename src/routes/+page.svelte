<script>
  import { cvcWords } from '$lib/words.js'
  import { SvelteSet } from 'svelte/reactivity'
  import CardGrid from '$lib/card-grid.svelte'
  import AnswerButtons from '$lib/answer-buttons.svelte'
  import { onMount } from 'svelte'
  import mqtt from 'mqtt'
  import { channelPrefix, mqttUrl } from '$lib/constants'

  let score = 0
  let cardIndex
  let showRightWrong = true
  const gameCode = Math.random().toString(36).slice(2, 6).toUpperCase()
  const channel = `${channelPrefix}-${gameCode}`
  console.log(`Game Code: ${gameCode}`)
  const connectedControllers = new SvelteSet()
  let client
  let cards = []

  function shuffleCards() {
    // Create an array of integers from 11-20
    const numbers = Array.from({ length: 10 }, (_, index) => index + 11).sort(
      () => 0.5 - Math.random(),
    )

    // Shuffle the CVC words and numbers, taking the first 9 of each
    cards = cvcWords
      .sort(() => 0.5 - Math.random())
      .slice(0, 9)
      .map((word, index) => ({
        word,
        number: numbers[index],
        isFlipped: false,
        isAnswered: false,
        isCorrect: undefined,
      }))
  }

  shuffleCards()

  // Connect to MQTT broker
  onMount(() => {
    client = mqtt.connect(mqttUrl)

    client.on('connect', () => {
      console.log('Connected to MQTT broker')
      client.subscribe(`${channel}/#`)
    })

    client.on('message', (topic, message) => {
      try {
        const data = JSON.parse(message.toString())
        console.log('Received message:', data, topic)
        if (data.type === 'connect') {
          connectedControllers.add(data.clientId)
          // Echo back the connection
          client.publish(
            `${channel}/connect`,
            JSON.stringify({
              type: 'connected_ack',
              clientId: data.clientId,
              cards,
              score,
              cardIndex,
              gameId: gameCode,
              message: 'connected',
            }),
          )
        }

        if (data.type === 'state') {
          cards = data.cards
          score = data.score
          showRightWrong = data.showRightWrong
          cardIndex = data.cardIndex
        }
        if (data.type === 'reset') {
          resetGame()
        }
      } catch (error) {
        console.error('Error parsing message:', error)
      }
    })

    return () => {
      if (client) {
        client.end()
      }
    }
  })

  function resetGame() {
    shuffleCards()
    score = 0
    cardIndex = undefined
    publishState()
  }

  function toggleCard(index) {
    // Prevent flipping if the card is already answered or another card is being evaluated
    if (cards[index].isAnswered || cardIndex !== undefined) return

    cards[index].isFlipped = !cards[index].isFlipped
    cardIndex = index
    cards = [...cards]
    publishState()
  }

  function evaluateAnswer(index, isCorrect) {
    if (cards[index].isAnswered) return

    cards[index].isAnswered = true
    cards[index].isCorrect = isCorrect

    if (isCorrect) {
      score += 1
    }

    cards = [...cards]
    cardIndex = undefined
    publishState()
  }
  const publishState = () => {
    client.publish(
      `${channel}/state`,
      JSON.stringify({
        type: 'state',
        cards,
        score,
        cardIndex,
        message: 'state',
      }),
    )
  }
</script>

<main>
  <div class="game-info">
    <div class="score">Score: {score}</div>
    <p>Game Code: {gameCode}</p>
    <p>Connected Controllers: {connectedControllers.size}</p>
    <button on:click={resetGame}>Reset Game</button>
  </div>
  <div class="grid-container">
    {#if cardIndex !== undefined && showRightWrong}
      <AnswerButtons {cardIndex} {evaluateAnswer} />
    {/if}
    <CardGrid {cards} {toggleCard} />
    <div id="game-instructions" class="sr-only">
      Flip cards to reveal CVC words. Evaluate each word as right or wrong to score points.
    </div>
  </div>
</main>

<style>
  main {
    padding: 1rem;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
  }

  .game-info {
    display: flex;
    flex-wrap: wrap;
    width: 100%;
    gap: 1rem;
    justify-content: space-between;
    align-items: baseline;
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
  }
</style>
