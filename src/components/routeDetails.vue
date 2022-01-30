<template>
  <div class="route-details">
    <p>Total distance is:</p>
    <span>{{ TotalDistance }} km</span>
    <p>Total duration is:</p>
    <span>{{ TotalDuration }} hr</span>
    <p>The average of trip duration is:</p>
    <span>{{ TripAverage }} min</span>
  </div>
</template>
<script>
export default {
  name: "routeDetails",
  props: ["TotalDistance", "TotalDuration", "TripAverage"],
};
</script>
<style>
.route-details {
  padding: 10px;
  border-radius: 5px;
  background: linear-gradient(146deg, #212529, #212529e6);
  box-shadow: 0 1px 10px #21252957;
  color: #fff;
  margin-top: 30px;
}
.route-details > p {
  padding: 0;
  margin: 10px 0 0;
  font-size: 16px;
  font-weight: 600;
}
.route-details > p:first-child {
  margin: 0;
}
.route-details > span {
  color: #cfd8dc;
}
</style>
