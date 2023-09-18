<template>
  <div id="app">
    <l-map ref="map" style="height: 800px; width: 1000px; margin: 0 auto;" :zoom="zoom" :center="center">
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <!-- l-popup will be handled in the JavaScript part -->
    </l-map>
  </div>
</template>

<script>
import axios from "axios";
import {LMap, LTileLayer} from 'vue2-leaflet';
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import LocationPin from "@/assets/location.png";

export default {
  name: 'App',
  components: {
    LMap,
    LTileLayer
  },
  data() {
    return {
      publications: [],
      url: 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
      zoom: 13,
      center: [46.94809, 7.44744]
    };
  },
  computed: {
    // Convert coordinates based on the publication's address
    convertedCoordinates() {
      return (publication) => {
        if (publication.address && publication.address.coordinates) {
          const {latitude, longitude} = publication.address.coordinates;
          return [latitude, longitude];
        }
        return [0, 0];
      };
    }
  },
  mounted() {
    this.getPublications();
  },
  methods: {
    async getPublications() {
      try {
        const response = await axios.get('https://blattcodeservices.com/pct/search');
        this.publications = response.data.publications;
        // Create custom markers and bind popups on mouseover
        this.createCustomMarkers();
      } catch (error) {
        console.error('Error fetching publications:', error);
      }
    },
    createCustomMarkers() {
      this.publications.forEach((publication) => {
        const customIcon = L.icon({
          iconUrl: LocationPin,
          iconSize: [30, 30],
        });
        const marker = L.marker(this.convertedCoordinates(publication), {
          icon: customIcon, // Assign the custom icon to the marker
        });
        marker.addTo(this.$refs.map.mapObject); // Add the marker to the map

        // Bind popup on mouseover
        marker.bindPopup(this.createPopupContent(publication), {
          keepInView: true,
          maxHeight: 300,
          offset: [0, -10],
        });

        // Add mouseover and mouseout event listeners to show and hide the popup
        marker.on("mouseover", () => {
          marker.openPopup();
        });

        // marker.on("mouseout", () => {
        //   marker.closePopup();
        // });
      });
    },
    createPopupContent(publication) {
      const formattedDescription = publication.description
          .split('/n') // Split the description into paragraphs using '/n'
          .map((paragraph) => `<p>${paragraph}</p>`) // Wrap each paragraph in <p> tags
          .join(''); // Join the paragraphs back together

      return `
    <div>
      ${publication.publicationTitle}<br>
      ${this.formatPrice(publication.price)}€<br>
      ${formattedDescription}
    </div>
  `;
    },

    formatPrice(value) {
      let val = (value/1).toFixed(2).replace('.', ',')
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")
    }
  }
};
</script>

<style>
</style>
