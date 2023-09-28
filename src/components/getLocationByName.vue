<template>
   <div>
      <section class="ui two column centered grid">
         <div class="column">
            <form class="ui segment large form">
               <div class="field">
                  <div class="ui right icon input small" :class="{loading:spinner}">
                     <input type="text" id="autocomplete" placeholder="Enter Location Name" v-model="location" @keyup.enter.native="onClickSearchLocation">
                     <i class="dot circle link icon"></i>
                  </div>
               </div>
               <button class="ui button" name="getLocation" @click="onClickSearchLocation">
                  Search
               </button>
            </form>
         </div>
      </section>
      <section id="map"></section>
   </div>
</template>

<script>

import axios  from 'axios';

export default {

   data() {
      return {
         location: "",
         spinner: false
      }
   },

   mounted() {
      let autocomplete = new google.maps.places.Autocomplete(
         this.$refs['autocomplete'],
         {
            bounds: new google.maps.LatLngBounds(
               new google.maps.LatLng(7.0711, 125.6128),
            ),
         }
      );

      autocomplete.addListener('place_changed', () => {
         var place = autocomplete.getPlace();
         console(place);
         this.location = place.formatted_address;
         this.showLocationOnMap(place.geometry.location.lat(), place.geometry.location.lng());
      });
   },

   methods: {
      onClickSearchLocation() {
         this.spinner = true;

         if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition((position) => {
               this.getLocationByName(position.coords.latitude, position.coords.longitude);

               this.showLocationOnMap(position.coords.latitude, position.coords.longitude);
            }),
            (error) => {
               this.spinner = false;
               console.log(error.message);
            }
         } else {
            this.spinner = false;
            alert('Geolocation is not supported by this browser.');
         }

      },

      getLocationByName(lat, lng) {
         axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" 
                     + latitude + "," + longitude 
                     + "&key=AIzaSyAAIeIPfYidBpqUke316LbK720IMd5m-sQ")
         .then((response) => {
            this.spinner = false;

            if (response.data.error_message) {
               console.log("Error: " + response.data.error_message);
               return;
            } else {
               console.log(response.data.results[0].geometry);
               this.location = response.data.results[0].formatted;
            }
         })
         .catch((error) => {
            this.spinner = false;
            console.log(error);
         });
      },

      showLocationOnMap(lat, lng) {
         var map = new google.maps.Map(document.getElementById('map'), {
            center: new google.maps.LatLng(lat, lng),
            zoom: 8,
            mapTypeId: google.maps.MapTypeId.ROADMAP
         });

         new google.maps.Marker({
            position: new google.maps.LatLng(lat, lng),
            map: map,
            title: 'Recent Searched Location'
         });
      }
   }
}

</script>

<style scoped>
#map {
   height: 400px;
   width: 100%;
}
</style>