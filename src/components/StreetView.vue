<template>
  <div id="game-page">
    <HeaderGame 
      :score="score"
      :round="round" />
    <div id="street-view-container">
      <div id="street-view">
      </div>
      <Maps
        :randomLatLng="randomLatLng"
        :round="round"
        :score="score"
        @calculateDistance="updateScore"
        @goToNextRound="goToNextRound"
        @playAgain="playAgain" />
    </div>
    <v-overlay 
      :value="overlay"
      opacity="0.8" 
      z-index="2"/>
  </div>
</template>

<script>
/* global google L */
import HeaderGame from '@/components/HeaderGame';
import Maps from '@/components/Maps';

export default {
    components: {
        HeaderGame,
        Maps,
    },
    data() {
        return {
            randomLatLng: null,
            score: 0,
            round: 1,
            overlay: false,
            panorama: null
        };
    },
    methods: {
        loadStreetView() {
            var service = new google.maps.StreetViewService();
            service.getPanorama({
                location: this.getRandomLatLng(),
                preference: 'nearest',
                radius: 100000,
                source: 'outdoor',
            }, this.checkStreetView);
        },
        getRandomLatLng() {
        // Generate a random latitude and longitude
            var lat = (Math.random() * 170) - 85;
            var lng = (Math.random() * 360) - 180;
            return new google.maps.LatLng(lat, lng);
        },
        isBannedLocation(error, result) {
            if(!result) {
                console.error(error);
                return true;
            }
            const bannedCountries = ['RU'];
            return bannedCountries.indexOf(result.results[0].locations[0].adminArea1) >= 0;
        },
        checkStreetView(data, status) {
        // Generate random streetview until the valid one is generated
            if (status === 'OK') {
                var geocoder = L.mapquest.geocoding({
                    thumbMaps: false,
                    maxResults: 1
                });
                geocoder.reverse(
                    [
                        data.location.latLng.lat(),
                        data.location.latLng.lng()
                    ],
                    (err, result) => {
                        if(this.isBannedLocation(err, result)) {
                            this.loadStreetView();
                            return;
                        }
                        
                        this.panorama.setPano(data.location.pano);
                        this.panorama.setPov({
                            heading: 270,
                            pitch: 0,
                        });

                        // Save the location's latitude and longitude
                        this.randomLatLng = data.location.latLng;
                    }
                );
            } else {
                this.loadStreetView();
            }
        },
        updateScore(distance) {
            this.score += distance;
            this.overlay = true;
        },
        goToNextRound() {
        // Reset
            this.randomLatLng = null;
            this.overlay = false;

            // Update the round
            this.round += 1;

            // Replace streetview with new one
            this.loadStreetView();
        },
        playAgain() {
        // Reset
            this.randomLatLng = null;
            this.distance = null;
            this.score = 0;
            this.round = 1;
            this.overlay = false;

            // Load streetview
            this.loadStreetView();
        }
    },
    mounted() {
        // Generate the first streetview and check if it's valid
        this.panorama = new google.maps.StreetViewPanorama(document.getElementById('street-view'));
        this.panorama.setOptions({
            addressControl: false,
            fullscreenControl: false,
            motionTracking: false,
            motionTrackingControl: false,
            showRoadLabels: false,
        });
        this.loadStreetView();
    },
};
</script>

<style scoped>
  #game-page {
    position: relative;
    height: 100%; 
    width: 100%; 
    top: 0; 
    left: 0;
  }

  #street-view-container {
    position: absolute;
    height: 100%; 
    width: 100%; 
    top: 0; 
    left: 0; 
  }

  #street-view {
    position: relative;
    min-height: 100%;
    width: 100%;
  }

  @media (max-width: 450px) {
    #game-page {
      position: fixed;
      height: 92%;
    }
  }
</style>
