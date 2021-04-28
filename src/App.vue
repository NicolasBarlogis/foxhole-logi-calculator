<template>
  <v-app>
    <v-main>
      <Banner/>
      <MassProductionFactory/>
      <div class="divider"></div>
      <Factory/>
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
    Garage,
    ConstructionYard,
    Shipyard
  },

  data: () => ({
    //
  }),
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
</style>
