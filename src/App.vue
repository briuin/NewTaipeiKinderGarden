<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-spacer></v-spacer>
      <span class="mr-2">New Taipei Kindergardens</span>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-content>
      <v-row align="center">
        <v-spacer></v-spacer>
        <v-col class="d-flex" cols="12" sm="6">
          <v-select
            :items="getAllDist()"
            v-model="selectedDist"
            label="區"
            dense
          ></v-select>
        </v-col>
        <v-col class="d-flex" cols="12" sm="6">
          <v-select
            :items="getAllType()"
            v-model="selectedType"
            label="類型"
            dense
          ></v-select>
        </v-col>
        <v-spacer></v-spacer>
      </v-row>
      <v-row align="center">
        <v-col class="d-flex" cols="12" sm="12">
          <v-btn @click="calculateAllMatched()">計算開車路徑</v-btn>
        </v-col>
      </v-row>
      <v-data-table
        :headers="headers"
        :items="filteredKindergardens"
        :items-per-page="perPage"
        class="elevation-1"
      >
        <template v-slot:item.direction="{ item }">
          <v-btn v-if="!item.direction" @click="calculateDirectionFrom(item)">GET</v-btn>
          <div v-else>
            <div>距離: {{ item.direction.kilometer.toFixed(2) }}公里</div>
            <div>時間: {{ item.direction.minute.toFixed(2) }}分鐘</div>
          </div>
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

interface Coordinate {
  lat: number;
  lng: number;
}

interface Kindergarden {
    name: string;
    type: string;
    maxStudents: string;
    phone: string;
    address: string;
    location: Coordinate;
    direction: {
      kilometer: number;
      minute: number;
    };
  }

@Component
export default class App extends Vue {
  perPage = 10;
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
    { text: "direction", value: "direction" },
  ];

  kindergardens: Array<Kindergarden> = [];

  locations: Array<{
    name: string;
    location: Coordinate;
    address: string;
  }> = [];

  currentPosition: Coordinate | null = null;

  selectedType = "";
  selectedDist = "";

  get filteredKindergardens() {
    return this.kindergardens.filter(
      (x) =>
        (this.selectedType === "" || this.selectedType === x.type) &&
        (this.selectedDist === "" || x.address.includes(this.selectedDist))
    );
  }

  getAllType() {
    const result: any = {};
    this.kindergardens.forEach((x) => {
      result[x.type] = true;
    });
    return [
      { text: "All", value: "" },
      ...Object.keys(result).map((x) => ({ text: x, value: x })),
    ];
  }

  getAllDist() {
    const result: any = {};
    this.kindergardens.forEach((x) => {
      const dist = x.address.split("區")[0].split("市")[1];
      result[dist] = true;
    });
    return [
      { text: "All", value: "" },
      ...Object.keys(result).map((x) => ({ text: x, value: x })),
    ];
  }

  async calculateAllMatched() {
    if (!this.currentPosition) {
      alert('Please allow location permission');
      return;
    }

    this.filteredKindergardens.forEach(async (x, i) => {
      if (x.direction || i >= this.perPage) {
        return;
      }
      const detail = await this.getDirection(this.currentPosition!, x.location);
      x.direction = {
        kilometer: detail.distance,
        minute: detail.duration
      };
      Vue.set(this.kindergardens, i, x);
    })
  }

  async calculateDirectionFrom(item: Kindergarden) {
    if (!this.currentPosition) {
      alert('Please allow location permission');
      return;
    }
    const detail = await this.getDirection(this.currentPosition, item.location);
    item.direction = {
      kilometer: detail.distance,
      minute: detail.duration
    };

    const kindergardenIndex = this.kindergardens.findIndex(x => x.name === item.name);
    if (kindergardenIndex >= 0) {
      Vue.set(this.kindergardens, kindergardenIndex, item);
    }
  }

  protected async created() {
    this.getUserPosition();
    const url = "./data.json";
    const result = await axios.get(`${url}`);
    const locationsResult = await axios.get(`./location.json`);
    this.kindergardens = result.data.map((x: HttpResponse) => {
      const location = locationsResult.data.find(
        (l: LocationHttpResponse) => l.name === x.title
      );
      return {
        name: x.title,
        type: x.type,
        maxStudents: x.recruit,
        phone: x.tel,
        address: x.address,
        location: location && location.location,
      };
    });
  }

  private async getDirection(start: Coordinate, end: Coordinate) {
    const key = `5b3ce3597851110001cf624879337138a41d4de8ab6a36f2ea031112`;
    const api = `https://api.openrouteservice.org/v2/directions/driving-car?api_key=${key}&start=${start.lng},${start.lat}&end=${end.lng},${end.lat}`;
    const result = await axios.get(`${api}`);

    const detail = result.data.features && result.data.features[0];
    if (detail) {
      return {
        distance: detail.properties.summary.distance / 1000,
        duration: detail.properties.summary.duration / 60
      }
    }

    return {
        distance: 0,
        duration: 0
      }
  }

  private getUserPosition() {
    navigator.geolocation.getCurrentPosition((position) => {
      this.currentPosition = {
        lat: position.coords.latitude,
        lng: position.coords.longitude,
      };
    });
  }
}
</script>
