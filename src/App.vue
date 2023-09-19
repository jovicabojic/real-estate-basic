<template>
  <div id="app">
    <l-map ref="map" style="height: 600px; width: 1000px; margin: 0 auto;" :zoom="zoom" :center="center">
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
import PlaceholderImage from "@/assets/placeholder.jpg";

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
          icon: customIcon,
        });
        marker.addTo(this.$refs.map.mapObject);

        marker.bindPopup(this.createPopupContent(publication), {
          keepInView: true,
          maxHeight: 400,
          offset: [0, -10],
        });
        marker.on("mouseover", () => {
          marker.openPopup();
        });
      });
    },
    createPopupContent(publication) {
      const maxDescriptionLength = 100;
      const truncatedDescription =
          publication.description.length > maxDescriptionLength
              ? `${publication.description.slice(0, maxDescriptionLength)}...`
              : publication.description;

      const imageUrl =
          publication.pictures.length > 0 ? publication.pictures[0].Url : PlaceholderImage;

      let additionalInfoContent = '';

      if (publication.rooms) {
        additionalInfoContent += `<p>Rooms: ${publication.rooms}</p>`;
      }

      if (publication.livingArea) {
        additionalInfoContent += `<p>Living area: ${publication.livingArea} m2</p>`;
      }

      return `<div class="custom-popup">
            <div class="popup-ribbon">
                ${publication.propertyCategory}
            </div>
            <div class="img-wrapper">
                  <img src="${imageUrl}" alt="" />
                  <div class="additional-info">
                    ${additionalInfoContent}
                  </div>
            </div>
            <div class="popup-content">
                <h3>${publication.publicationTitle}</h3>
                <p>${truncatedDescription}</p>
                <h2>${this.formatPrice(publication.price)}€</h2>
            </div>
           </div>`;
    },

    formatPrice(value) {
      let val = (value / 1).toFixed(2).replace('.', ',')
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")
    }
  }
};
</script>

<style>
body {
  padding: 0;
  margin: 0;
}

.leaflet-popup-content {
  margin: 0;
}

.leaflet-popup-content-wrapper {
  padding: 0;
  overflow: hidden;
}

.leaflet-container a.leaflet-popup-close-button {
  right: 20px;
  top: 20px;
  width: 30px;
  height: 30px;
  background: rgba(0,0,0,.5);
  border-radius: 50%;
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
}

.custom-popup {
  position: relative;
}

.custom-popup .popup-ribbon {
  background: #db0253;
  position: absolute;
  top: 20px;
  left: 20px;
  color: #fff;
  padding: 5px 10px;
  border-radius: 4px;
  z-index: 2;
}

.custom-popup img {
  width: 100%;
}

.popup-content {
  padding: 20px;
}

.popup-content h3 {
  margin-top: 0;
}

.custom-popup .img-wrapper {
  position: relative;
}

.custom-popup .img-wrapper .additional-info {
  background: rgba(0,0,0, .6);
  border-radius: 4px;
  color: #fff;
  position: absolute;
  left: 20px;
  bottom: 20px;
  padding: 5px 10px;
  display: flex;
  gap: 20px;
}

.custom-popup .img-wrapper .additional-info p {
  margin: 0;
}
</style>
