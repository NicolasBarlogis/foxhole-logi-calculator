<template>
  <v-container>
    <v-row>
      <v-col 
        cols="12"
        class="text-center"
      >
        <h1 class="factory-title">
          MASS PRODUCTION FACTORY
        </h1>
      </v-col>
      <v-col cols="12" md="6">
        <div class="factory-header">
          ITEMS
        </div>
        <v-tabs
          v-model="tab"
          background-color="#363636"
          centered
          dark
          icons-and-text
          color="#e57541"
        >
          <v-tabs-slider></v-tabs-slider>
          <v-tab
            v-for="(category, categoryId) in dataByCategory"
            :key="category.name"
          >
            <v-img
              :src="((tab == categoryId)?'icons/' + category.name + '-active.png':'icons/' + category.name + '.png')"
              :alt="category.name"
              max-height="40"
              max-width="40"
            />
          </v-tab>
        </v-tabs>

        <v-tabs-items v-model="tab">
          <v-tab-item
            v-for="(category,categoryId) in dataByCategory"
            :key="category.name"
            class="items-bg"
          >
            <v-container>
              <v-row class="text-center">
                <v-col
                  v-for="item in category.items"
                  :key="item.key"
                  cols="2"
                  class="item-container"
                  @click="addToQueue(categoryId, item.key)"
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
          </v-tab-item>
        </v-tabs-items>
      </v-col>
      <v-col cols="12" md="6">
        <div class="factory-header">
          <span>
            QUEUE
          </span>
          <span class="align-right">
            <a
              class="orange--text mr-4"
              @click="resetQueues()"
            >
              <i>RESET</i>
            </a>
          </span>
        </div>
        <v-container class="queue-body">
          <v-col cols="12"
            v-for="(category, categoryId) in dataByCategory"
            :key="category.name"
          >
            <div class="queue-items">
              <v-img
                :src="'icons/' + category.name + '.png'"
                :alt="category.name"
                max-height="45"
                max-width="45"
                class="queue-category"
              />
              <div
                v-if="queues[categoryId].quantity > 0"
                class="queue-container"
              >
                <v-tooltip
                  color="#000"
                  top
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-img
                      :src="'icons/' + queues[categoryId].item.key + '.png'"
                      :alt="queues[categoryId].item.key"
                      max-height="35"
                      max-width="35"
                      @click="removeFromQueue(categoryId)"
                      class="queue-item"
                      v-bind="attrs"
                      v-on="on"
                    />
                  </template>
                  {{ queues[categoryId].item.name }}
                </v-tooltip>
                <h3 class="vertical-center">
                  x  {{ queues[categoryId].quantity }}
                  <span v-if="['vehicle','shippable'].includes(queues[categoryId].item.category)"> (x 3 per crate)</span>
                </h3>
              </div>
            </div>
            <div class="queue-count">
              <span v-html="getQueueCost(categoryId)"/>
            </div>
          </v-col>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import dataFactory from "./../data/warden/factory.json";
  import dataGarage from "./../data/warden/garage.json";
  import dataShipyard from "./../data/warden/shipyard.json";
  import dataMPF from "./../data/warden/mass-production-factory.json";
  import dataConstructionYard from "./../data/warden/construction-yard.json";

  export default {
    name: 'MassProductionFactory',

    data: () => ({
      tab: null,
      rawData: [],
      dataByKey: {},
      dataByCategory: [],
      queues: []
    }),
    created: function() {
      let context = this
      // assemble data for MPF
      let factoryAuthorizedCategory = ["basic","explosive"]
      dataFactory.forEach(function(item) {
        if(factoryAuthorizedCategory.includes(item.category)) {
          context.rawData.push(item)
        }
      })
      this.rawData.push(...dataGarage)
      this.rawData.push(...dataShipyard)
      this.rawData.push(...dataMPF)
      this.rawData.push(...dataConstructionYard)

      // init queues and sorted data
      let activeCategory =  0 
      this.dataByCategory[0] = {
        name: this.rawData[0].category,
        items: []
      }      
      this.$set(this.queues, 0, { "item" : null, "quantity" : 0 })
      this.rawData.forEach(function(item) {
        context.dataByKey[item.key] = item
        if(context.dataByCategory[activeCategory].name !== item.category) {
          activeCategory++
          context.dataByCategory[activeCategory] = {
            name: item.category,
            items: []
          }
          context.$set(context.queues, activeCategory, { "item" : null, "quantity" : 0 })
        }
        context.dataByCategory[activeCategory].items.push(item)
      })
      console.log(this.dataByKey)
      console.log(this.dataByCategory)
      console.log(this.queues)
    },
    methods: {
      addToQueue: function(categoryId, itemKey) {
        if(this.queues[categoryId].quantity === 0 || this.queues[categoryId].item.key === itemKey) {
          this.queues[categoryId].item = this.dataByKey[itemKey]
          this.queues[categoryId].quantity++
        }
      },
      removeFromQueue: function(categoryId) {
        if(this.queues[categoryId].quantity > 0) {
          this.queues[categoryId].quantity--
        }
      },
      getQueueCost: function(categoryId) {
        let cost = {
          bmat: 0,
          emat: 0,
          hemat: 0,
          rmat: 0
        }
        let discount = 0
        for(let i = 1 ; i <= this.queues[categoryId].quantity ; i++) {
          // max 50% off
          let costLeft = 1 - (0.1 * Math.min(i, 5))
          discount += 0.1 * Math.min(i, 5)
          // vehicles and shippables by crates of 3
          if(["vehicle","shippable"].includes(this.queues[categoryId].item.category)) {
            cost.bmat += (this.queues[categoryId].item.bmat * 3) * costLeft
            cost.emat += (this.queues[categoryId].item.emat * 3) * costLeft
            cost.hemat += (this.queues[categoryId].item.hemat * 3) * costLeft
            cost.rmat += (this.queues[categoryId].item.rmat * 3) * costLeft
          } else {
            cost.bmat += this.queues[categoryId].item.bmat * costLeft
            cost.emat += this.queues[categoryId].item.emat * costLeft
            cost.hemat += this.queues[categoryId].item.hemat * costLeft
            cost.rmat += this.queues[categoryId].item.rmat * costLeft
          }
        }
        let response = this.costToString(cost)
        return (response.length>0)?(response+ " ("+(discount/this.queues[categoryId].quantity*100).toFixed(1)+"% discount)"):""
      },
      getQueueDuration: function(categoryId) {
        return categoryId
      },
      resetQueues: function() {
        let context = this
        this.queues.forEach(function(queue,queueId) {
          context.queues[queueId].quantity = 0
        })
      },
      costToString: function(cost) {
        let response = ""
        response += (cost.bmat > 0)?(Math.ceil(cost.bmat).toLocaleString('en')+"<img src=\"icons/bmat.png\" alt=\"bmat\" title=\"bmat\" class=\"mat-icon\"/>"):""
        response += (cost.emat > 0)?(Math.ceil(cost.emat).toLocaleString('en')+"<img src=\"icons/emat.png\" alt=\"emat\" title=\"emat\" class=\"mat-icon\"/>"):""
        response += (cost.hemat > 0)?(Math.ceil(cost.hemat).toLocaleString('en')+"<img src=\"icons/hemat.png\" alt=\"hemat\" title=\"hemat\" class=\"mat-icon\"/>"):""
        response += (cost.rmat > 0)?(Math.ceil(cost.rmat).toLocaleString('en')+"<img src=\"icons/rmat.png\" alt=\"rmat\" title=\"rmat\" class=\"mat-icon\"/>"):""
        return response
      },
      timeToString: function(time) {
        let minutes = Math.floor(time / 60)
        let secondes = time % 60
        let response = ""
        response += (minutes > 0)?(""+minutes+"min "):""
        response += (secondes > 0)?(""+secondes+"s"):""
        return response
      }
    }
  };
</script>

<style type="text/css">
  .factory-title {
    background-color: #6a6a6a;
  }
  .factory-header {
    padding-left: 15px;
    padding-top: 5px;
    padding-bottom: 5px;
    background-color: #6a6a6a;
  }
  .items-bg {
    background-color: #060606;
  }
  .item-container {
    padding: 2px;
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
  .queue-body {
    background-color: #1c1c1c;
  }
  .queue-items:after {
    content: "";
    clear: both;
    display: table;
  }
  .queue-category {
    float: left;
    width: 45px;
    margin-right: 15px;
    margin-bottom: 5px;
  }
  .queue-item {
    float: left;
    width: 35px;
    margin: 2px;
    cursor: pointer;
    z-index:999!important;
  }  
  .queue-count {
    background-color: #464646;
    min-height: 20px;
    padding-top: 4px;
    padding-bottom: 4px;
    padding-left: 15px;
    padding-right: 15px;
  }
  .align-right {
    float: right
  }
  .mat-icon {
    max-height: 35px;
    padding-right: 15px;
    padding-left: 5px;
    margin-top: 2px;
  }
  .vertical-center {
    -ms-transform: translateY(25%);
    transform: translateY(25%);
  }
</style>
