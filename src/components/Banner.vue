<template>
  <v-container>
    <v-row>
      <v-col>
        <v-dialog
          v-model="aboutDialog"
          max-width="1200"
        >
          <template v-slot:activator="{ on, attrs }">
            <span
              id="about-button"
              class="grey--text lighten-4 text-decoration-underline"
              v-bind="attrs"
              v-on="on"
            >
              About
            </span>
          </template>
          <v-card>
            <v-card-title class="headline grey darken-2 white--text">
              About this website
            </v-card-title>
            <v-card-text class="grey darken-4 white--text">
              <p v-html="readme" class="pt-4"/>
            </v-card-text>
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>
    <v-row class="text-center mt-5">
      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3">
          Foxhole Logi Calculator
        </h1>
        <p class="subheading font-weight-regular">
          A handy tool to calculate your logistic run cost & time
        </p>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import readmeMd from "./../../README.md"
  export default {
    name: 'Banner',
    data: () => ({
      aboutDialog: false,
      readme: ""
    }),
    created: function() {
      console.log(readmeMd)
      this.readme = readmeMd.html
    }
  }
</script>

<style type="text/css">
  #about-button {
    float: right;
    cursor: pointer;
  }
  h2 {
    padding-top: 25px;
  }
</style>
