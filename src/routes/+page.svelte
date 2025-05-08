<script>
	import Map from '$lib/Map.svelte';
	import Sidebar from '$lib/Sidebar.svelte';
	import { writable } from 'svelte/store';

	// This store will hold the selected filter ID (e.g., 'bid', 'cc', or '' for all)
	const selectedFilterIdStore = writable('');

	function handleFilterChange(event) {
		selectedFilterIdStore.set(event.detail);
	}
</script>

<div class="page-container">
	<Sidebar on:filterChange={handleFilterChange} selectedFilter={$selectedFilterIdStore} />
	<div class="map-wrapper">
		<Map currentFilterStore={selectedFilterIdStore} />
	</div>
</div>

<style>
	.page-container {
		display: flex;
		height: 100vh; /* Full viewport height */
		width: 100vw;  /* Full viewport width */
	}

	.map-wrapper {
		flex-grow: 1; /* Map wrapper takes remaining space */
		height: 100%; /* Fill height of its flex container */
		display: flex; /* Allows Map component with height:100% to fill it */
	}

	/* Apply some global styles to ensure full page layout */
	:global(html, body) {
		margin: 0;
		padding: 0;
		height: 100%;
		width: 100%;
		overflow: hidden; /* Prevent scrollbars on body if page-container manages all space */
	}

	/* SvelteKit's announcer can sometimes mess with 100vh layouts if not needed */
	:global(#svelte-announcer) {
		display: none;
	}
</style>