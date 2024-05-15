<template>
  <div class="container min-h-screen mx-auto flex flex-col justify-between pt-">
    <div>
      <header class="px-2 py-3">
        <h1 class="text-4xl  py-3">Local River Monitoring Stations</h1>
        <div v-if="!permissionDenied" class="flex items-center">
          <p>Showing river data for
          </p>
          <p class="font-bold ml-1 text-3xl px-2 py-1 border-4 border-slate-200 rounded">{{ locationName }}</p>
        </div>

      </header>
      <div v-if="!permissionDenied" class="m-3">
        <button v-if="showButton" :disabled="buttonDisabled" class="bg-blue-500 text-white font-bold py-2 px-4 rounded"
          :class="{ 'animate-pulse': loadingState, 'hover:bg-blue-700': !loadingState }" @click="getGeoLocationData">{{
          loadBtnText
        }}</button>

        <div v-else class="bg-gray-700 text-white inline-block py-2 px-4 rounded">
          <p class=""> Here is the latest data for river monitoring stations within 5km of your current location in {{
            locationName
          }}
          </p>
        </div>
      </div>
      <div class="p-4">
        <div v-if="permissionDenied">
          <p>Unable to retrieve your location from the browser. Please enable location data in your browser and refresh
          </p>
        </div>
        <ul v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">

          <Card v-if="dataLoaded" v-for="river in data.data.items" :key="river.RLOIid" :river="river['@id']"
            :loadingState="loadingState" />

        </ul>
      </div>
    </div>
    <div>
    </div>
    <footer class="px-2 py-6">
      <p v-if="queryCoOrds">Using your browser defined location at {{ queryCoOrds }}</p>
      <p>
        This app uses Environment Agency flood and river level data from the
        real-time data API (Beta)
      </p>
    </footer>
  </div>
</template>

<script setup>
const loadBtnText = ref('Get data for your current location')
const queryLink = ref('')
const queryCoOrds = ref()

const data = ref('')
const dataLoaded = ref(true)
const buttonDisabled = ref(false)
const loadingGeoData = ref(false)
const locationName = ref('Sheffield')

const loadingState = ref(false)
const showButton = ref(true)
const permissionDenied = ref(false)



data.value = await useFetch('https://environment.data.gov.uk/flood-monitoring/id/stations?town=Sheffield')

async function success(position) {
  const latitude = position.coords.latitude;
  const longitude = position.coords.longitude;

  getLocationName(latitude, longitude)

  queryLink.value = `https://environment.data.gov.uk/flood-monitoring/id/stations?lat=${latitude}&long=${longitude}&dist=5&parameter=level`;
  queryCoOrds.value = `latitude: ${latitude} °, longitude: ${longitude} °`;
  data.value = await useFetch(queryLink)
  loadingState.value = false
  loadBtnText.value = `Located you near ${locationName.value}`;
  dataLoaded.value = true
  showButton.value = false
}

function error() {
  loadBtnText.value = "Unable to retrieve your location from the browser. Please enable location data in your browser and refresh";

  showButton.value = false
  permissionDenied.value = true
}

function getGeoLocationData() {
  loadingState.value = true
  buttonDisabled.value = true
  loadingGeoData.value = true
  if (!navigator.geolocation) {
    loadBtnText.value = "Geolocation is not supported by your browser";
    permissionDenied.value = true
  } else {
    loadBtnText.value = "Locating...";
    navigator.geolocation.getCurrentPosition(success, error);
  }
  loadingGeoData.value = false
}

const getLocationName = async (latitude, longitude) => {
  const { data: locationData } = await useFetch(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${latitude},${longitude}&key=${process.env.GOOGLEKEY}`)

  for (let ac = 0; ac < locationData.value.results[0].address_components.length; ac++) {
    let component = locationData.value.results[0].address_components[ac];

    switch (component.types[0]) {
      case 'neighborhood':
        locationName.value = component.long_name
        return;
      case 'locality':
        locationName.value = component.long_name
        return;
      case 'postal_town':
        locationName.value = component.long_name
        return;
      case 'locality':
        locationName.value = component.long_name
        return;
      case 'administrative_area_level_2':
        locationName.value = component.short_name
        return;
      case 'administrative_area_level_1':
        locationName.value = component.short_name
        return;
      case 'country':
        locationName.value = component.long_name
        return;
    }
  };
}



</script>
