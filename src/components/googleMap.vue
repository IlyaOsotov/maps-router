<template>
   <div class="map-container">
    <GmapMap
      ref="mapRef"
      :center="{lat:55.7492, lng:37.6161}"
      :zoom="14"
      class="google-map"
    ></GmapMap>
    <input @keyup.enter="addCity" v-model="newCity">
    <button @click="addCity">Добавить точку</button>
    <draggable
      class="list-group"
      v-model="citiesList"
      group="cities"
      @start="drag=true"
      @end="drag=false"
      @change="calculateAndDisplayRoute()">
      <div class="list-group-item-container" v-for="element in citiesList" :key="element">
        <div class="inline list-group-item">{{element}}</div>
        <button class="inline remove" @click="removeCity(element)">x</button>
      </div>
    </draggable>
   </div>
</template>

<script>
import { gmapApi } from 'vue2-google-maps';
import draggable from 'vuedraggable';

export default {
  name: 'google-map',
  data() {
    return {
      citiesList: ['Красная площадь', 'парк горького'],
      newCity: '',
      directionsService: null,
      directionsDisplay: null,
    };
  },
  components: {
    draggable,
  },
  computed: {
    google: gmapApi,
  },
  methods: {
    addCity() {
      this.citiesList.push(this.newCity);
      this.newCity = '';
      this.calculateAndDisplayRoute();
    },
    removeCity(element) {
      this.citiesList = this.citiesList.filter(el => el !== element);
      this.calculateAndDisplayRoute();
    },
    calculateAndDisplayRoute() {
      if (this.citiesList.length < 2) { return; }
      const [first, ...other] = this.citiesList;
      const last = other.pop();
      this.directionsService.route({
        origin: first,
        destination: last,
        travelMode: 'WALKING', // DRIVING
        waypoints: other.length === 0 ? [] : other.map(el => ({ location: el })),
      }, (response, status) => {
        if (status === 'OK') {
          this.directionsDisplay.setDirections(response);
        }
      });
    },
    updateCities() {
      // TODO: UPDATE cities
    },
  },
  mounted() {
    this.$refs.mapRef.$mapPromise.then((map) => {
      this.directionsService = new this.google.maps.DirectionsService();
      this.directionsDisplay = new this.google.maps.DirectionsRenderer({ draggable: true, map });
      const that = this;
      this.directionsDisplay.addListener('directions_changed', () => {
        // that.updateCities(that.directionsDisplay.getDirections());
        that.updateCities();
      });


      this.calculateAndDisplayRoute(this.directionsDisplay,
        this.directionsService);
    });
  },
};
</script>

<style scoped>
button {
  background-color: #4CAF50;
  border: none;
  color: white;
  padding: 15px 32px;
  margin: 0 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  height: 45px;
}
input {
  height: 40px;
}
.inline {
  display: inline-block;
}
.remove {
  background-color: coral;
  padding: 15px 25px;
}
.map-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
}
.list-group {
  display: flex;
  flex-direction: column;
  padding-left: 0;
  margin-bottom: 0;
  min-height: 20px;
}
.list-group-item-container {
  margin: 5px 0;
}
.list-group-item {
  cursor: move;
  position: relative;
  padding: .75rem 1.25rem;
  margin-bottom: -1px;
  background-color: #fff;
  border: 1px solid rgba(0,0,0,.125);
  width: 200px;
}
.google-map {
  width: 640px;
  height: 480px;
  margin: 0 20px;
  background: gray;
}
</style>
