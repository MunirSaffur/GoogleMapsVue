<template>
  <tr>
    <td>{{ passengerName }}</td>
    <td>{{ tripDuration }}</td>
    <td>{{ tripDistance }}</td>
    <td>{{ pickUpPointOrder }}</td>
  </tr>
</template>

<script>
export default {
  name: "dataTable",
  props: ["passengerName", "tripDuration", "tripDistance", "pickUpPointOrder"],
};
</script>
