<template>
  <v-container>
    <v-row>
      <v-col 
        cols="12"
        class="text-center"
      >
        <h1 class="factory-title">
          FACTORY - SPREADSHEET MODE
        </h1>
        <span class="align-right mt-2">
          <a
            class="orange--text mr-4"
            @click="resetQuantities()"
          >
            <i>RESET</i>
          </a>
        </span>
      </v-col>
      <v-col cols="12">
        <v-data-table
          :headers="headers"
          :items="items"
          item-key="key"
          dark
          :items-per-page="200"
          hide-default-footer
        >
          <template v-slot:header.bmat="{}">
            B-MATS<img src="icons/bmat.png" alt="bmat" title="bmat" class="mat-icon"/>
          </template>
          <template v-slot:header.emat="{}">
            E-MATS<img src="icons/emat.png" alt="emat" title="emat" class="mat-icon"/>
          </template>
          <template v-slot:header.hemat="{}">
            HE-MATS<img src="icons/hemat.png" alt="hemat" title="hemat" class="mat-icon"/>
          </template>
          <template v-slot:header.rmat="{}">
            R-MATS<img src="icons/rmat.png" alt="rmat" title="rmat" class="mat-icon"/>
          </template>
          <template v-slot:item.name="{ item }">
            <div v-if="item.isCategory">
              <h2 class="text-uppercase mb-5">
                {{ categoriesName[item.name] }}
              </h2>
            </div>
            <v-row
              v-else
            >
              <v-col
                cols="2"
                class="mt-4 mb-1"
              >
                 <v-tooltip
                  color="#000"
                  bottom
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-img
                      v-bind="attrs"
                      v-on="on"
                      :src="'icons/' + item.key + '.png'"
                      :alt="item.key"
                      max-height="30"
                      max-width="30"
                    />
                  </template>
                  <div>
                    {{ item.name }}
                    <b class="orange--text font-weight-black">
                      x{{ item.perCrate }}
                    </b>
                    <span class="red--text">
                      {{ ((item.requireTech)?" (Tech)":"") }}
                    </span>
                  </div>
                  <div
                    v-html="costToString(item)"
                    class="text-center"
                  />
                  <div
                    v-html="ammoToString(item)"
                    class="text-center"
                  />
                </v-tooltip>
              </v-col>
              <v-col class="mt-1 mb-1">
                <v-tooltip
                  color="#000"
                  bottom
                >
                  <template v-slot:activator="{ on, attrs }">
                    <span
                      v-bind="attrs"
                      v-on="on"
                    >
                      {{ item.name }}
                    </span>
                    <div v-html="costToString(item)"/>
                  </template>
                  <div>
                    {{ item.name }}
                    <b class="orange--text font-weight-black">
                      x{{ item.perCrate }}
                    </b>
                    <span class="red--text">
                      {{ ((item.requireTech)?" (Tech)":"") }}
                    </span>
                  </div>
                  <div
                    v-html="costToString(item)"
                    class="text-center"
                  />
                  <div
                    v-html="ammoToString(item)"
                    class="text-center"
                  />
                </v-tooltip>
              </v-col>
            </v-row>
          </template>
          <template v-slot:item.quantity="{ item }">
            <v-text-field
              v-if="!item.isCategory"
              :value="quantities[item.id]"
              @input="updateQuantity($event, item)"
              class="short-input"
              :clearable="true"
              type="number"
              min="0"
              hide-details
              single-line
            />
            <div v-else>
              TOTAL
            </div>
          </template>
          <template v-slot:item.bmat="{ item }">
            <div v-if="item.bmat > 0">
              {{ (item.bmat * quantities[item.id]).toLocaleString() }}
              <img src="icons/bmat.png" alt="bmat" title="bmat" class="mat-icon"/>
            </div>
            <div v-else-if="item.isCategory">
              {{ getCategoryTotal(item,'bmat').toLocaleString() }}
              <img src="icons/bmat.png" alt="bmat" title="bmat" class="mat-icon"/>
            </div>
          </template>
          <template v-slot:item.emat="{ item }">
            <div v-if="item.emat > 0">
              {{ (item.emat * quantities[item.id]).toLocaleString() }}
              <img src="icons/emat.png" alt="emat" title="emat" class="mat-icon"/>
            </div>
            <div v-else-if="item.isCategory">
              {{ getCategoryTotal(item,'emat').toLocaleString() }}
              <img src="icons/bmat.png" alt="bmat" title="bmat" class="mat-icon"/>
            </div>
          </template>
          <template v-slot:item.hemat="{ item }">
            <div v-if="item.hemat > 0">
              {{ (item.hemat * quantities[item.id]).toLocaleString() }}
              <img src="icons/hemat.png" alt="hemat" title="hemat" class="mat-icon"/>
            </div>
            <div v-else-if="item.isCategory">
              {{ getCategoryTotal(item,'hemat').toLocaleString() }}
              <img src="icons/bmat.png" alt="bmat" title="bmat" class="mat-icon"/>
            </div>
          </template>
          <template v-slot:item.rmat="{ item }">
            <div v-if="item.rmat > 0">
              {{ (item.rmat * quantities[item.id]).toLocaleString() }}
              <img src="icons/rmat.png" alt="rmat" title="rmat" class="mat-icon"/>
            </div>
            <div v-else-if="item.isCategory">
              {{ getCategoryTotal(item,'rmat').toLocaleString() }}
              <img src="icons/bmat.png" alt="bmat" title="bmat" class="mat-icon"/>
            </div>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import factoryData from "./../data/warden/factory.json";

  export default {
    name: 'FactoryAdvanced',

    data: () => ({
      tab: null,
      dataByKey: {},
      dataByCategory: [],
      items: [],
      quantities: [],
      headers: [
        {text: 'ITEM', value: 'name'},
        {text: 'QUANTITY', value: 'quantity'},
        {value: 'bmat'},
        {value: 'emat'},
        {value: 'hemat'},
        {value: 'rmat'},
      ],
      queues: [],
      categoriesName: {
        'basic': 'light weapons',
        'explosive': 'heavy weapons',
        'tool': 'utiilities',
        'heal': 'medical supplies',
        'garrison': 'supplies'
      }
    }),
    created: function() {
      let context = this
      let activeCategory =  0 
      this.dataByCategory[0] = {
        name: factoryData[0].category,
        items: []
      }
      this.items.push({ name: factoryData[0].category, key: factoryData[0].category, isCategory: true, id: 0 })
      this.$set(this.queues, 0, [])
      factoryData.forEach(function(item) {
        context.dataByKey[item.key] = item
        item.id = context.items.length
        context.quantities[item.id] = 0
        if(context.dataByCategory[activeCategory].name !== item.category) {
          activeCategory++
          context.items.push({ name: item.category, key: item.category, isCategory: true, id: activeCategory })
          context.dataByCategory[activeCategory] = {
            name: item.category,
            items: []
          }
          context.$set(context.queues, activeCategory, [])
        }
        context.items.push(item)
        context.dataByCategory[activeCategory].items.push(item)
      })
    },
    methods: {
      updateQuantity: function(newVal, item) {
        if(Number.isFinite(parseInt(newVal))) {
          this.$set(this.quantities,item.id,parseInt(newVal))
        } else {
          this.$set(this.quantities,item.id,0)
        }
      },
      getCategoryTotal: function(category, field) {
        let total = 0
        this.dataByCategory[category.id].items.forEach(item => {
          total += item[field] * this.quantities[item.id]
        })
        return total
      },
      resetQuantities: function() {
        for(let i = 0; i < this.quantities.length; i++) {
          this.$set(this.quantities,i,0)
        }
      },
      costToString: function(cost) {
        let response = ""
        response += (cost.bmat > 0)?(cost.bmat.toLocaleString('en')+"<img src=\"icons/bmat.png\" alt=\"bmat\" title=\"bmat\" class=\"mat-icon\"/>"):""
        response += (cost.emat > 0)?(cost.emat.toLocaleString('en')+"<img src=\"icons/emat.png\" alt=\"emat\" title=\"emat\" class=\"mat-icon\"/>"):""
        response += (cost.hemat > 0)?(cost.hemat.toLocaleString('en')+"<img src=\"icons/hemat.png\" alt=\"hemat\" title=\"hemat\" class=\"mat-icon\"/>"):""
        response += (cost.rmat > 0)?(cost.rmat.toLocaleString('en')+"<img src=\"icons/rmat.png\" alt=\"rmat\" title=\"rmat\" class=\"mat-icon\"/>"):""
        return response
      },
      ammoToString: function(item) {
        if(item.ammo.length === 0) {
          return ""
        }
        let response = "Use "
        let context = this
        item.ammo.forEach(function(ammo) {
          response += context.dataByKey[ammo].name + "<img src=\"icons/" + ammo + ".png\" alt=\"" + 
            context.dataByKey[ammo].name + "\" title=\"" + context.dataByKey[ammo].name + "\" class=\"mat-icon\"/>"
        })
        return response
      }
    }
  };
</script>

<style type="text/css">
  .short-input {
    max-width: 100px!important;
  }
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
  .last-item-bloc {
    margin-right: 15px;
  }
  .odd-bloc {
    background-color: #464646;
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
