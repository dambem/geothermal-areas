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


  let deckContainer;
  let deck;
  let map;




  onMount(() => {

    function getColorForFeature(f) {
      if (f.properties.Geotherm_1 === "Pre-Permian, limestone, southern province") {
        return [255, 153, 51, 90]; 
      }
      else if (f.properties.GeologicHa === "Hydrothermal") {
        return [66, 134, 244, 90]; 
      }
      else if (f.properties.Geotherm_2 === "Intracratonic Basin") {
        return [76, 175, 80, 90]; 
      }
      return [90, 90, 250, 90];
    }
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
          stroke: true,
          getFillColor: getColorForFeature,
          getText: f => f.properties.Geotherm_1,

        })
      ],
      getTooltip: ({object}) => {
        if (!object) {
          return null;
        }
        return {
          html: `
            <div>
              <h3>${object.properties.Geothermal || object.properties.Geotherm_1 || 'Geothermal Area'}</h3>
              <p><strong>Age:</strong> ${object.properties.BGSAgeDesc || 'Unknown'}</p>
              <p><strong>Type:</strong> ${object.properties.GeologicHa || 'Unknown'}</p>
              <p><strong>Status:</strong> ${object.properties.Status || 'Unknown'}</p>
              <p><strong>Partner:</strong> ${object.properties.Partner1 || 'Unknown'}</p>
              <p><strong>Region:</strong> ${object.properties.Geographic || object.properties.Geograph_1 || 'Unknown'}</p>
            </div>
          `,
          style: {
            backgroundColor: 'white',
            fontSize: '12px',
            borderRadius: '4px',
            padding: '8px'
          }
        };
      }
    });
    map.addControl(deck)

    map.addControl(new maplibregl.NavigationControl())
  });

  });
</script>

<main>
  <p> please wait - loading </p>
  <div class="deck-container" bind:this={deckContainer}></div>
  <a href="https://webapps.bgs.ac.uk/services/ngdc/accessions/index.html#item185538" class='disclaimer'> Geothermal Sites - Contains data supplied by Natural Environment Research Council. Full Dataset Linked </a>
  <a class="disclaimer2" href='https://www.bemben.co.uk'> Geothermal Areas created by Damian Bemben </a>
</main>

<style>
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


  .deck-container {
    position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
  }


</style>
