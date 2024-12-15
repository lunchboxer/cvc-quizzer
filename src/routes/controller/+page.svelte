<script>
  import { onMount } from 'svelte'
  import mqtt from 'mqtt'
  import { channelPrefix, mqttUrl } from '$lib/constants'
  import CardGrid from '$lib/card-grid.svelte'
  import AnswerButtons from '$lib/answer-buttons.svelte'
  import Fa from 'svelte-fa'
  import {
    faRepeat,
    faPaperPlane,
    faPlugCircleXmark,
    faBroom,
  } from '@fortawesome/free-solid-svg-icons'

  let code = ''
  const clientId = 'controller_' + Math.random().toString(36).slice(2, 8)
  let client
  let isConnected = false
  let cardIndex
  let cards = []
  let score = 0
  let showRightWrong = false

  onMount(() => {
    const storedCode = localStorage.getItem('code')
    if (storedCode) {
      code = storedCode
      connectToGame()
    }
    return () => {
      if (client) {
        client.end()
      }
    }
  })

  function updateCode(newCode) {
    code = newCode
    localStorage.setItem('code', code)
  }

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

  const resetGame = () => {
    const channel = `${channelPrefix}-${code.toUpperCase()}`
    client.publish(
      `${channel}/reset`,
      JSON.stringify({
        type: 'reset',
        message: 'reset',
      }),
    )
  }

  const togglerightwrong = () => {
    showRightWrong = !showRightWrong
    if (isConnected) {
      publishState()
    }
  }

  const pullState = () => {
    const channel = `${channelPrefix}-${code.toUpperCase()}`
    client.publish(
      `${channel}/state`,
      JSON.stringify({
        type: 'pullState',
        clientId,
        message: 'pullState',
      }),
    )
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
    cards = []
    isConnected = false
    const channel = `${channelPrefix}-${code.toUpperCase()}`
    client.publish(
      `${channel}/disconnect`,
      JSON.stringify({
        type: 'disconnect',
        message: 'disconnect',
      }),
    )
    client.end()
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
          if (data.clientId && data.clientId !== clientId) return
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
</script>

<main>
  <div class="game-info">
    <h2>Game Controller</h2>
    {#if cards.length > 0}
      <div class="score">Score: {score}</div>
    {/if}
    <div class="connect-form">
      {#if isConnected}
        <button on:click={resetGame}><Fa icon={faBroom} /></button>
        <button on:click={pullState}><Fa icon={faRepeat} /></button>
        <button on:click={publishState}><Fa icon={faPaperPlane} /></button>
        <button on:click={disconnect}><Fa icon={faPlugCircleXmark} /></button>
      {:else}
        <label for="code">Game Code:</label>
        <input
          type="text"
          bind:value={code}
          maxlength="4"
          on:input={(event) => updateCode(event.target.value)}
        />
        <button on:click={connectToGame}>Connect</button>
      {/if}
      <button on:click={togglerightwrong}>
        {#if showRightWrong}
          hide buttons
        {:else}
          show buttons
        {/if}
      </button>
    </div>
  </div>

  {#if cards.length > 0}
    <div class="grid-container">
      <CardGrid {cards} {toggleCard} {cardIndex} />
    </div>
    {#if cardIndex !== undefined}
      <AnswerButtons {cardIndex} {evaluateAnswer} />
    {/if}
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
    flex-wrap: wrap;
  }
  .connect-form {
    margin: 2rem 0;
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    justify-content: center;
  }

  .grid-container {
    display: flex;
    justify-content: space-between;
    flex-direction: column;
    align-items: center;
    width: calc(100% - 6rem);
    margin-bottom: 2rem;
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
