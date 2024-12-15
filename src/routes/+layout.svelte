<script>
  import { onMount } from 'svelte'
  import Footer from '$lib/footer.svelte'
  import { Fa } from 'svelte-fa'
  import { faSun, faMoon } from '@fortawesome/free-solid-svg-icons'
  import '../app.css'
  let isDarkMode = true

  onMount(() => {
    isDarkMode = globalThis.matchMedia('(prefers-color-scheme: dark)').matches
    updateTheme()
  })

  function toggleTheme() {
    isDarkMode = !isDarkMode
    updateTheme()
  }

  function updateTheme() {
    document.documentElement.classList.toggle('dark-mode', isDarkMode)
  }
</script>

<header>
  <h1>CVC Quizzer</h1>
  <button on:click={toggleTheme}><Fa icon={isDarkMode ? faSun : faMoon} /></button>
</header>
<slot />
<Footer />

<style>
  h1 {
    font-weight: inherit;
    margin: 0;
    padding: 0;
  }
  button {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    margin: 0;
  }
  header {
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
    align-items: baseline;
    padding: 1rem;
  }
</style>
