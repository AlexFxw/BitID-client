<template>
  <div class="outer">
    <div class="inter">
      <el-tooltip effect="dark" content="Confirm" placement="top">
        <el-button class="button1" type="success" icon="el-icon-check" round v-on:click="Confirm">Refresh</el-button>
      </el-tooltip>
      <el-tooltip effect="dark" content="Stop" placement="top">
        <el-button class="button2" type="danger" icon="el-icon-close" round v-on:click="StopUpdate">Stop</el-button>
      </el-tooltip>
      <el-tooltip effect="dark" content="Add a new tag" placement="top">
        <el-button class="button3"  type="primary" icon="el-icon-edit" circle v-on:click="DefineTag"></el-button>
      </el-tooltip>
      <el-tooltip effect="dark" content="delete a tag" placement="top">
        <el-button class="button4"  type="danger" icon="el-icon-delete" circle v-on:click="DeleteTag"></el-button>
      </el-tooltip>
    </div>
    <!-- <div class = "second">
    <objList class="myobjlist" @selectList="getSelected"></objList>
    </div> -->
    <div class="vfor">
      <el-table
      :data="tableData"
      style="width: 100%"
      @selection-change="handleSelectionChange">
    <el-table-column
      type="selection"
      width="55">
    </el-table-column>
      <el-table-column
        prop="name"
        label="Object"
        width="180"
        :filters="namefilter"
        :filter-method="filterTag">
      </el-table-column>
      <el-table-column
        prop="status"
        label="Status">
      </el-table-column>
    </el-table>
      <!-- <el-card class="box-card">
        <div v-for="item in items" :key="item" class="text item">
          {{ item }}
        </div>
      </el-card> -->
    <!-- <li v-for="item in items">{{ item }}</li> -->
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { tagData, serverHost, CONFIG } from "../global";
import router from "../router/index";
import ObjectList from "./ObjectList.vue";
import axios from 'axios';
import VueAxios from 'vue-axios';
Vue.component("objList", ObjectList);

export default Vue.extend({
  data() {
    return {
      selectedObj: "",
      items: {},
      namefilter:[],
      stopHandler: undefined,
      options:[],
      tableData: [],
      multipleSelection: []
    };
  },
  methods: {
    getSelected: function(msg) {
      this.selectedObj = msg;
    },
    Refresh: function() {
      let xhr = new XMLHttpRequest();
      xhr.open("GET", serverHost + "/get-objects", true);
      let vm = this;
      xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
          let response = JSON.parse(xhr.responseText)
          vm.$data.options = response['objects'];
        }
      };
      xhr.send(null);
    },
    handleDelete: function(index, row){
      alert(index);
      alert(row);
    },
    getObjectState: function() {
      let vm = this;
      let xhr = new XMLHttpRequest();
      xhr.open("GET", CONFIG.GET_ALL_OBJ, true);
      xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
          let response = JSON.parse(xhr.responseText);
          for(let s in response){
            console.log(s);
            console.log(response[s])
            console.log(response[s][0])
            if(response[s][0] !== 'undetected') {
              vm.$data.items[s] = response[s][0];
            }
          }
        }
      }
      xhr.send(null)
      // console.log(this.$data.items);
      this.tableData = [];
      var tempdic={};
      for (var i=0;i<this.options.length;i++) {
        tempdic={};
        tempdic['name'] = this.options[i];
        tempdic['status'] = this.items[this.options[i]];
        this.tableData.push(tempdic);
      }
      // console.log(this.tableData);
      var name = "";
      this.namefilter = []
      for (var i=0;i<this.options.length;i++) {
        tempdic={};
        tempdic['text'] = this.options[i];
        tempdic['value'] = this.options[i];
        this.namefilter.push(tempdic);
      }
    },
    Confirm: function() {
      this.Refresh();
      this.StopUpdate();
      this.stopHandler = setInterval(this.getObjectState, CONFIG.UPDATE_INTERVAL);
    },
    StopUpdate: function(){
      if(this.stopHandler !== undefined) {
        clearInterval(this.stopHandler);
        this.stopHandler = undefined;
      }
    },
    DefineTag: function() {
      this.StopUpdate();
      router.push({name: 'Form'});
    },
     DeleteTag: function() {
       var item;
       console.log("Delete!!!!");
       var name;
      //  console.log(this.multipleSelection)
       for(item in this.multipleSelection) {
          // name = this.tableData[item];
          var data = {'objName':this.multipleSelection[item]['name']};
          this.axios
            .post(serverHost + "/remove-object", this.qs.stringify(data))
            .then(res => {
              console.log("删除成功");
            })
            .catch(err => {
              console.log(err);
            });
       }
    },
    created(){
      this.Refresh();
    },
    filterTag(value, row) {
      return row.name === value;
    },
    handleSelectionChange(val) {
        this.multipleSelection = val;
      }
  }
});
</script>

<style scoped>
  .outer{
    margin-top: 5%;
    text-align: center
  }
  .vfor{
    margin-top:1%;
    margin-left: 30%;
    width:600px;
  }
  .button1{
    width: 150px;
    margin-right: 50px;
    margin-bottom: 50px;
  }
  .button2{
    width: 150px;
  }
  .button3{
    margin-left: 12%;
  }
  .second{
  }
  .inter{
    margin-right: 0%; 
  }
  .text {
    font-size: 20px;
  }

  .item {
    padding: 5px 0;
  }
  .box-card {
    height:480px;
    text-align: left;
    padding-left: 10%;
  }
</style>
