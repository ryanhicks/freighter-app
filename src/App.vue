<template>
  <div id="app">
    <ShipmentHeader/>
    <ShipmentOffers/>
  </div>
</template>

<script>
import ShipmentHeader from "./components/ShipmentHeader";
import ShipmentOffers from "./components/ShipmentOffers";

export default {
  name: "App",
  components: {
    ShipmentHeader,
    ShipmentOffers
  }
};
</script>
