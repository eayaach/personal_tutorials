<script>
  let count = 0
</script>

<main>
  <h1>Hello, Svelte!</h1>
  <p>Count: {count}</p>
  <button on:click={() => count++}>Click me</button>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    font-family: sans-serif;
  }

  h1 {
    font-size: 2rem;
    margin-bottom: 1rem;
  }

  button {
    margin-top: 1rem;
    padding: 0.5rem 1.5rem;
    font-size: 1rem;
    cursor: pointer;
    border: none;
    border-radius: 6px;
    background: #ff3e00;
    color: white;
  }

  button:hover {
    background: #cc3200;
  }
</style>
