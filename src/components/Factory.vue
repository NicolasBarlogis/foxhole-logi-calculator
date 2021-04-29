<template>
  <v-container>
    <v-row>
      <v-col 
        cols="12"
        class="text-center"
      >
        <h1 class="factory-title">
          FACTORY
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
          <div class="queue-count">
            GRAND TOTAL<br/>
            <span v-html="getQueuesCost()"/>
            <span class="align-right">
              {{ getQueuesDuration() }}
            </span>
          </div>
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
              <v-tooltip
                color="#000"
                top
                v-for="(item, i) in queues[categoryId]"
                :key="i"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-img
                    :src="'icons/' + item.key + '.png'"
                    :alt="item.key"
                    max-height="35"
                    max-width="35"
                    @click="removeFromQueue(categoryId, i)"
                    class="queue-item"
                    v-bind="attrs"
                    v-on="on"
                  />
                </template>
                {{ item.name }}
              </v-tooltip>
            </div>
            <div class="queue-count">
              <span v-html="getQueueCost(categoryId)"/>
              <span class="align-right">
                {{ getQueueDuration(categoryId) }}
              </span>
            </div>
          </v-col>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import factoryData from "./../data/warden/factory.json";

  export default {
    name: 'Factory',

    data: () => ({
      tab: null,
      dataByKey: {},
      dataByCategory: [],
      queues: []
    }),
    created: function() {
      let context = this
      let activeCategory =  0 
      this.dataByCategory[0] = {
        name: factoryData[0].category,
        items: []
      }
      this.$set(this.queues, 0, [])
      factoryData.forEach(function(item) {
        context.dataByKey[item.key] = item
        if(context.dataByCategory[activeCategory].name !== item.category) {
          activeCategory++
          context.dataByCategory[activeCategory] = {
            name: item.category,
            items: []
          }
          context.$set(context.queues, activeCategory, [])
        }
        context.dataByCategory[activeCategory].items.push(item)
      })
    },
    methods: {
      addToQueue: function(categoryId, itemKey) {
        this.queues[categoryId].push(this.dataByKey[itemKey])
      },
      removeFromQueue: function(categoryId, itemIndex) {
        this.queues[categoryId].splice(itemIndex,1)
      },
      getQueueCost: function(categoryId) {
        let cost = {
          bmat: 0,
          emat: 0,
          hemat: 0,
          rmat: 0
        }
        this.queues[categoryId].forEach(function(item) {
          cost.bmat += item.bmat
          cost.emat += item.emat
          cost.hemat += item.hemat
          cost.rmat += item.rmat
        })
        return this.costToString(cost)
      },
      getQueueDuration: function(categoryId) {
        let time = 0
        let hasNullTime = false
        this.queues[categoryId].forEach(function(item) {
          if(item.productionTime !== null) {
            time += item.productionTime
          } else {
            hasNullTime = true
          }
        })
        return (hasNullTime)?this.timeToString(time)+" +":this.timeToString(time)
      },
      getQueuesCost: function() {
        let cost = {
          bmat: 0,
          emat: 0,
          hemat: 0,
          rmat: 0
        }
        this.queues.forEach(function(queue) {
          queue.forEach(function(item) {
            cost.bmat += item.bmat
            cost.emat += item.emat
            cost.hemat += item.hemat
            cost.rmat += item.rmat
          })
        })
        let response = this.costToString(cost)
        return (response.length > 0)?response:"N/A"
      },
      getQueuesDuration: function() {
        let maxTime = 0
        let hasNullTime = false
        this.queues.forEach(function(queue) {
          let time = 0
          queue.forEach(function(item) {
            if(item.productionTime !== null) {
              time += item.productionTime
            } else {
              hasNullTime = true
            }
          })
          if(time > maxTime) {
            maxTime = time
          }
        })
        let response = (hasNullTime)?this.timeToString(maxTime)+" +":this.timeToString(maxTime)
        return (response.length > 0)?("MAX "+response):""
      },
      resetQueues: function() {
        let context = this
        this.queues.forEach(function(queue,queueId) {
          context.queues[queueId].splice(0, context.queues[queueId].length)
        })
      },
      costToString: function(cost) {
        let response = ""
        response += (cost.bmat > 0)?(cost.bmat.toLocaleString('en')+"<img src=\"icons/bmat.png\" alt=\"bmat\" title=\"bmat\" class=\"mat-icon\"/>"):""
        response += (cost.emat > 0)?(cost.emat.toLocaleString('en')+"<img src=\"icons/emat.png\" alt=\"emat\" title=\"emat\" class=\"mat-icon\"/>"):""
        response += (cost.hemat > 0)?(cost.hemat.toLocaleString('en')+"<img src=\"icons/hemat.png\" alt=\"hemat\" title=\"hemat\" class=\"mat-icon\"/>"):""
        response += (cost.rmat > 0)?(cost.rmat.toLocaleString('en')+"<img src=\"icons/rmat.png\" alt=\"rmat\" title=\"rmat\" class=\"mat-icon\"/>"):""
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
</style>
