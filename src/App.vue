<template>
  <v-app>
    <v-main>
      <Banner/>
      <MassProductionFactory/>
      <div class="divider"></div>
      <v-container class="small-container text-end">
        <a 
          @click="switchFactoryAdvancedMode"
          class="orange--text"
        >
          <span v-if="!factoryAdvancedMode">Switch to factory spreadsheet mode</span>
          <span v-else>Switch to factory classic mode</span>
        </a>
      </v-container>
      <Factory v-if="!factoryAdvancedMode"/>
      <FactoryAdvanced v-else/>
      <div class="divider"></div>
      <Garage/>
      <div class="divider"></div>
      <ConstructionYard/>
      <div class="divider"></div>
      <Shipyard/>
    </v-main>
  </v-app>
</template>

<script>
import Banner from './components/Banner';
import MassProductionFactory from './components/MassProductionFactory';
import Factory from './components/Factory';
import FactoryAdvanced from './components/FactoryAdvanced';
import Garage from './components/Garage';
import ConstructionYard from './components/ConstructionYard';
import Shipyard from './components/Shipyard';

/*
 * wrong icon for shippable
 * active vehicle icon bad color
 */

export default {
  name: 'App',

  components: {
    Banner,
    MassProductionFactory,
    Factory,
    FactoryAdvanced,
    Garage,
    ConstructionYard,
    Shipyard
  },

  data: () => ({
    factoryAdvancedMode: false
  }),
  methods: {
    switchFactoryAdvancedMode: function() {
      this.factoryAdvancedMode = !this.factoryAdvancedMode
    }
  }
};
</script>

<style type="text/css">
  #app {
    background-color: #050505;
    color: #fff;
  }
  .divider {
    margin-top: 70px;
  }
  .small-container {
    padding-top: 0;
    padding-bottom: 0;
  }
</style>
