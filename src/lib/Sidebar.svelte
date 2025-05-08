<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let filterOptions = [
    'bid', 'cc', 'cd', 'dsny', 'fb', 'hc', 'hd', 
    'ibz', 'nta', 'nycongress', 'pp', 'sa', 'sd', 'ss', 'zipcode'
  ];
  export let selectedFilter = ''; // Default to empty, meaning no filter / show all

  function handleChange(event) {
    selectedFilter = event.target.value;
    dispatch('filterChange', selectedFilter);
  }
</script>

<div class="sidebar-container">
  <h3>Filter by ID Type</h3>
  <select bind:value={selectedFilter} on:change={handleChange}>
    <option value="">Show All</option>
    {#each filterOptions as option}
      <option value={option}>{option.toUpperCase()}</option>
    {/each}
  </select>
</div>

<style>
  .sidebar-container {
    width: 220px; /* Adjust as needed */
    padding: 1rem;
    background-color: #f0f0f0;
    border-right: 1px solid #ccc;
    height: 100%; /* Fill height of its flex container */
    overflow-y: auto; /* Add scroll if content overflows */
    box-sizing: border-box;
  }
  h3 {
    margin-top: 0;
    font-size: 1.1em;
  }
  select {
    width: 100%;
    padding: 0.5em;
    border-radius: 4px;
    border: 1px solid #ccc;
    box-sizing: border-box;
  }
</style> 