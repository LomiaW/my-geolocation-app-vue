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
      <!-- Show the time zone and local time of the latest searched location -->
      <div class="ui container centered" v-show="location">
         <div class="ui small message">
            <p>Time Zone: {{ timeZone }}</p>
            <p>Local Time: {{ localTime }}</p>
         </div>
         <div class="ui divider"></div>
      </div>
      
      <section id="map" ref="map"></section>
      <div class="ui divider"></div>
      <section id="table" ref="table">
         <div class="ui container">
            <table class="ui celled table">
               <thead>
                  <tr>
                     <th>Select</th>
                     <th>Searched Location</th>
                  </tr>
               </thead>
               <tbody>
                  <!-- Loop through the locations array and display locations -->
                  <tr v-for="(location, index) in locations" :key="index">
                     <td>
                        <div class="ui checkbox">
                           <input type="checkbox" name="checkbox">
                           <label></label>
                        </div>
                     </td>
                     <td>{{ location }}</td>
                  </tr>
               </tbody>
            </table>
            <div>
               <button class="ui button" id="clearButton" @click="onClickClearButton">Clear</button>
               <button class="ui button" id="moreButton" @click="onClickMoreButton">More</button>
            </div>
         </div>
      </section>
   </div>
</template>

<script>

import axios  from 'axios';

const GOOGLE_MAPS_API_KEY='AIzaSyAAIeIPfYidBpqUke316LbK720IMd5m-sQ'

export default {

   data() {
      return {
         location: "",
         coords: {
            lat: 0,
            lng: 0
         },
         locations: [],
         page: 1,
         timeZone: "",
         localTime: ""
      }
   },

   mounted() {
      // Initialize the locations array with pagination
      var locations = JSON.parse(localStorage.getItem('locations'));
      this.locations = locations? locations.slice((this.page - 1) * 10, this.page * 10) : [];

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
         
         // Save the location in the local storage array for later use
         this.saveSearchedLocation(this.location);

         // Use the Time Zone API for the searched location
         axios.get('https://maps.googleapis.com/maps/api/timezone/json?location=' 
                     + this.coords.lat + ',' + this.coords.lng 
                     + '&timestamp=' + Math.floor(Date.now() / 1000) 
                     + '&key=' + GOOGLE_MAPS_API_KEY)
         .then(response => {
            console.log(response.data)
            this.timeZone = response.data.timeZoneName;
            this.localTime = new Date(Date.now() + response.data.rawOffset * 1000 + response.data.dstOffset * 1000).toLocaleString();
         })
         .catch(error => {
            console.log(error)
         })

         this.showLocationOnMap(this.coords.lat, this.coords.lng);
      });
   },

   methods: {
      onClickSearchLocation() {
         var textInput = this.$refs['searchTextField'].value;
         console.log(textInput)
         // Find Place From Text API
         // https://maps.googleapis.com/maps/api/place/textsearch/output?parameters
         axios.get('https://maps.googleapis.com/maps/api/place/findplacefromtext/json&input='+ textInput +'&inputtype=textquery&key=${GOOGLE_MAPS_API_KEY}')
         .then(response => {
            console.log(response.data)
            this.coords.lat = response.data.candidates[0].geometry.location.lat;
            this.coords.lng = response.data.candidates[0].geometry.location.lng;
            this.location = response.data.candidates[0].formatted_address;

            // Save the location in the local storage array for later use
            this.saveSearchedLocation(this.location);

            this.showLocationOnMap(this.coords.lat, this.coords.lng);
         })
         .catch(error => {
            console.log(error)
         })
      },

      onClickClearButton() {
         var items = [];
         var checkboxes = document.getElementsByName('checkbox');
         for (var i=0; i<checkboxes.length; i++) {
            if (checkboxes[i].checked) {
               items.push(checkboxes[i].parentNode.parentNode.nextSibling.textContent);
            }
         }
         this.clearSelectedLocations(items);
      },

      onClickMoreButton() {
         // show next 10 locations
         var locations = JSON.parse(localStorage.getItem('locations'));
         if (locations.length <= 10) {
            alert('No more locations to show!');
            return;
         } else {
            this.locations = locations.slice((this.page - 1) * 10, this.page * 10);
            this.page++;
            // console.log(this.locations)
         } 
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
      },

      saveSearchedLocation(location) {
         let locations = localStorage.getItem('locations') ? JSON.parse(localStorage.getItem('locations')) : [];
         locations.push(location);
         localStorage.setItem('locations', JSON.stringify(locations));
         // console.log(locations)
      },

      clearSelectedLocations(items) {
         if (items.length > 0) {
            var locations = JSON.parse(localStorage.getItem('locations'));
            items.forEach(item => {
               locations.splice(locations.indexOf(item), 1);
            });
            localStorage.setItem('locations', JSON.stringify(locations));
         } else {
            alert('Please select at least one location to clear!');
         }
      },
   }
}

</script>

<style scoped>
#map {
   height: 400px;
   width: 100%;
}

#table {
   height: 400px;
   width: 100%;
   overflow: auto;
}
</style>