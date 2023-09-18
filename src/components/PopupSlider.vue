<template>
  <div v-if="publication">
    <p>{{ publication && publication.publicationTitle }}</p>
    <VueSlickCarousel :dots="true">
      <div v-for="(picture, index) in publication.pictures" :key="index">
        <img :src="picture.Url" alt="" />
      </div>
    </VueSlickCarousel>
  </div>
</template>
<script>
import VueSlickCarousel from "vue-slick-carousel"
import 'vue-slick-carousel/dist/vue-slick-carousel.css'
import 'vue-slick-carousel/dist/vue-slick-carousel-theme.css'

export default {
  props: {
    publication: {
      type: Object
    }
  },
  components: {
    VueSlickCarousel
  }
}
</script>
