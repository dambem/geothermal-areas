<script>
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'
  import { onMount } from 'svelte';
  import { Deck } from '@deck.gl/core';
  import {MapboxOverlay as DeckOverlay} from '@deck.gl/mapbox';
  import { GeoJsonLayer } from '@deck.gl/layers';
  import maplibregl from 'maplibre-gl';
  import 'maplibre-gl/dist/maplibre-gl.css';
  import geojsonData from './assets/geothermal_areas_v2.json';
  import Rainbow from 'rainbowvis.js';

  let deckContainer;
  let deck;
  let map;
  let typeOptions = [];
  let selectedTypes = [];

  let ageOptions = [];
  let selectedAge = [];

  const types = new Set();
  const age = new Set();
  var filteredData;

  function hexToRgba(hex, alpha = 90) {
    hex = hex.replace('#', '');
    if (hex.length === 3) {
      hex = hex[0] + hex[0] + hex[1] + hex[1] + hex[2] + hex[2];
    }
    // Parse the hex values
    const r = parseInt(hex.substring(0, 2), 16);
    const g = parseInt(hex.substring(2, 4), 16);
    const b = parseInt(hex.substring(4, 6), 16);
    // Return the RGBA array for deck.gl
    return [r, g, b, alpha];
  }
  function filterGeoJSON() {
      if (!geojsonData) return [];
      return {
        ...geojsonData,
        features: geojsonData.features.filter(feature => {
          const ageMatch = selectedAge.length === 0 ||
          selectedAge.includes(feature.properties.BGSAgeDesc)
          return ageMatch
        })
      }
    }
  function getColorForFeature(f) {
      var rainbow = new Rainbow()
      rainbow.setNumberRange(0, types.size)
      var geo_1 = f.properties.Geotherm_1
      return hexToRgba(rainbow.colorAt(typeOptions.indexOf(geo_1)), 90)
    }

  onMount(() => {
    geojsonData.features.forEach(feature => {
      if (feature.properties.Geotherm_1) types.add(feature.properties.Geotherm_1);
      if (feature.properties.BGSAgeDesc) age.add(feature.properties.BGSAgeDesc);

    })

    typeOptions = Array.from(types).filter(Boolean);
    selectedTypes = [...typeOptions];

    ageOptions = Array.from(age).filter(Boolean);
    selectedAge = [...ageOptions];
 
    const map = new maplibregl.Map({
      container: deckContainer,
      style: 'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
      center: [0.45, 51.47],
      zoom: 4,
      bearing: 0,
      pitch: 30
    });

    map.on('load', () => {
    deck = new DeckOverlay({
      interleaved: true,
      layers: [
        new GeoJsonLayer({
          id: 'geojson-plot',
          data: geojsonData,
          pickable: true,
          stroke: false,
          extruded: true,
          filled: true,
          opacity: 0.5,
          getLineColor: [255, 255, 255],
          getElevation: f => 2000+1500*Math.random(),
          getFillColor: getColorForFeature,
        })
      ],
      getTooltip: ({object}) => {
        if (!object) {
          return null;
        }
        return {
          html: `
            <div class='overlay'>
              <h3>${object.properties.Geotherm_1 || 'Geothermal Area'}</h3>
              <p><strong>Age:</strong> ${object.properties.BGSAgeDesc || 'Unknown'}</p>
              <p><strong>Type:</strong> ${object.properties.GeologicHa || 'Unknown'}</p>
              <p><strong>Status:</strong> ${object.properties.Status || 'Unknown'}</p>
              <p><strong>Plate Type:</strong> ${object.properties.PlateTectS || 'Unknown'}</p>

              <p><strong>Source:</strong> ${object.properties.References}</p>
            </div>
          `,
          style: {
            backgroundColor: 'white',
            fontSize: '12px',
            borderRadius: '10px',
            padding: '8px',
            boxShadow: '0 0 10px rgba(0, 0, 0, 0.2)',
          }
        };
      }
    });
    map.addControl(deck)

    map.addControl(new maplibregl.NavigationControl())
  });


  });


  function updateLayers() {
    deck.setProps({
      layers: [
        new GeoJsonLayer({
          id: 'geojson-plot',
          data: filteredData,
          pickable: true,
          stroke: false,
          extruded: true,
          filled: true,
          opacity: 0.5,
          getLineColor: [255, 255, 255],
          getElevation: f => 1000+500*Math.random(),
          getFillColor: getColorForFeature,
        })
      ]
    });
  }

  $: {
    console.log("age-check:", selectedAge)
    filteredData = filterGeoJSON();
    if (deck && filteredData) {
    updateLayers();
    }
  }
</script>

<main>
  <div class="filter-panel">
    <div class="filter-section">
      <h2>UK Geothermal Sites</h2>
      <p>A small demo using deck.gl to visualise a large geojson dataset of geothermal sites  within the UK</p>
      <hr>
      <h3>Age</h3>
      {#each ageOptions as age}
        <label class='input' >
          <input type="checkbox" bind:group={selectedAge} value={age} />
          {age}
        </label>
        <br>
      {/each}
      <hr>
      <a href="https://webapps.bgs.ac.uk/services/ngdc/accessions/index.html#item185538"> Geothermal Sites - Contains data supplied by Natural Environment Research Council. Full Dataset Linked </a>
      <hr>
      <a href='https://www.bemben.co.uk'> Geothermal Areas Map created by Damian Bemben </a>

    </div>
  </div>
  <div class="deck-container" bind:this={deckContainer}></div>
</main>

<style>
  .input{
    text-align: left;
  }
  .overlay {
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    border-radius: 4px;
  }
  .disclaimer {
    color:black;
    position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 90%;
  }

  .disclaimer2 {
    color:black;
    position: absolute;
      top: 90%;
      left: 0;
      right: 0;
      bottom: 10;
  }
  .filter-section {
    color:black;
  }
  .filter-panel {
    position: absolute;
    top: 2%;
    left: 1%;
    width: 400px;
    max-height: 80vh;
    overflow-y: auto;
    background-color: white;
    border-radius: 4px;
    padding: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    z-index: 10;
  }
  .deck-container {
    position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
  }


</style>
