<script>
  import { onMount } from 'svelte'
  import { sortBy } from 'lodash'
  import PouchDB from 'pouchdb-browser'

  import TodoItem from './todo-item.svelte'

  // Set up local PouchDB and continuous replication to remote CouchDB
  let db = new PouchDB('db')
  const replication = PouchDB.sync('db', 'http://localhost:5984/svelte-todo-db', {
    live: true,
    retry: true
  }).on('change', async function (info) {
    await updateTodos()
  }).on('error', function (err) {
    console.log('Replication error:', err)
  })

  // Set up our vars and defaults
  let newTodoText = ''
  let sortByWhat = 'createdAt'
  let filterByWhat = ''
  let isLoading = true
  // All the todos directly from the PouchDB. Sorting and filtering comes later
  let todos = []
  $: sortedAndFilteredTodos = sortBy(todos, [sortByWhat]).filter((todo) => {
    const [filterKey, filterValue] = filterByWhat.split(':')
    // Only filter if there’s a proper filter set
    return filterKey && filterValue ? todo[filterKey].toString() === filterValue : true
  })

  // Helper for reloading all todos from the local PouchDB. It’s on-device and has basically zero latency,
  // so we can use it quite liberally instead of keeping our local state up to date like you’d do
  // in a Redux reducer. It also saves us from having to rebuild the local state todos from the data we sent
  // to the database and the `_id` and `_rev` values that were sent back.
  async function updateTodos() {
    const allDocs = await db.allDocs({
      include_docs: true
    })
    todos = allDocs.rows.map(row => row.doc)
    isLoading = false
  }

  // Event handlers for adding, updating and removing todos
  async function add(event) {
    const newTodo = {
      text: newTodoText,
      complete: false,
      createdAt: new Date().toISOString()
    }
    const addition = await db.post(newTodo)
    if (addition.ok) {
      await updateTodos()
    }
    newTodoText = ''
  }

  async function updateStatus(event) {
    const { todo } = event.detail
    const update = await db.put(todo)
    if (update.ok) {
      await updateTodos()
    }
  }

  async function removeItem(event) {
    const { todo: todoToRemove } = event.detail
    const removal = await db.remove(todoToRemove)
    if (removal.ok) {
      // For removal, we can just update the local state instead of reloading everything from PouchDB,
      // since we no longer care about the doc’s revision.
      todos = todos.filter((todo) => {
        return todo._id !== todoToRemove._id
        })
    }
  }

  // Load todos on first run
  onMount(async () => {
    await updateTodos()
  })
</script>

<style>
  ul {
    list-style: none;
    padding: 0;
  }
  button {
    margin-left: 0.75em;
  }
  input[type='text'] {
    width: 440px;
  }
</style>

{#if isLoading}
  <h1>
    Loading your todos…
  </h1>
{:else}
  {#if todos.length === 0}
    <h1>
      Zero Todos! Nice ✊
    </h1>
  {:else}
    <h1>
      Showing {sortedAndFilteredTodos.length} of {todos.length} todos
    </h1>
  {/if}
{/if}

<div>
  <label for="ordre">Sort by:</label>
  <select bind:value={sortByWhat}>
    <option value='createdAt'>Time</option>
    <option value='text'>Todo text</option>
    <option value='complete'>Completion</option>
  </select>
</div>
<div>
  <label for="filtre">Filter:</label>
  <select bind:value={filterByWhat}>
    <option value=''>Show all todos</option>
    <option value='complete:true'>Show completed todos</option>
    <option value='complete:false'>Show open todos</option>
  </select>
</div>

<ul>
  {#each sortedAndFilteredTodos as todo (todo._id)}
    <TodoItem todo={todo} on:remove={removeItem} on:update={updateStatus}/>
  {/each}
</ul>

<form on:submit|preventDefault={add}>
  <input type='text' bind:value={newTodoText}>
  <button type='submit'>➕ Add new task</button>
</form>
