<script>
  import { createEventDispatcher } from 'svelte'

  const dispatch = createEventDispatcher()

  function remove() {
    dispatch('remove', {todo})
  }

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
    display: inline-block;
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

<li>
  {#if todo.complete}
    <span class='is-complete'>{todo.text}</span>
    <button on:click={toggleStatus}>❌</button>
  {:else}
    <span>{todo.text}</span>
    <button on:click={toggleStatus}>✔️</button>
  {/if}
  <button on:click={remove}>💥</button>
</li>
