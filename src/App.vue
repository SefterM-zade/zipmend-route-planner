<template>
  <v-app>
    <v-app-bar
      app
      color="white"
      dark
      style="z-index: 10000;"
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Name"
          class="shrink mt-1"
          contain
          min-width="100"
          src="@/assets/zipmend-logo.svg"
          width="100"
        />
      </div>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <router-view/>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    //
  }),
};
</script>