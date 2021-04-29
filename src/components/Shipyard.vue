<template>
  <v-container>
    <v-row>
      <v-col 
        cols="12"
        class="text-center"
      >
        <h1 class="shipyard-title">
          SHIPYARD
        </h1>
      </v-col>
      <v-col cols="12">
        <div class="shipyard-header">
          SELECT VEHICLE
        </div>
        <v-container>
          <v-row class="text-center items-bg">
            <v-col
              v-for="item in items"
              :key="item.key"
              cols="2"
              class="item-container"
            >
              <v-tooltip
                color="#000"
                bottom
              >
                <template v-slot:activator="{ on, attrs }">
                  <div
                    v-bind="attrs"
                    v-on="on"
                    class="item-bg text-center"
                  >
                    <v-img
                      :src="'icons/' + item.key + '.png'"
                      :alt="item.key"
                      class="item-img"
                      max-height="60"
                      max-width="60"
                    />
                  </div>
                </template>
                <div>
                  {{ item.name + ((item.requireTech)?" (Tech)":"")}}
                </div>
                <div
                  v-html="costToString(item)"
                  class="text-center"
                />
              </v-tooltip>
            </v-col>
          </v-row>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import shipyardData from "./../data/warden/shipyard.json";

  export default {
    name: 'Shipyard',

    data: () => ({
      items: []
    }),
    created: function() {
      this.items = shipyardData
    },
    methods: {
      costToString: function(cost) {
        let response = ""
        response += (cost.bmat > 0)?(cost.bmat.toLocaleString('en')+"<img src=\"icons/bmat.png\" alt=\"bmat\" title=\"bmat\" class=\"mat-icon\"/>"):""
        response += (cost.emat > 0)?(cost.emat.toLocaleString('en')+"<img src=\"icons/emat.png\" alt=\"emat\" title=\"emat\" class=\"mat-icon\"/>"):""
        response += (cost.hemat > 0)?(cost.hemat.toLocaleString('en')+"<img src=\"icons/hemat.png\" alt=\"hemat\" title=\"hemat\" class=\"mat-icon\"/>"):""
        response += (cost.rmat > 0)?(cost.rmat.toLocaleString('en')+"<img src=\"icons/rmat.png\" alt=\"rmat\" title=\"rmat\" class=\"mat-icon\"/>"):""
        return response
      }
    }
  };
</script>

<style type="text/css">
  .shipyard-title {
    background-color: #6a6a6a;
  }
  .shipyard-header {
    padding-left: 15px;
    padding-top: 5px;
    padding-bottom: 5px;
    background-color: #6a6a6a;
  }
  .items-bg {
    background-color: #060606;
  }
  .item-bg {
    padding-top: 12px;
    padding-bottom: 12px;
    padding-right: 5px;
    padding-left: 5px;
    background-color: #464646;
    width: 100%;
    cursor: pointer;
  }
  .item-bg:hover {
    background-color: #646464;
  }
  .item-img {
    margin-left: auto;
    margin-right: auto
  }
  .mat-icon {
    max-height: 35px;
    padding-right: 15px;
    padding-left: 5px;
    margin-top: 2px;
  }
</style>
