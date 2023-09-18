<template>
  <div id="app">
    <l-map style="height: 800px; width: 1000px; margin: 0 auto;" :zoom="zoom" :center="center">
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <l-marker
          v-for="publication in publications"
          :key="publication.id"
          :lat-lng="convertedCoordinates(publication)"
      >
        <l-icon ref="icon">
          <img class="pin-icon" src="@/assets/logo.png" alt=""/>
        </l-icon>
        <l-popup>
          <div>
            {{ publication.publicationTitle }}
            <VueSlickCarousel :arrows="true" :dots="true">
              <div v-for="(picture, index) in publication.pictures" :key="index">
                <img :src="picture.Url" alt="" />
              </div>
            </VueSlickCarousel>
          </div>
        </l-popup>
      </l-marker>
    </l-map>
  </div>
</template>

<script>
import axios from "axios";
import {LMap, LTileLayer, LMarker, LPopup, LIcon} from 'vue2-leaflet';
import VueSlickCarousel from 'vue-slick-carousel'
import 'vue-slick-carousel/dist/vue-slick-carousel.css'
import 'vue-slick-carousel/dist/vue-slick-carousel-theme.css'
import "leaflet";
import "leaflet/dist/leaflet.css";

export default {
  name: 'App',
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LIcon,
    VueSlickCarousel
  },
  data () {
    return {
      publications: [],
      url: 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}.png',
      attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
      zoom: 13,
      center: [46.94809, 7.44744]
    };
  },
  computed: {
    // Convert coordinates based on the publication's address
    convertedCoordinates() {
      return (publication) => {
        if (publication.address && publication.address.coordinates) {
          const { latitude, longitude } = publication.address.coordinates;
          return [latitude, longitude];
        }
        return [0, 0]; // Default coordinates if not available
      };
    }
  },
  mounted() {
    // Call the getPublications method to fetch data
    this.getPublications();
  },
  methods: {
    async getPublications() {
      try {
        const response = await axios.get('https://blattcodeservices.com/pct/search');
        this.publications = response.data.publications; // Set the fetched data to the publications array
      } catch (error) {
        console.error('Error fetching publications:', error);
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body {
  padding: 0;
  margin: 0;
}

img {
  width: 100%;
}

.leaflet-container .leaflet-marker-pane img, .leaflet-container .leaflet-shadow-pane img, .leaflet-container .leaflet-tile-pane img, .leaflet-container img.leaflet-image-layer, .leaflet-container .leaflet-tile {
  width: 100%;
}
</style>
