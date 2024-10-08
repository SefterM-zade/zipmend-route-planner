<script>
  import RouteSearch from './RouteSearch.vue';
  import axios from 'axios';
  import RouteResult from './RouteResult.vue';
  import RouteMap from './RouteMap.vue';
  export default {
    components: {RouteSearch, RouteResult, RouteMap},
    data() {
      return {
        errorMessage: null,
        avoidCountries: [],
        loading: false,
        
        duration: "",
        distance: "",

        showResult: false,
        countryListWithCode: [
          {
            text: 'DE',
            value: 74
          },
          {
            text: 'CH',
            value: 193
          },
          {
            text: 'AT',
            value: 11
          },
          {
            text: 'NL',
            value: 200
          },
          {
            text: 'BE',
            value: 17
          },
          {
            text: 'FR',
            value: 70
          },
          {
            text: 'ES',
            value: 187
          },
          {
            text: 'PL',
            value: 159
          }
        ],
        loadingIndex: 1,
        loadingSpots: [{
          value: null, 
          index: 0
        }],

        unloadingIndex: 1,
        unloadingSpots: [{
          value: null, 
          index: 0
        }],
        
        routeCoords: [],
        center: [52.52000659999999, 13.404954],
      }
    },
    computed: {
      disableBtn() {

        for (let i = 0; i < this.loadingSpots.length; i++) {
          if (!this.loadingSpots[i].value) {  
            return true
          }
        }
        
        for (let i = 0; i < this.unloadingSpots.length; i++) {
          if (!this.unloadingSpots[i].value) {
            return true
          }
        }

        return false
      },
    },
    methods: {
      formatDuration(seconds) {
        const hours = Math.floor(seconds / 3600);
        const minutes = Math.floor((seconds % 3600) / 60);
        
        let result = '';
        
        if (hours > 0) {
            result += `${hours} h`;
        }
        
        if (minutes > 0) {
            if (result) result += ' ';
            result += `${minutes} min`;
        }

        return result || '0 min'; 
      },
      decodePolyline(encoded) {
        let coordinates = [];
        let index = 0;
        let lat = 0;
        let lng = 0;

        while (index < encoded.length) {
          let b, shift = 0, result = 0;
          do {
            b = encoded.charCodeAt(index++) - 63; // 63 to adjust ASCII code
            result |= (b & 0x1f) << shift; // Use bitwise operations to decode
            shift += 5;
          } while (b >= 0x20);

          const dlat = ((result & 1) ? ~(result >> 1) : (result >> 1));
          lat += dlat;

          shift = 0;
          result = 0;
          do {
            b = encoded.charCodeAt(index++) - 63;
            result |= (b & 0x1f) << shift;
            shift += 5;
          } while (b >= 0x20);

          const dlng = ((result & 1) ? ~(result >> 1) : (result >> 1));
          lng += dlng;

          coordinates.push([lat / 1e5, lng / 1e5]); // Divide by 1e5 to get correct precision
        }

        return coordinates;
      },
      resetResult() {
        if (this.showResult) {
          this.showResult = false
        }

        if (this.routeCoords.length) this.routeCoords = []

        if (this.errorMessage) this.errorMessage = null
      },
      setLoadingSpot(index, address) {
        this.loadingSpots[index].value = address
        
        if (address) this.center = [address.lat, address.lng]
        else this.center = [52.52000659999999, 13.404954]

        this.resetResult()
      },
      setUnloadingSpot(index, address) {
        this.unloadingSpots[index].value = address
        
        if (address) this.center = [address.lat, address.lng]
        else this.center = [52.52000659999999, 13.404954]

        this.resetResult()
      },  
      addUnloadingSpot() {
        this.resetResult()
        this.unloadingSpots.push({value: null, index: this.unloadingIndex ++})
      },
      removeUnloadingSpot(unloadingSpot) {
        this.resetResult()
        this.unloadingSpots = this.unloadingSpots.filter(u => u.index !== unloadingSpot.index)
      },
      addLoadingSpot() {
        this.resetResult()
        this.loadingSpots.push({value: null, index: this.loadingIndex ++})
      },
      removeLoadingSpot(loadingSpot) {
        this.resetResult()

        this.loadingSpots = this.loadingSpots.filter(l => l.index !== loadingSpot.index)
      },
      setResultDetails(summary, geometry) {
        this.distance = `${(summary.distance / 1000).toFixed(2)} km`
        this.duration = this.formatDuration(summary.duration)

        this.routeCoords = this.decodePolyline(geometry)
      },
      calculateRoute() {
        this.resetResult()
        this.loading = true

        const tempCoordinatesOfLoadingSpots = this.loadingSpots.map(spot => [spot.value.lng, spot.value.lat])
        const tempCoordinatesOfUnloadingSpots = this.unloadingSpots.map(spot => [spot.value.lng, spot.value.lat])

        const coordinates = [...tempCoordinatesOfLoadingSpots, ...tempCoordinatesOfUnloadingSpots]

        const options = {
          avoid_countries: this.avoidCountries.length > 0 ? this.avoidCountries: []
        }
        const requestBody = {
          coordinates,
          options,
        }

        axios.post('https://maps.zipmend.com/ors/v2/directions/driving-hgv', requestBody)
          .then(res => {          
            this.loading = false  
            this.setResultDetails(res.data.routes[0].summary, res.data.routes[0].geometry)
            this.showResult = true
          })
          .catch(err => {
            this.loading = false  
            this.errorMessage = err.response.data.error.message
          })

      }
    }
  }
</script>
<template>
  <v-row class="flex-column flex-md-row">
    <v-col class="col-auto form-details" style="max-width: 600px">
      <h2 class="title">Route configuration</h2>
      <v-row>
        <v-col>
          <h3 class="section-title">Loading</h3>
        </v-col>
        <v-col class="d-flex">
          <v-spacer></v-spacer>
          <span class="add-btn" @click="addLoadingSpot">
            Add loading spot
          </span>
        </v-col>
      </v-row>
      <div v-for="(loadingSpot, i) in loadingSpots" :key="loadingSpot.index">
        <RouteSearch :key="loadingSpot.index" @change="address => setLoadingSpot(i, address)" :hasDeleteBtn="i > 0">
          <img src="@/assets/x.svg" alt="delete" v-if="i > 0" @click="removeLoadingSpot(loadingSpot)" style="cursor: pointer;" class="mt-4">
        </RouteSearch>
      </div>
      <v-row>
        <v-col>
          <h3 class="section-title">Unloading</h3>
        </v-col>
        <v-col class="d-flex">
          <v-spacer></v-spacer>
          <span @click="addUnloadingSpot" class="add-btn">
            Add unloading spot
          </span>
        </v-col>
      </v-row>
      <div v-for="(unloadingSpot, i) in unloadingSpots" :key="'unload' + unloadingSpot.index">
        <RouteSearch :key="'unload' + unloadingSpot.index" @change="address => setUnloadingSpot(i, address)" :hasDeleteBtn="i > 0">
          <img src="@/assets/x.svg" alt="delete" v-if="i > 0" @click="removeUnloadingSpot(unloadingSpot)" style="cursor: pointer;" class="mt-4">
        </RouteSearch>
      </div>
      <v-row>
        <v-col>
          <h3 class="section-title mb-4">More features</h3>
          <v-select
            :items="countryListWithCode"
            label="Avoid countries"
            outlined
            v-model="avoidCountries"
            multiple
            clearable
          >
          </v-select>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-btn @click="calculateRoute" width="100%" class="calculate-btn" :disabled="disableBtn" :loading="loading">
            Calculate Route
          </v-btn>
        </v-col>
      </v-row>
      <RouteResult v-if="showResult" :duration="duration" :distance="distance"/>
      <v-alert
        outlined
        type="warning"
        v-if="errorMessage"
        class="mt-8"
      >
        {{ errorMessage }}
      </v-alert>
    </v-col>
    <v-col>
      <RouteMap 
        :center="center"
        :routeCoords="routeCoords"
        :loadingSpots="loadingSpots"
        :unloadingSpots="unloadingSpots"
      />
    </v-col>
  </v-row>
</template>
<style>
  .add-btn {
    text-decoration: underline;
    font-family: "Lato", sans-serif;
    cursor: pointer;
    font-weight: 700;
    color: #121231;
  }

  .section-title {
    font-weight: 700;
    font-family: "Lato", sans-serif;
    color: #121231;
    font-size: 20px;
  }
  .title {
    font-weight: 700;
    font-family: "Lato", sans-serif;
    color: #121231;
    font-size: 24px;
    margin-bottom: 32px;
  }

  .result {
    border-radius: 8px;
    border: 1px solid#9E9E9E;
    padding: 16px;
    width: 252px;
    height: 97px;
  }

  .result img {
    margin-right: 8px;
  }

  .result .header {
    font-family: "Lato", sans-serif;
    font-size: 16px;
    font-weight: 400;
    line-height: 19.2px;
    letter-spacing: 0;
    color: #121231;
    margin-bottom: 0;
  }

  .result .value {
    font-family: "Lato", sans-serif;
    font-size: 32px;
    font-weight: 400;
    line-height: 38.4px;
    text-align: left;
    margin-bottom: 0;
    color: #121231;
  }

  .form-details {
    max-height: 88vh;
    overflow-y: auto;
  }

  .theme--light.v-btn.calculate-btn {
    background-color: #E50043;
    color: white;
    text-transform: none;
    font-size: 16px;
    font-family: "Lato", sans-serif;
    letter-spacing: 0;
    font-weight: 700;
  }
</style>