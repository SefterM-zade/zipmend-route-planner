<script>
  import { loadScript } from "vue-plugin-load-script";
  import axios from 'axios';
  export default {
    props: {
      hasDeleteBtn: {
        default: false
      }
    },
    data() {
      return {
        autocompleteService: null,
        countryList: [
          'DE',
          'CH',
          'AT',
          'NL',
          'BE',
          'FR',
          'ES',
          'PL'
        ],
        country: 'DE',
        loading: false,
        placeId: null,
        addressSuggestions: [],
        search: null
      }
    },
    mounted() {
      this.loadGoogleMapsAPI()
    },
    watch: {
      search(value) {
        console.log('inside watch')
        if (value) {
          this.loading = false
        }
        this.searchPlaces(value)
      }
    },
    methods: {
      resetAddress() {
        this.placeId = null
        this.addressSuggestions = []
        this.$emit('change', null)
      },
      getSuggestions() {
        this.loading = true
        if (this.placeId) {
          axios.get(`https://maps.googleapis.com/maps/api/geocode/json?place_id=${this.placeId}&key=${process.env.VUE_APP_API_KEY}`)
            .then(res => {
              this.loading = false
              const coordinates = res.data.results[0].geometry.location
              this.$emit('change', coordinates)
            })
            .catch(err => {
              console.log(err)
              this.loading = false
            })
        }
        else {
          this.$emit('change', null)
          this.loading = false
        }
      },
      loadGoogleMapsAPI() {
        loadScript('https://maps.googleapis.com/maps/api/js?key=AIzaSyDcuSoUhAyNDtR-7FzCaX94iGRTK4oV2Ms&libraries=places')
          .then(() => {
            this.autocompleteService = new google.maps.places.AutocompleteService()
          })
          .catch((error) => {
            console.error("Failed to load Google Maps API", error);
          });
      },
      searchPlaces(address) {
        console.log(address)
        const options = {
          input: address,
          componentRestrictions: { country: this.country }
        }

        this.autocompleteService.getPlacePredictions(
          options,
          (predictions, status) => {
            this.loading = false

            console.log('predictions', predictions)
            console.log('status', status)
            if (status !== google.maps.places.PlacesServiceStatus.OK) {
              this.addressSuggestions = []
              return
            }

            this.addressSuggestions = predictions
          }
        )
      },
    }
  }
</script>
<template>
  <v-row class="flex-col">
    <v-col class="col-md-auto col-3">
      <v-select
        :items="countryList"
        label="Country"
        outlined
        v-model="country"
        @change="resetAddress"
        style="width: 100px"
      >
      </v-select>
    </v-col>
    <v-col class="col-md-auto col">
      <v-autocomplete
        :loading="loading"
        outlined
        clearable
        append-icon=""
        label="Address"
        v-model="placeId"
        :items="addressSuggestions"
        item-text="description"
        :search-input.sync="search"
        @input="getSuggestions"
        item-value="place_id"
        :class="[hasDeleteBtn ? 'w-357': 'w-400']"
      >
      </v-autocomplete>
    </v-col>
    <v-col v-if="hasDeleteBtn">
      <slot></slot>
    </v-col>
  </v-row>
</template>
<style>
  .w-400 {
    width: 400px;
  }

  .w-357 {
    width: 357px;
  }
</style>