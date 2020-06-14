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
      ></v-data-table>
    </v-content>
  </v-app>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import axios from "axios";

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
    { text: "phone", value: "phone" }
  ];

  kindergardens = [];

  protected async created() {
    const url = "/data.json";
    const result = await axios.get(`${url}`);
    this.kindergardens = result.data.map((x: any) => {
      return {
        name: x.title,
        type: x.type,
        maxStudents: x.recruit,
        phone: x.tel,
        address: x.address
      }
    });
  }
}
</script>
