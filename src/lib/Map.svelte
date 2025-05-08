<script>
  import { onMount, onDestroy } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import 'maplibre-gl/dist/maplibre-gl.css';
  import { PMTiles, Protocol } from 'pmtiles';
  // import { base } from '$app/paths'; // No longer needed for external URL

  let mapContainer;
  let map;

  export let currentFilterStore = null; // Expecting a Svelte readable store

  const pmtilesUrl = 'https://vector-tile-test-app.s3.us-east-1.amazonaws.com/all_boundaries.pmtiles';

  let currentFilterValue = '';
  let unsubscribeFilterStore = () => {};

  function updateMapFilter(filterId) {
    if (!map || !map.isStyleLoaded() || !map.getSource('s3-tiles')) return;

    if (filterId && filterId !== '') {
      // Assuming the property in your PMTiles features to filter by is named 'id'
      map.setFilter('polygons-layer', ['==', ['get', 'id'], filterId]);
    } else {
      map.setFilter('polygons-layer', null); // Remove filter to show all
    }
  }

  onMount(() => {
    const protocol = new Protocol();
    maplibregl.addProtocol('pmtiles', protocol.tile);

    map = new maplibregl.Map({
      container: mapContainer,
      style: {
        version: 8,
        sources: {
          's3-tiles': { // Unique source name
            type: 'vector',
            url: `pmtiles://${pmtilesUrl}`,
            attribution: 'Data from S3' // Update attribution as needed
          }
        },
        layers: [
          {
            id: 'background',
            type: 'background',
            paint: {
              'background-color': '#f0f0f0'
            }
          },
          {
            id: 'polygons-layer',
            type: 'fill',
            source: 's3-tiles', // Placeholder - update this!
            'source-layer': 'all_boundaries',
            // Initial filter can be null (show all) or set based on initial store value
            // filter: ['==', ['get', 'id'], initialFilterValue], // Handled by subscription
            paint: {
              'fill-color': '#3bb2d0',
              'fill-opacity': 0.5,
              'fill-outline-color': '#000000'
            }
          }
        ]
      },
      center: [(-74.66184938203348 + -72.97034397052578) / 2, (40.25252938803669 + 41.282818272331866) / 2], // Center of maxBounds
      zoom: 9, // Initial zoom
      minZoom: 9,
      maxZoom: 16,
      maxBounds: [
        [-74.66184938203348, 40.25252938803669],
        [-72.97034397052578, 41.282818272331866]
      ]
    });

    if (currentFilterStore) {
      // Initialize currentFilterValue from store in case it has a non-default initial value
      // $currentFilterStore can be used here if script runs after store is available, but direct access in onMount is safer.
      // For writable stores, you can get its value once using get(currentFilterStore) if you import { get } from 'svelte/store'
      // However, the subscription will set it and call updateMapFilter anyway.
      
      unsubscribeFilterStore = currentFilterStore.subscribe(value => {
        const needsUpdate = currentFilterValue !== value;
        currentFilterValue = value;
        // Only update if map is loaded and value changed, to prevent unnecessary calls on mount
        if (map && map.isStyleLoaded() && needsUpdate) {
          updateMapFilter(currentFilterValue);
        }
      });
    }

    map.on('load', () => {
      // Apply the filter once the map style is loaded, using the current value from the store
      updateMapFilter(currentFilterValue);
    });

    // No need for explicit map.remove() in onDestroy if not using HMR that causes issues.
    // Svelte handles component destruction. The protocol itself might be an edge case for HMR.
  });

  onDestroy(() => {
    unsubscribeFilterStore(); // Important to prevent memory leaks
    if (map) {
        // map.remove(); // Consider if map instance needs explicit removal for complex scenarios or HMR
    }
  });
</script>

<div bind:this={mapContainer} style="width: 100%; height: 100%;"></div> 