<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-spacer></v-spacer>
      <span class="mr-2">New Taipei Kindergardens</span>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-content>
      <v-data-table
        :headers="headers"
        :items="kindergardens"
        :items-per-page="20"
        class="elevation-1"
      >
        <template v-slot:item.direction="{ item }">
          <v-btn @click="calculateDirectionFrom(item.location)">GET</v-btn>
        </template>
      </v-data-table>
    </v-content>
  </v-app>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import axios from "axios";

interface HttpResponse {
  type: string;
  title: string;
  tel: string;
  address: string;
  recruit: string;
  twd97X: string;
  twd97Y: string;
  wgs84aX: string;
  wgs84aY: string;
}

interface LocationHttpResponse {
    name: string;
    location: {
      lat: number;
      lng: number;
    }; 
    address: string;
}

@Component
export default class App extends Vue {
  headers = [
    {
      text: "Name",
      align: "start",
      sortable: false,
      value: "name",
    },
    { text: "max students", value: "maxStudents" },
    { text: "address", value: "address" },
    { text: "type", value: "type" },
    { text: "phone", value: "phone" },
    { text: "direction", value: 'direction'}
  ];

  kindergardens: Array<{
    name: string;
    type: string;
    maxStudents: string;
    phone: string;
    address: string;
    location: {
      lat: number;
      lng: number;
    }; 
  }> = [];

  locations: Array<{
    name: string;
    location: {
      lat: number;
      lng: number;
    }; 
    address: string;
  }> = [];

  currentPosition: {lat: number; lng: number} = {
    lat: 0,
    lng: 0
  };

  calculateDirectionFrom(location: { lat: number; lng: number }) {
      this.getDirection(this.currentPosition, location);
  }

  protected async created() {
    navigator.geolocation.getCurrentPosition((position) => {
      this.currentPosition = {
        lat: position.coords.latitude,
        lng: position.coords.longitude
      }
    });
    const url = "./data.json";
    const result = await axios.get(`${url}`);
    const locationsResult = await axios.get(`./location.json`);
    this.kindergardens = result.data.map((x: HttpResponse) => {
      const location = locationsResult.data.find((l: LocationHttpResponse) => l.name === x.title);
      return {
        name: x.title,
        type: x.type,
        maxStudents: x.recruit,
        phone: x.tel,
        address: x.address,
        location: location && location.location
      };
    });
    
  }

  private async getDirection(start: { lat: number; lng: number }, end: { lat: number; lng: number }) {
    const key = `5b3ce3597851110001cf624879337138a41d4de8ab6a36f2ea031112`;
    const api = `https://api.openrouteservice.org/v2/directions/driving-car?api_key=${key}&start=${start.lng},${start.lat}&end=${end.lng},${end.lat}`;
    const result = await axios.get(`${api}`);
    console.log(result.data);
    const detail = result.data.features && result.data.features[0];
    if (detail) {
      alert(detail.properties.summary.distance/1000 + ',' + detail.properties.summary.duration / 60)
    }
  }
}
</script>
