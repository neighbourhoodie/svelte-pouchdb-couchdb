<script>
  import { createEventDispatcher } from 'svelte'
  import { fade } from 'svelte/transition'
  import { debounce } from 'lodash'

  const dispatch = createEventDispatcher()

  function remove() {
    dispatch('remove', {todo})
  }

  function updateText() {
    dispatch('update', {todo})
  }
  // On attend quelques instants pour modifier l'action après frappé grâce à la fonction debounde
  const debouncedUpdateText = debounce(updateText, 500)

  function toggleStatus() {
    dispatch('update', {
      todo: {
        ...todo,
        complete: !todo.complete
      }
    })
  }

  export let todo
</script>

<style>
  .is-complete {
    text-decoration: line-through;
    color: green;
    width: 440px;
    display: inline-block;
  }
  input[type="text"] {
    width: 440px;
  }
  input[disabled] {
    background: none;
    border: 1px solid #0000;
  }
  button {
    border-radius: 50%;
    width: 2.25em;
    height: 2.25em;
    line-height: 1.3em;
    text-align: center;
    margin-left: 0.75em;
    padding: 0;
  }
</style>

<li transition:fade>
  {#if todo.complete}
    <input class='is-complete' value={todo.text} disabled />
    <button on:click={toggleStatus}>❌</button>
  {:else}
    <!-- On détect les clicks et met à jour l'état du todo -->
    <input type='text' on:keyup={debouncedUpdateText} bind:value={todo.text}>
    <button on:click={toggleStatus}>✔️</button>
  {/if}
  <button on:click={remove}>💥</button>
</li>
