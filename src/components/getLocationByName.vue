<template>
   <div>
      <section class="ui two column centered grid">
         <div class="column">
            <form class="ui segment large form">
               <div class="field">
                  <div class="ui right icon input small">
                     <input 
                        type="text"
                        id="searchTextField"
                        ref="searchTextField" 
                        placeholder="Enter Location Name" 
                        v-model="location" 
                        @keyup.enter.native="onClickSearchLocation">
                  </div>
               </div>
               <button class="ui button" id="searchButton" @click="onClickSearchLocation">
                  Search
               </button>
            </form>
         </div>
      </section>
      <div class="ui divider"></div>
      <section id="map" ref="map"></section>
   </div>
</template>

<script>

import axios  from 'axios';

export default {

   data() {
      return {
         location: "",
         coords: {
            lat: 0,
            lng: 0
         }
      }
   },

   mounted() {
      const searchBox = new google.maps.places.SearchBox(this.$refs['searchTextField'], {
            bounds: new google.maps.LatLngBounds(
               new google.maps.LatLng(43.8561002 -79.3370188),
            ),
         }
      );

      searchBox.addListener('places_changed', () => {
         const places = searchBox.getPlaces();
         this.coords.lat = places[0].geometry.location.lat();
         this.coords.lng = places[0].geometry.location.lng();
         this.location = places[0].formatted_address;
         this.showLocationOnMap(this.coords.lat, this.coords.lng);
      });
   },

   methods: {
      onClickSearchLocation() {
         const coords = this.getLocationCoordsByName(this.$refs['searchTextField'].value);
         this.showLocationOnMap(coords.lat, coords.lng);
      },

      getLocationCoordsByName(location) {
         let coords = {
            lat: 0,
            lng: 0
         };

         axios.get(`https://maps.googleapis.com/maps/api/geocode/json?address=${location}&key=AIzaSyAAIeIPfYidBpqUke316LbK720IMd5m-sQ`)
            .then((response) => {
               console.log(response.data);
               if (response.data.status !== 'OK') {
                  throw new Error('Unable to get coordinates for the location');
               } else if (response.data.status === 'ZERO_RESULTS') {
                  throw new Error('No results found for the location');
               } else {
                  console.log(response.data.results[0].geometry.location.lat);
                  console.log(response.data.results[0].geometry.location.lng);
               }               
            })
            .catch((error) => {
               console.log(error);
            });
         
         return coords;
      },

      showLocationOnMap(lat, lng) {
         var map = new google.maps.Map(this.$refs['map'], {
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