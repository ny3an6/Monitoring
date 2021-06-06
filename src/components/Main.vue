<template>
  <div>
    <div>
      <b-card no-body>
        <b-tabs pills card vertical class="base_station_info">
          <b-tab title="Current Cell" active>
            <b-card-text>
              <b-list-group >
                <b-list-group-item label="Spinning">MCC: {{this.mainInfo.mcc}}</b-list-group-item>
                <b-list-group-item label="Spinning">MNC: {{this.mainInfo.mnc}}</b-list-group-item>
                <b-list-group-item label="Spinning">LAC: {{this.mainInfo.lac}}</b-list-group-item>
                <b-list-group-item label="Spinning">Cell Id: {{this.mainInfo.cellId}}</b-list-group-item>
                <b-list-group-item label="Spinning">Номер Радиоканала: {{this.mainInfo.ch}}</b-list-group-item>
                <b-list-group-item label="Spinning">Уровень сигнала: {{this.mainInfo.rssi}}</b-list-group-item>
              </b-list-group>
              <b-button variant="danger" v-on:click="getMainInformation()" class="button-main-info">Обновить</b-button>
            </b-card-text>
          </b-tab>
            <b-tab title="Tab 2" ><b-card-text>Tab contents 2</b-card-text></b-tab>
            <b-tab title="Neighbours Information" >
              <b-card-text>
                    <div class="container">
                      <h3 class="p-3 text-center">Информаци по соседним сотам</h3>
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
                                  <td>{{neighbor.cellId}}</td>
                                  <td>{{neighbor.cellName}}</td>
                                  <td>{{neighbor.lac}}</td>
                                  <td>{{neighbor.ch}}</td>
                                  <td>{{neighbor.rssi}}</td>
                              </tr>
                          </tbody>
                      </table>
                   </div>
                   <b-button variant="danger" v-on:click="getNeighbotsInformation()" class="button">Обновить</b-button>
              </b-card-text>
            </b-tab>
        </b-tabs>
      </b-card>
    </div>
  </div>
</template>

<script>
import Axios from "axios";

export default {
  name: "Main",
  data() {
    return {
      mainInfo: {},
      fieldsNeighbors: ['cellName', 'cellId', 'lac', 'ch', 'rssi'],
      neighbors: {}
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
    }
  },
    
  async created(){
    console.log("Hello")
    await Axios.get("http://localhost:8888")
      .then(response => {
        console.log(response.data);
        this.mainInfo = response.data;       
      })
      .catch(error => {
        console.log(error);
      })
      Axios.get("http://localhost:8888/neighbors")
        .then(response => {
          console.log(typeof(response.data));
          this.neighbors = response.data;        
          for ( let key in this.neighbors){
            console.log(Object.keys(this.neighbors[key]))
            for(let i in this.neighbors[key]){
              console.log(i);
              console.log(typeof(i));   
            }
          }
        })
        .catch(error => {
          console.log(error);
        })
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
