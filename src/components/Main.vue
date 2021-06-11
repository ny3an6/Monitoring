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
            </b-card-text>
          </b-tab>
            <b-tab title="Графики">
              <b-card-text>
                <LineChart :chartData="rssiCollection"/>
                <LineChart :chartData="berCollection"/>
              </b-card-text>
            </b-tab>
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
                            <th>Время проведения измерения</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr v-for="neighbor in neighbors" :key="neighbor.cellId">
                            <td>{{ neighbor.cell_id }}</td>
                            <td>{{ neighbor.cell_name }}</td>
                            <td>{{ neighbor.lac }}</td>
                            <td>{{ neighbor.ch }}</td>
                            <td>{{ neighbor.rssi }}</td>
                            <td>{{ neighbor.scan_date }}</td>
                          </tr>
                        </tbody>
                      </table>
                      </div>
              </b-card-text>
            </b-tab>
        </b-tabs>
      </b-card>
    </div>
  </div>
</template>

<script>

import LineChart from './LineChart.vue'

export default {
  name: "Main",
  data() {
    return {
      neighbors: [],
      currentNeighborCellDate:{},
      connection: null,
      currentCell:{},
      rssiCollection: {},
      berCollection: {},
      chartData: [],
      arraOfDate: null,
      arraOfBer: null,
      arrayOfRssi: null
    };
  },
  components:{
    LineChart
  },
  methods: {
    sendDataToCharts(){
      let dateResults = this.chartData.map(x=> x.scan_date)
      let rssiResults = this.chartData.map(x=> Number.parseInt(x.rssi.slice(0, -3)))
      let berResults = this.chartData.map(x=> x.ber)

      this.arraOfDate = dateResults,
      this.arrayOfRssi = rssiResults,
      this.arraOfBer = berResults

      this.rssiCollection = {
        labels: this.arraOfDate,
        datasets: [
          {
            label: 'Уровень сигнала',
            backgroundColor: 'transparent',
            borderColor: 'rgb(75, 192, 192)',
            pointBorderColor: '#f87979',
            data: this.arrayOfRssi
          }
        ]
      }
      this.berCollection = {
        labels: this.arraOfDate,
        datasets:[
          {
            label: 'BER',
            backgroundColor: 'transparent',
            borderColor: 'rgb(75, 192, 192)',
            pointBorderColor: '#f87979',
            data: this.arraOfBer
          }
        ]  
      }
      console.log(this.rssiCollection.datasets[0].data)
    }
  },
  mounted(){
    
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
      
      let receivedData = JSON.parse(event.data); 
      switch(receivedData.channel){
        case "measured_current_cell_info_insert":
          vm.currentCell = receivedData.data;
          console.log("SCAN_DATE" + vm.currentCell.scan_date)
          console.log("RSSI" + vm.currentCell.rssi)

          vm.chartData.push(vm.currentCell)
          console.log(vm.chartData)
          vm.sendDataToCharts()
          break;
        case "measured_neighbors_cell_info_insert":
          console.log(vm.neighbors)
      
          vm.currentNeighborCellDate = receivedData.data.scan_date;
          vm.neighbors = vm.neighbors.filter((item) => item.scan_date === vm.currentNeighborCellDate);
          vm.neighbors.push(receivedData.data);
          break;
      }     
    };
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
