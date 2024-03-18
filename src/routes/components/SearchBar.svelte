<script lang="ts">
  /* global google */

  import { onMount } from 'svelte';
  import type { MdFilledTextField } from '@material/web/textfield/filled-text-field';

  export let location: google.maps.LatLng | undefined;
  export let placesLibrary: google.maps.PlacesLibrary;
  export let map: google.maps.Map;
  export let initialValue = '';
  export let zoom = 19;

  let textFieldElement: MdFilledTextField;

  onMount(async () => {
    await textFieldElement.updateComplete;
    const inputElement = textFieldElement.renderRoot.querySelector('input') as HTMLInputElement;
    
    // Initiate Autocomplete
    const autocomplete = new placesLibrary.Autocomplete(inputElement, {
      fields: ['formatted_address', 'geometry', 'name'],
    });

    // Listener for autocomplete selection
    autocomplete.addListener('place_changed', () => {
      const place = autocomplete.getPlace();
      if (place.geometry) {
        updateLocationFromPlace(place);
      } else {
        console.error('Place not found');
        textFieldElement.value = '';
      }
    });

    // Listener for map clicks
    map.addListener('click', (e) => {
      reverseGeocode(e.latLng);
    });
  });

  // Function to update the location from a place object
  function updateLocationFromPlace(place) {
    if (!place.geometry || !place.geometry.location) {
      textFieldElement.value = '';
      return;
    }
    map.setCenter(place.geometry.location);
    map.setZoom(zoom);
    location = place.geometry.location;
    textFieldElement.value = place.name || place.formatted_address || '';
  }

  // Function to reverse geocode a LatLng object
  async function reverseGeocode(latLng) {
    const geocoder = new google.maps.Geocoder();
    try {
      const results = await geocoder.geocode({ location: latLng });
      if (results.results[0]) {
        updateLocationFromPlace(results.results[0]);
      } else {
        textFieldElement.value = 'No address found';
      }
    } catch (error) {
      textFieldElement.value = 'Error finding address';
    }
  }
</script>

<md-filled-text-field bind:this={textFieldElement} label="Search an address" value={initialValue}>
  <md-icon slot="leadingicon">search</md-icon>
</md-filled-text-field>
