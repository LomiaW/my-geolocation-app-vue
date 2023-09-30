<template>
   <section class="ui two row centered grid">
      <div class="row">
         <button class="ui button" name="getMyLocation" @click="onPressGetMyLocation">
            Get My Location
         </button>
      </div>
      <div class="ui row grid">
         <div class="ui message red" v-show="location">{{ location }}</div>
      </div>
   </section>
</template>

<script>
   import axios  from 'axios';

   export default {

      data() {
         return {
            location: "",
            error:"",
         }
      },

      methods: {
         onPressGetMyLocation() {
            if(navigator.geolocation) {
               navigator.geolocation.getCurrentPosition(position=> {
                  var latitude = position.coords.latitude;
                  var longitude = position.coords.longitude;

                  axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" 
                              + latitude + "," + longitude 
                              + "&key=AIzaSyAAIeIPfYidBpqUke316LbK720IMd5m-sQ")
                  .then(response => {
                     if (response.data.status === "OK") {
                        var city = response.data.results[0].address_components[3].long_name;
                        var province = response.data.results[0].address_components[6].long_name;
                        var country = response.data.results[0].address_components[7].long_name;
                        this.location = city + ", " + province + ", " + country;
                     } else {
                        console.log("Error: " + response.data.error_message);
                     }
                  })
                  .catch(error => {
                     console.log(error.message);
                  });
               });
            } else {
               this.error = error.message;
               console.log("Geolocation is not supported by this browser.");
            }
         },
      }
   }
</script>

<style>
.ui.button {
   background-color: #ee585d;
   color: white;
}
</style>