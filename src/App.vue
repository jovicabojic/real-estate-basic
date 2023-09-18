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
import 'vue-slick-carousel/dist/vue-slick-carousel.css'
import 'vue-slick-carousel/dist/vue-slick-carousel-theme.css'

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
      center: [46.94809, 7.44744],
      carouselOptions: {
        infinite: true,
        slidesToShow: 1,
        slidesToScroll: 1
      },
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
        // Create a custom icon
        const customIcon = L.icon({
          iconUrl: LocationPin, // Update with the correct path to your icon
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
      return `
        <div>${publication.publicationTitle}
        <VueSlickCarousel :arrows="true" :dots="true">
        ${this.createCarouselSlides(publication)}</VueSlickCarousel>
        </template></div>`
    },

    createCarouselSlides(publication) {
      return publication.pictures.map((picture) => `
        <div>
          <img src="${picture.Url}" alt="" />
        </div>
      `).join('');
    },
  }
};
</script>
