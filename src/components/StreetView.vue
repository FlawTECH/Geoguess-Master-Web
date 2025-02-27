<template>
  <div id="game-page">
    <HeaderGame 
      :score="score"
      :round="round"
      v-on:reset-position="onResetPosition"
      v-on:refresh-streetview="onRefreshStreetView" />
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

import { countryCodes } from '@/helpers/country.helper';

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
            panorama: null,
            startPano: null,
            countriesChance: null
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
            // Check country probability
            let randomNumber = Math.random();
            return this.countriesChance.get(result.results[0].locations[0].adminArea1) < randomNumber;
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
                        
                        this.startPano = data.location.pano;
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
        },
        onResetPosition() {
            this.panorama.setPano(this.startPano);
            this.panorama.setPov({
                heading: 270,
                pitch: 0,
            });
        },
        onRefreshStreetView() {
            // This is a dirty fix and should be considered unstable.
            // Google creates an instance and doesn't allow itself getting destroyed
            // Removing the generated HTML kind of fixes the issue,
            // But the page should be refreshed after a successful round.
            // https://stackoverflow.com/questions/10485582/what-is-the-proper-way-to-destroy-a-map-instance
            let currentPano = this.panorama.getPano();
            document.getElementById('street-view').innerHTML = '';
            this.panorama = new google.maps.StreetViewPanorama(document.getElementById('street-view'));
            this.panorama.setOptions({
                addressControl: false,
                fullscreenControl: false,
                motionTracking: false,
                motionTrackingControl: false,
                showRoadLabels: false,
            });
            this.panorama.setPano(currentPano);
            this.panorama.setPov({
                heading: 270,
                pitch: 0,
            });
        }
    },
    mounted() {
        // Init variables
        this.countriesChance = new Map();
        countryCodes.forEach(code => {
            this.countriesChance.set(code, 1.0);
        });
        this.countriesChance.set('RU', 0.3);
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
