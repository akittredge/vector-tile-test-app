<script>
  import { onMount } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import 'maplibre-gl/dist/maplibre-gl.css';
  import { PMTiles } from 'pmtiles';
  import { base } from '$app/paths'; // Import base path

  let mapContainer;
  let map;

  // Construct the PMTiles URL using the base path
  // The PMTiles file will be in the static folder, so it will be at the root of the deployed site,
  // or at the root of the base path if one is set.
  const pmtilesUrl = `${base}/sample.pmtiles`;

  onMount(() => {
    const protocol = new pmtiles.Protocol();
    maplibregl.addProtocol('pmtiles', protocol.tile);

    map = new maplibregl.Map({
      container: mapContainer,
      style: {
        version: 8,
        sources: {
          'sample-tiles': { // Unique source name
            type: 'vector',
            url: `pmtiles://${pmtilesUrl}`,
            attribution: 'Sample Data'
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
            id: 'points-layer',
            type: 'circle',
            source: 'sample-tiles',
            'source-layer': 'sample.geojson', // Tippecanoe defaults to the filename as the layer name
            filter: ['==', '$type', 'Point'],
            paint: {
              'circle-radius': 5,
              'circle-color': '#007cbf'
            }
          },
          {
            id: 'lines-layer',
            type: 'line',
            source: 'sample-tiles',
            'source-layer': 'sample.geojson',
            filter: ['==', '$type', 'LineString'],
            paint: {
              'line-color': '#ff9900',
              'line-width': 2
            }
          },
          {
            id: 'polygons-layer',
            type: 'fill',
            source: 'sample-tiles',
            'source-layer': 'sample.geojson',
            filter: ['==', '$type', 'Polygon'],
            paint: {
              'fill-color': '#3bb2d0',
              'fill-opacity': 0.5,
              'fill-outline-color': '#000000'
            }
          }
        ]
      },
      center: [0, 0],
      zoom: 1
    });

    map.on('load', () => {
      // Optional: Fit map to the bounds of the PMTiles data after it loads
      // This requires the PMTiles plugin to expose bounds, or you might need to define them manually
      // For simplicity, we'll skip this for now but it's a common enhancement.
    });

    return () => {
      map.remove();
      // Protocol cleanup consideration mentioned earlier still applies.
    };
  });
</script>

<div bind:this={mapContainer} style="width: 100%; height: 500px;"></div> 