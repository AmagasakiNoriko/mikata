<template>
  <v-app>
    <v-main>
      <HelloWorld @send-data="receiveData" ></HelloWorld>
      <table v-if="rTkData">
      <thead>
        <tr>
          <th v-for="(header, index) in rTHHeaders" :key="index">{{ header }}</th>
        </tr>
      </thead>   
      <tbody>
        <tr v-for="(row, rowIndex) in rTkData" :key="rowIndex" @click="handleClick(row)">
          <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
        </tr>
      </tbody>
      </table>  

  <div id="app">
    <h3>その他観測点の表示</h3>
 

  <!-- モーダル -->
  <div class="modal" tabindex="-1" role="dialog" v-if="showModal">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">データを入力</h5>
          <button type="button" class="close" @click="showModal = false" aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <input v-model="newItem.name" placeholder="観測点名を入力" class="form-control">
          <input v-model="newItem.affiliate" placeholder="観測点の所属を入力" class="form-control">
          <input v-model="newItem.prefecture" placeholder="観測点の都道府県" class="form-control">
          <input v-model="newItem.number" placeholder="観測点番号" class="form-control">
          <input v-model="newItem.NCteishie" placeholder="NC停止" class="form-control">
          <input v-model="newItem.keiketu" placeholder="計画欠測" class="form-control">
          <input v-model="newItem.startDay" placeholder="開始日" class="form-control">
          <input v-model="newItem.startClock" placeholder="開始時刻" class="form-control">
          <input v-model="newItem.endDay" placeholder="終了日時" class="form-control">
          <input v-model="newItem.endClock" placeholder="終了時刻" class="form-control">
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" @click="showModal = false">キャンセル</button>
          <button type="button" class="btn btn-primary" @click="addItem">追加</button>
        </div>
      </div>
    </div>
  </div>

     <!-- 表 -->
     <table class="table mt-3">
      <thead>
        <tr>
           <th v-for="(header, index) in rTHHeaders" :key="index">{{ header }}</th>
        </tr>
      </thead>
      <tbody>
       <tr v-for="(item, indo) in items" :key="indo">
         <td>{{ item.name }}</td>
         <td>{{ item.affiliate }}</td>
         <td>{{ item.prefecture }}</td>
         <td>{{ item.number }}</td>
         <td>{{ item.NCteishi }}</td>
         <td>{{ item.keiketu }}</td>
         <td>{{ item.startDay }}</td>
         <td>{{ item.startClock }}</td>
         <td>{{ item.endDay }}</td>
         <td>{{ item.endClock }}</td>
       </tr>
      </tbody>
     </table>

  <button class="btn btn-primary" @click="showModal = true">データを追加</button>

  </div>

      <ObStaMap :drawMapData="selectedItem" ref="childRef"></ObStaMap>
    </v-main>
  </v-app>
</template>

<script >
import HelloWorld from './components/HelloWorld.vue';
import ObStaMap from './components/ObStaMap.vue';

export default{
  data(){
    return{
      rTkData: '',
      rTHHeaders: '',
      selectedItem : null,
      showModal: false,
      newItem: { 
        name: '', 
        affiliate: '',
        prefecture: '', 
        number: '',
        NCteishi:'',
        keiketu:'',
        startDay:'',
        startClock:'',
        endDay:'',
        endClock:''
       },
      items: []
    };
  },

  methods:{
   receiveData(data1,data2){
    this.rTkData = data1;
    this.rTHHeaders = data2;
   },
   handleClick(row){
    this.selectedItem = row;
    this.$refs.childRef.plotMarkersOnMap();
   },
   addItem() {
        if (this.newItem.name !== '' ) {
          this.items.push({ ...this.newItem });
          this.newItem = { 
            name: '', 
            affiliate: '',
            prefecture: '', 
            number: '',
            NCteishi:'',
            keiketu:'',
            startDay:'',
            startClock:'',
            endDay:'',
            endClock:''
          };
          this.showModal = false;
        }

    }
  }  
};

</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}
th, td {
  border: 1px solid #ddd;
  padding: 8px;
}
th {
  background-color: #0000FF;
  text-align: left;
}
button {
    background-color: #37393b;
    color: white;
    height: 25px;
    width: 100%;
    border: 1px solid;
    border-radius: 5px;
    cursor: pointer;
    font-size: 15px;
  }

  .custom-button:hover {
    background-color: #494b4d;
  }
</style>
