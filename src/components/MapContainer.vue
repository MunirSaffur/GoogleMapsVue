<template>
  <div class="hello">
    <h1 class="mb-4">Map Project</h1>
    <div class="row">
      <div class="col-md-9">
        <!-- Adding Company Location: start -->
        <div class="company-location">
          <b-form>
            <b-input-group class="mb-3">
              <GmapAutocomplete
                placeholder="Enter Origin Location"
                class="originInput"
                @place_changed="setPlace"
                :options="{
                  fields: ['geometry', 'formatted_address'],
                }"
              />
              <b-input-group-append class="ms-2">
                <b-button class="btn-success" @click="addOriginPoint">
                  Add Origin Location
                </b-button>
              </b-input-group-append>
            </b-input-group>
          </b-form>
          <p id="origin_err"></p>
        </div>
        <!-- Adding Company Location: end -->

        <!-- Map Container: start -->
        <GmapMap ref="baseMap" :center="center" :zoom="12" class="map">
          <GmapMarker
            :key="index"
            v-for="(m, index) in markers"
            :position="m.position"
            @click="center = m.position"
          />
          <DirectionsRenderer
            travelMode="DRIVING"
            :origin="startLocation"
            :destination="endLocation"
            :waypoints="waypnt"
            :optimizeWaypoints="true"
          />

          <!-- this if you want to show the infoWindow in map -->
          <!-- <gmap-info-window :position="infoBox" :opened="true">
            {{ "Total distance is: " + totalDistance + " km" }}
            <br />
            {{ "Total duration is: " + totalDuration + " hr" }}
            <br />
            {{ "The average of trip duration is: " + averageDuration + " min" }}
          </gmap-info-window> -->
        </GmapMap>
        <!-- Map Container: end -->
      </div>
      <div class="col-md-3">
        <!-- Add passenger' Name & Loacation: start -->
        <b-form>
          <b-form-group
            label="passenger's Name"
            label-for="PassengerName"
            class="my-3 text-start"
          >
            <b-form-input
              id="passengerName"
              type="text"
              placeholder="Ente a name"
              value=""
              class="passengerInputs"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Passenger's destination point"
            label-for="passengerLocation"
            class="my-3 text-start"
          >
            <GmapAutocomplete
              id="passengerLocation"
              class="passengerInputs"
              placeholder="Type here..."
              @place_changed="setPlace"
              :options="{
                fields: ['geometry', 'formatted_address'],
              }"
            />
          </b-form-group>
          <b-button class="btn btn-success w-100" @click="addDestinationPoint">
            Get Direction
          </b-button>
        </b-form>
        <!-- Add passengers's Name & Loacation: end -->
        <p id="passenger_err"></p>
        <b-alert id="alertErr" show variant="danger"></b-alert>

        <!-- Route details  -->
        <routeDetails
          :TotalDistance="totalDistance"
          :TotalDuration="calcTotalDuration"
          :TripAverage="averageDuration"
        />
      </div>
    </div>
    <div class="row">
      <div class="col-md-8">
        <!-- data table: start -->
        <div class="mt-4">
          <h3>Passengers table</h3>
          <table class="table">
            <thead>
              <tr>
                <th>Passenger name</th>
                <th>Trip duration</th>
                <th>Trip Distance</th>
                <th>Pickup point order</th>
              </tr>
            </thead>
            <tbody>
              <dataTable
                :key="tableIndex"
                v-for="(t, tableIndex) in passengers"
                :passengerName="t.name"
                :tripDuration="t.tripDurationText"
                :tripDistance="t.tripDistanceText"
                :pickUpPointOrder="t.pointOrder"
              />
            </tbody>
          </table>
        </div>
        <!-- data table: end -->
      </div>
    </div>
  </div>
</template>

<script>
import { gmapApi } from "vue2-google-maps";
import dataTable from "./dataTable.vue";
import DirectionsRenderer from "./DirectionsRenderer";
import routeDetails from "./routeDetails.vue";

const originPointAmount = [];
const destinationPointAmount = [];

export default {
  name: "MapContainer",
  data() {
    return {
      center: { lat: 41.04538761823768, lng: 28.990736439754617 },
      currentPlace: null,
      markers: [],
      originPoint: [],
      passengerInfo: { name: "", pickUpPoint: {} },
      passengers: [],
      startLocation: null,
      endLocation: null,
      waypnt: [],
      totalDistance: 0,
      totalDuration: 0,
      calcTotalDuration: 0,
      averageDuration: 0,
      infoBox: null,
      pointOrder: 0,
    };
  },
  computed: {
    google: gmapApi,
  },
  mounted() {
    // this is for TrafficLayer map mode
    // this.$refs.baseMap.$mapPromise.then((map) => {
    //   const trafficLayer = new window.google.maps.TrafficLayer();
    //   trafficLayer.setMap(map);
    // });
  },
  methods: {
    setPlace(place) {
      this.currentPlace = place;
    },
    addDestinationPoint() {
      const passengerName = document.getElementById("passengerName").value;
      if (this.currentPlace && passengerName !== "") {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng(),
        };

        this.markers.push({ position: marker });
        this.tripDistanceDuration(marker);
        destinationPointAmount.push(this.currentPlace.formatted_address);
        this.passengerInfo.name = passengerName;
        this.passengerInfo.pickUpPoint = marker;
        if (this.markers.length > 1) {
          this.infoBox = this.markers[0].position;
        }
        this.center = marker;
        this.currentPlace = null;
        this.endLocation = marker;
        this.waypnt.push({
          location: marker,
          stopover: false,
        });
      } else if (passengerName === "") {
        this.showErr("Passenger name can't be blank!", "passenger_err");
      } else if (this.totalDuration > 2) {
        this.showErr("The rout duration is over 2 hours!", "passenger_err");
      } else {
        this.showErr("You need to add an origin location", "passenger_err");
      }
    },

    addOriginPoint() {
      if (this.currentPlace && originPointAmount.length < 1) {
        const originMarker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng(),
        };
        this.markers.push({ position: originMarker });
        this.originPoint.push({ originMarker });
        originPointAmount.push(this.currentPlace.formatted_address);
        this.center = originMarker;
        this.startLocation = originMarker;
      } else if (originPointAmount.length == 0) {
        this.showErr("You need to add an origin location", "origin_err");
      } else if (originPointAmount.length == 1) {
        this.showErr(
          "You have added an origin location, to reset it please reload the window",
          "origin_err"
        );
      }
    },

    tripDistanceDuration(destination) {
      const request = {
        origins: [this.markers[this.markers.length - 2].position],
        destinations: [destination],
        travelMode: window.google.maps.TravelMode.DRIVING,
        unitSystem: window.google.maps.UnitSystem.METRIC,
        avoidHighways: false,
        avoidTolls: false,
      };
      this.pointOrder += 1;
      const serv = new window.google.maps.DistanceMatrixService();
      serv.getDistanceMatrix(request, (response) => {
        this.passengers.push({
          pointOrder: this.pointOrder,
          ...this.passengerInfo,
          tripDistanceText: response.rows[0].elements[0].distance.text,
          tripDistanceValue: response.rows[0].elements[0].distance.value,
          tripDurationText: response.rows[0].elements[0].duration.text,
          tripDurationValue: response.rows[0].elements[0].duration.value,
        });

        // total route distance & total route duration
        this.totalDistance +=
          response.rows[0].elements[0].distance.value / 1000;

        this.totalDuration +=
          response.rows[0].elements[0].duration.value / 3600;
        this.calcTotalDuration = this.totalDuration.toFixed(2);

        // average of passengers tripDuration
        let averageDuration = (
          this.passengers.reduce(
            (total, next) => total + next.tripDurationValue,
            0
          ) /
          this.passengers.length /
          60
        ).toFixed(2);
        this.averageDuration = averageDuration;

        // validation
        if (this.totalDuration > 2) {
          this.showAlert("Warning! the duration of route is over 2 hours.");
        }
        if (destinationPointAmount.length > 9) {
          this.showAlert("Warning! Passengers amount is over 9.");
        }
      });
    },
    showAlert(alertMessage) {
      document.getElementById("alertErr").innerHTML = alertMessage;
      setTimeout(() => {
        document.getElementById("alertErr").innerHTML = "";
      }, 5000);
    },
    showErr(Message, errId) {
      const errContainer = document.getElementById(`${errId}`);
      errContainer.innerHTML = Message;
      setTimeout(() => {
        errContainer.innerHTML = "";
      }, 3000);
    },
  },
  components: {
    dataTable,
    DirectionsRenderer,
    routeDetails,
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.map {
  width: 100%;
  height: 400px;
}
.originInput {
  width: 70%;
  border: 1px solid #b0bec5;
  padding: 0 20px;
  border-radius: 3px !important;
}
.btn-success,
.btn-success:hover {
  color: #fff;
  background-color: #ff5722 !important;
  border-color: #ff5722 !important;
  box-shadow: 0 1px 10px #ff57224a;
}
.passengerInputs {
  width: 100%;
  padding: 5px 10px;
  border: 1px solid #b0bec5;
  border-radius: 3px !important;
}
input:focus {
  color: #212529;
  background-color: #fff;
  border-color: #ff562241;
  outline: 0;
  box-shadow: 0 0 0 5px #ff562213;
}
button:focus {
  box-shadow: 0 0 0 5px #ff562213;
}
#passenger_err,
#origin_err {
  margin: 10px 0;
  color: red;
}
#alertErr:empty {
  display: none;
}
#alertErr {
  position: fixed;
  bottom: 10px;
  right: 10px;
  padding: 10px 30px;
  background: linear-gradient(45deg, #d32f2f8c, #ff572280);
  color: #fff;
  box-shadow: 0 1px 10px #ff57224d;
  border: 0;
}
</style>
