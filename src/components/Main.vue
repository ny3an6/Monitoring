<template>
  <div>
    <div>
      <b-card no-body>
        <b-tabs pills card vertical class="base_station_info">
          <b-tab title="Текущая сота" active>
            <b-card-text>
              <div>{{currentCell}}</div>
              <h3 class="p-3 text-center">Информация вашей текущей соты</h3>
              <b-list-group >
                <b-list-group-item label="Spinning">MCC: {{currentCell.mcc}}</b-list-group-item>
                <b-list-group-item label="Spinning">MNC: {{currentCell.mnc}}</b-list-group-item>
                <b-list-group-item label="Spinning">LAC: {{currentCell.lac}}</b-list-group-item>
                <b-list-group-item label="Spinning">Cell Id: {{currentCell.cell_id}}</b-list-group-item>
                <b-list-group-item label="Spinning">Номер Радиоканала: {{currentCell.ch}}</b-list-group-item>
                <b-list-group-item label="Spinning">Уровень сигнала: {{currentCell.rssi}}</b-list-group-item>
              </b-list-group>
              <!-- <b-button variant="danger" v-on:click="getMainInformation()" class="button-main-info">Обновить</b-button> -->
            </b-card-text>
          </b-tab>
            <b-tab title="Графики" ><b-card-text>Tab contents 2</b-card-text></b-tab>
            <b-tab title="Информация соседних сот">
              <b-card-text>
                    <div class="container">
                      <h3 class="p-3 text-center">Информация по соседним сотам</h3>
                      <table class="table table-striped table-bordered">
                        <thead>
                          <tr>
                            <th>CellId</th>
                            <th>CellName</th>
                            <th>LAC</th>
                            <th>Ch</th>
                            <th>Уровень сигнала</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr v-for="neighbor in neighbors" :key="neighbor.cellId">
                            <td>{{ neighbor.cell_id }}</td>
                            <td>{{ neighbor.cell_name }}</td>
                            <td>{{ neighbor.lac }}</td>
                            <td>{{ neighbor.ch }}</td>
                            <td>{{ neighbor.rssi }}</td>
                          </tr>
                        </tbody>
                      </table>
                      </div>
                        <!-- <b-button variant="danger" v-on:click="getNeighbotsInformation()" class="button">Обновить</b-button> -->
              </b-card-text>
            </b-tab>
        </b-tabs>
      </b-card>
    </div>
  </div>
</template>

<script>
import Axios from "axios";
// import Neighbors from "./Neighbors.vue";

export default {
  // components: { Neighbors },
  name: "Main",
  data() {
    return {
      mainInfo: {},
      neighbors: [],
      connection: null,
      currentCell:{}
    };
  },
  methods: {
    getNeighbotsInformation(){
      Axios.get("http://localhost:8888/neighbors")
        .then(response => {
          console.log(typeof(response.data));
          this.neighbors = response.data;        
        })
        .catch(error => {
          console.log(error);
        })
    },
    getMainInformation(){
      Axios.get("http://localhost:8888")
        .then(response => {
          console.log(typeof(response.data));
          this.mainInfo = response.data;        
        })
        .catch(error => {
          console.log(error);
        })
    },
    parsedSocketInformation(){
      if(this.socketInfo === null){
        return null;
      }
      else {
        return null
      }
    }
  },
    
  async created(){
    
    let vm = this;
    let connection = new WebSocket("ws://localhost:8888/ws/dblistener");

    connection.onopen = async function (event) {
      console.log(event);
      connection.send("subscribe measured_current_cell_info_insert")
      connection.send("subscribe measured_neighbors_cell_info_insert")
      console.log("Successfully connected to the echo websocket server...");
    };
    
    connection.onmessage = function (event) {
      if(JSON.parse(event.data) !== undefined){
      let receivedData = JSON.parse(event.data);

      setTimeout(() => {console.log("Wait 7 sec")}, 7000)
      for(let cell in vm.neighbors){
            console.log(cell)
            vm.neighbors.pop();
      }  
      switch(receivedData.channel){
        case "measured_current_cell_info_insert":
          vm.currentCell = receivedData.data;
          break;
        case "measured_neighbors_cell_info_insert":
          console.log(vm.neighbors)
          vm.neighbors.push(receivedData.data)
          break;
      }
      }
    };
    console.log("CURRENTCEL: " + vm.currentCell)
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .base_station_info{
    max-width: 1005px;
  }
  .button{
    position: relative;
    left: 23px;
  }
  .button-main-info{
    position: relative;
    left: 0;
    top: 12px;
  }
</style>
