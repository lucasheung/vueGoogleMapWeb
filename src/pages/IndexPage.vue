<template>
  <div class="q-pa-md q-gutter-sm">
    <div class="q-pa-md text-h5">{{ $t("webTitle") }}</div>

    <div class="row">
      <div class="col-1">
        <q-btn
          flat
          class="full-height"
          icon="my_location"
          @click="getCurrentLocation"
        />
      </div>
      <input
        outlined
        ref="autocomplete"
        id="autocomplete"
        class="col-10"
        type="text"
        v-model="location"
        :placehodler="$t('searchLocation')"
      />
      <q-btn flat @click="searchLocation" icon="search" />
    </div>

    <div id="map"></div>
    <div class="q-pl-xl">
      {{ $t("currentTimeZone", { timeZone: this.timeZone, time: this.time }) }}
    </div>
    <div class="q-pa-sm">
      <q-table
        flat
        :rows="locations"
        :columns="columns"
        :title="$t('currentTimeZone')"
        row-key="location"
        selection="multiple"
        v-model:selected="selected"
        row-per-page="10"
        :rows-per-page-options="[10]"
      >
        <template v-slot:top>
          <q-btn
            no-caps
            class="q-ml-sm"
            color="primary"
            :label="$t('removeRow')"
            @click="removeMarker()"
          />
        </template>
      </q-table>
    </div>
  </div>
</template>

<script>
import { defineComponent, toRaw } from "vue";
import axios from "axios";

export default defineComponent({
  name: "IndexPage",
  data() {
    return {
      location: "",
      map: null,
      locations: [],
      columns: [
        {
          name: "location",
          align: "left",
          sortable: false,
          required: true,
          label: this.$t("location"),
          align: "left",
          field: (row) => row.location,
        },
      ],
      selected: [],
      autocomp: null,
      markers: [],
      timeZone: null,
      time: null,
    };
  },
  computed: {
    apiKey() {
      return this.$store.state.moduleExample.apiKey;
    },
  },
  methods: {
    searchLocation() {
      var place = this.autocomp.getPlace();
      if (place && place.geometry) {
        var existed = false;
        this.locations.forEach((location) => {
          if (location.name == place.name) existed = true;
        });
        if (existed) return;

        var temp = place;
        temp.location = place.name;
        this.locations.push(temp);
        this.setMarker();
        this.location = "";
        var timestamp = (new Date().getTime() / 1000).toFixed(0);
        var lat = place.geometry.location.lat();
        var lng = place.geometry.location.lng();
        var str = [lat, lng].toString();
        axios
          .get(
            `https://maps.googleapis.com/maps/api/timezone/json?location=${str}&timestamp=${timestamp}&key=${this.apiKey}`
          )
          .then((res) => {
            this.timeZone = res.data.timeZoneName;
            this.time = new Date().toLocaleString("en-US", {
              timeZone: res.data.timeZoneId,
            });
          });
      } else {
        this.$q.notify({
          message: this.$t("placeNotFound", { place: this.location }),
          timeout: 1500,
          type: "negative",
          position: "top",
        });
      }
    },
    initMap() {
      if (!window.google) window.location.reload();
      this.map = new window.google.maps.Map(document.getElementById("map"), {
        center: { lat: 43.6532, lng: -79.3832 },
        zoom: 3,
        maxZoom: 20,
        streetViewControl: false,
      });
    },
    setMarker() {
      for (let i = 0; i < this.locations.length; i++) {
        const mark = this.locations[i];
        var mar = new window.google.maps.Marker({
          map: this.map,
          draggable: false,
        });
        if (mark.geometry) mar.setPosition(mark.geometry.location);
        this.markers.push(mar);
      }
    },
    removeMarker() {
      var markersRemain = this.locations;
      this.selected.forEach((location) => {
        var temp = markersRemain.indexOf(location);
        if (temp != -1) markersRemain.splice(temp, 1);
      });
      this.markers.map((marker) => toRaw(marker).setMap(null));
      this.locations = markersRemain;
      this.setMarker();
      this.selected = [];
    },
    getCurrentLocation() {
      if (navigator.geolocation) {
        this.$q.loading.show();
        navigator.geolocation.getCurrentPosition(
          (position) => {
            var coord = [position.coords.latitude, position.coords.longitude];
            this.map.setCenter({ lat: coord[0], lng: coord[1] });

            new google.maps.Marker({
              position: { lat: coord[0], lng: coord[1] },
              map: this.map,
              label: this.$t("yourLocation"),
            });
            this.$q.loading.hide();
          },
          (err) => {
            this.$q.notify({
              message: err.message,
              timeout: 1500,
              type: "negative",
              position: "top",
            });
          }
        );
      } else {
        this.$q.notify({
          message: this.$t("geolocationNotSupport"),
          timeout: 1500,
          type: "negative",
          position: "top",
        });
      }
    },
  },
  mounted() {
    this.initMap();

    this.autocomp = new window.google.maps.places.Autocomplete(
      this.$refs["autocomplete"],
      {
        types: ["address"],
        fields: ["address_components", "geometry", "name", "timezone"],
      }
    );
    window.google.maps.event.addListener(this.autocomp, "place_changed", () => {
      this.searchLocation();
    });
  },
});
</script>
<style>
#map {
  height: 35vh;
}
</style>
