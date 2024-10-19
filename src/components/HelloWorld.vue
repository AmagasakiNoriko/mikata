<template>
  <div>
    <input type="file" @change="handleFileUpload">
    <h3 @click="addJMAEvent">気象庁観測点</h3>
    <table v-if="jmaData !== 0">
      <thead>
        <tr>
          <th v-for="(header, index) in tableHeaders" :key="index">{{ header }}</th>
        </tr>
      </thead>   
      <tbody>
        <tr v-for="(row, rowIndex) in jmaData" :key="rowIndex">
          <td v-for="(cell, cellIndex) in row" :key="cellIndex">
            <input
            v-if="cellIndex === 7 || cellIndex === 9"
            type="checkbox"
            >
          {{ cell }}
          </input>
          </td>
        </tr>
      </tbody>
    </table>

    <h3>防災科研観測点</h3>
    <table v-if="formattedExcelData">
      <thead>
        <tr>
          <th v-for="(header, index) in tableHeaders" :key="index">{{ header }}</th>
        </tr>
      </thead>   
      <tbody>
        <tr v-for="(row, rowIndex) in bousaikakenData" :key="rowIndex">
          <td v-for="(cell, cellIndex) in row" :key="cellIndex">
            <input
            v-if="cellIndex === 7 || cellIndex === 9"
            type="checkbox"
            >
          {{ cell }}
          </input>
          </td>
        </tr>
      </tbody>
    </table>    

    <h3>自治体観測点</h3>
    <table v-if="formattedExcelData">
      <thead>
        <tr>
          <th v-for="(header, index) in tableHeaders" :key="index">{{ header }}</th>
        </tr>
      </thead>   
      <tbody>
        <tr v-for="(row, rowIndex) in jititaiData" :key="rowIndex">
          <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
        </tr>
      </tbody>
    </table>    

    <h3>本日の予定</h3>
    <table v-if="formattedExcelData">
      <thead>
        <tr>
          <th v-for="(header, index) in tableHeaders" :key="index">{{ header }}</th>
        </tr>
      </thead>   
      <tbody>
        <tr v-for="(row, rowIndex) in todayData" :key="rowIndex">
          <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
        </tr>
      </tbody>
    </table>    
  </div>

  <h3 @click = "sendData">東京管内の観測点の読み込み</h3>  
</template>

<script>
import { ref } from "vue";
import * as XLSX from "xlsx";

//東京管内の県のデータ
const tokyoData = ["東京都","神奈川","埼玉","栃木","群馬","茨城","千葉","山梨","静岡","岐阜","長野","石川","富山","新潟"];
export default {
  data() {
    return {
      excelData: null,      // Excelから抽出したデータを格納
      tableHeaders: null,    // Excelのヘッダー情報を格納
      jmaData: [] ,  //気象庁観測点の情報を保存する変数
      bousaikakenData: null , //防災科学研究所のデータ
      jititaiData:null , //自治体データ
      formattedExcelData: null,    //フォーマットされたエクセルデータ
      ncSagyou:null , //NC作業ありデータ
      keiketusagyou:null, //計画欠測ありデータ
      tokyokannnai: null, //東京管内の観測点のデータ
      todayData:null
    };
  },

  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const data = new Uint8Array(e.target.result);
          const workbook = XLSX.read(data, { type: "array" });

          // 最初のシートを取得
          const firstSheetName = workbook.SheetNames[0];
          const worksheet = workbook.Sheets[firstSheetName];

          // シートデータをJSON形式に変換
          const rawData = XLSX.utils.sheet_to_json(worksheet, { header : 1 });

          // フォーマット処理を適用
          this.formattedExcelData = this.applyFormatting(rawData);
         
          //フォーマットされたデータに対して
          //気象庁の観測点のみをピックアップ
          // this.jmaData = this.formattedExcelData.filter(item => item.includes('気象庁'));
          //防災科研の観測点をピックアップ
          this.bousaikakenData = this.formattedExcelData.filter(item => item.includes('防災科研'));
          //自治体観測点をピックアップ
          this.jititaiData = this.formattedExcelData.filter(item => item.includes('自治体'));
          //NC作業あり観測点をピックアップ
          this.ncSagyou = this.formattedExcelData.filter(row => row[4]=== 'あり');
          //計画欠測あり作業をピックアップ
          this.keiketusagyou = this.formattedExcelData.filter(row => row[5]=== 'あり');
          //東京管内の予定をピックアップ
          this.tokyokannnai = this.formattedExcelData.filter(row => tokyoData.some(word => row[2].includes(word)));
          // 本日の日付取得
          const today = new Date().toLocaleDateString('ja-JP').replace(/\//g, '/');
          // console.log (today);
          //本日の予定をフィルタリング
          this.todayData = this.formattedExcelData.filter(item=>{
            const startDate = item[6];
            const endDate = item[8];
            // console.log(startDate);
            return today >=startDate && today <=endDate;
          });
          //console.log(todayData);

          // データが存在する場合、ヘッダー情報を取得
          if (rawData.length) {
            this.tableHeaders = rawData[0]; // ヘッダー行を取得
            this.excelData = rawData;                   // データ本体を設定
            // console.log(this.excelData);
            // this.jmaData = this.excelData.filter(item => item.観測点の所属 === "気象庁"); //気象庁観測点のみを抜き出し
            //this.formattedjmaData = this.applyFormattnig(jmaData); //気象庁観測点のデータにフォーマット処理を適用
          }
        };
        reader.readAsArrayBuffer(file); //ファイルをarrayバッファーとして読み込む
      }
    },

    addJMAEvent(){
      if(this.jmaData.length != 0){
        this.jmaData = [];
      }else{
      this.jmaData = this.formattedExcelData.filter(item => item.includes('気象庁'));
      }
    },

    sendData(){
      this.$emit('sendData', this.tokyokannnai,  this.tableHeaders);
    },
    
    applyFormatting(data){
      const headers = data[0]; //一行目はヘッダー
      const formattedData = data.map((row,rowIndex) => {
        //　一行目はヘッダなのでスキップ
        // if(rowIndex === 0) return row ;
        return row.map((cell,colIndex)=> {
          const header = headers[colIndex]; //列のヘッダー情報を取得
          return this.formatCell(cell,header); // formatCellでフォーマットを実行
        });
      });
      return formattedData
    },

    formatCell(value, header) {
     // 観測点番号の列は日付処理を行わない 
     if(header === '観測点番号'){
        return value;
      }
      // 日付のフォーマット処理（列名に応じて条件を設定）
      if (typeof value === 'number') {
        // シリアル値を日付として解釈できるか確認
        const date = XLSX.SSF.parse_date_code(value);    
        if (date) {
          //シリアル値が小数点のみの場合（時刻）
          if(Math.floor(value)===0) {
            //時刻のフォーマット
            const hours = date.H;
            const minutes = date.M;
            const seconds = date.S;
            return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
          } else {          
          // 日付のフォーマット（YYYY-MM-DD）
          return new Date(Date.UTC(date.y, date.m - 1, date.d)).toLocaleDateString();
          }
        }
       }
      // その他の値はそのまま返す
      return value || '';
     },
  },
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
</style>
