<script>
  import { onMount } from 'svelte'
  import mqtt from 'mqtt'
  import { channelPrefix, mqttUrl } from '$lib/constants'
  import CardGrid from '$lib/card-grid.svelte'
  import AnswerButtons from '$lib/answer-buttons.svelte'

  let code = ''
  const clientId = 'controller_' + Math.random().toString(36).slice(2, 8)
  let client
  let isConnected = false
  let cardIndex
  let cards = []
  let score = 0
  let showRightWrong = false

  const evaluateAnswer = (index, isCorrect) => {
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

  const togglerightwrong = () => {
    if (isConnected) {
      publishState()
    }
  }

  const toggleCard = (index) => {
    // Prevent flipping if the card is already answered or another card is being evaluated
    if (cards[index].isAnswered || cardIndex !== undefined) return

    cards[index].isFlipped = !cards[index].isFlipped
    cardIndex = index
    cards = [...cards]
    publishState()
  }

  const publishState = () => {
    const channel = `${channelPrefix}-${code.toUpperCase()}`
    client.publish(
      `${channel}/state`,
      JSON.stringify({
        type: 'state',
        cards,
        score,
        cardIndex,
        showRightWrong,
        message: 'state',
      }),
    )
  }

  function disconnect() {
    client.end()
    cards = []
    isConnected = false
  }
  async function connectToGame() {
    if (code.length !== 4) {
      return
    }

    client = mqtt.connect(mqttUrl)

    client.on('connect', () => {
      const channel = `${channelPrefix}-${code.toUpperCase()}`
      console.log('Connected to MQTT broker')
      client.subscribe(`${channel}/#`)
      // Send connection message
      client.publish(
        `${channel}/connect`,
        JSON.stringify({
          type: 'connect',
          clientId,
          message: 'connected',
        }),
      )
    })

    client.on('message', (topic, message) => {
      try {
        const data = JSON.parse(message.toString())
        console.log('Received message:', data, topic)
        console.log(clientId, data.clientId)
        console.log(clientId === data.clientId)
        if (data.type === 'connected_ack' && data.clientId === clientId) {
          console.log('Connected to game: ' + data.gameId)
          isConnected = true
          cards = data.cards
          score = data.score
          cardIndex = data.cardIndex
        }
        if (data.type === 'state') {
          cards = data.cards
          score = data.score
          cardIndex = data.cardIndex
        }
      } catch (error) {
        console.error('Error parsing message:', error)
      }
    })

    client.on('error', (error) => {
      console.error('Connection error: ' + error.message)
    })
  }

  onMount(() => {
    return () => {
      if (client) {
        client.end()
      }
    }
  })
</script>

<main>
  <div class="game-info">
    <h2>Game Controller</h2>
    {#if cards.length > 0}
      <div class="score">Score: {score}</div>
    {/if}
    <div class="connect-form">
      {#if isConnected}
        <button on:click={disconnect}>Disconnect from {code.toUpperCase()}</button>
      {:else}
        <label for="code">Game Code:</label>
        <input type="text" bind:value={code} maxlength="4" />
        <button on:click={connectToGame}>Connect</button>
      {/if}
      <input type="checkbox" bind:checked={showRightWrong} on:change={togglerightwrong} />
      <label for="showRightWrong">Show Right/Wrong</label>
    </div>
  </div>

  {#if cards.length > 0}
    <div class="grid-container">
      {#if cardIndex !== undefined}
        <AnswerButtons {cardIndex} {evaluateAnswer} />
      {/if}
      <CardGrid {cards} {toggleCard} />
    </div>
  {/if}
</main>

<style>
  main {
    padding: 1rem;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
  }

  .score {
    font-size: 1.5rem;
    margin-bottom: 1rem;
  }
  .game-info {
    display: flex;
    justify-content: space-between;
    width: 100%;
    align-items: baseline;
  }
  .connect-form {
    margin: 2rem 0;
    display: flex;
    gap: 1rem;
    justify-content: center;
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
  label {
    padding: 0.5rem;
    font-size: 1.2rem;
  }

  input[type='text'] {
    padding: 0.5rem;
    font-size: 1.2rem;
    width: 120px;
    text-align: center;
    text-transform: uppercase;
  }
</style>
