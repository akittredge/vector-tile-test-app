<script>
  import { onMount } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import 'maplibre-gl/dist/maplibre-gl.css';
  import { PMTiles, Protocol } from 'pmtiles';
  // import { base } from '$app/paths'; // No longer needed for external URL

  let mapContainer;
  let map;

  const pmtilesUrl = 'https://vector-tile-test-app.s3.us-east-1.amazonaws.com/all_boundaries.pmtiles';

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
            filter: ['==', '$type', 'Polygon'],
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

    map.on('load', () => {
      // You might want to fit map to the bounds of your new S3 PMTiles data
    });

    return () => {
      map.remove();
    };
  });
</script>

<div bind:this={mapContainer} style="width: 100%; height: 500px;"></div> 