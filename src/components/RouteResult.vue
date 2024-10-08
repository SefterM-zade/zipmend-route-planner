<script>
  export default {
    props: ['distance', 'duration']
  }
</script>
<template>
  <div>
    <hr style="margin: 32px 0; border-color: #9E9E9E;" />
      <v-row>
        <v-col class="col-auto">
          <div class="result">
            <div class="d-flex mb-2">
              <img src="@/assets/distance.svg" />
              <p class="header">Distance</p>
            </div>
            <p class="value">{{ distance }}</p>
          </div>
        </v-col>
        <v-col col="auto">
          <div class="result">
            <div class="d-flex mb-2">
              <img src="@/assets/time.svg" />
              <p class="header">Driving time</p>
            </div>
            <p class="value">{{ duration }}</p>
          </div>
        </v-col>
      </v-row>
  </div>
</template>