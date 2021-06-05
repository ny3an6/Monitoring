<template>
  <div>
    <b-list-group>
      <b-list-group-item label="Spinning">MCC: {{this.mainInfo.mcc}}</b-list-group-item>
      <b-list-group-item label="Spinning">MNC: {{this.mainInfo.mnc}}</b-list-group-item>
      <b-list-group-item label="Spinning">LAC: {{this.mainInfo.lac}}</b-list-group-item>
      <b-list-group-item label="Spinning">Cell Id: {{this.mainInfo.cellId}}</b-list-group-item>
      <b-list-group-item label="Spinning">Номер Радиоканала: {{this.mainInfo.ch}}</b-list-group-item>
      <b-list-group-item label="Spinning">Уровень сигнала: {{this.mainInfo.rssi}}</b-list-group-item>
    </b-list-group>
  </div>
</template>

<script>
import Axios from "axios";

export default {
  name: "Main",
  data() {
    return {
      mainInfo: {},
    };
  },
  
  async created(){
    console.log("Hello")
    //setTimeout(() => { console.log("мир"); }, 3000);
    await Axios.get("http://localhost:8888")
      .then(response => {
        console.log(response.data);
        this.mainInfo = response.data;       
      })
      .catch(error => {
        console.log(error);
      })
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
