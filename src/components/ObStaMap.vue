<template>
	<div>
	  <input type="file" @change="handleFileUpload" />
	  <div id="map" style="height: 500px;"></div>
	</div>
  </template>
  
  <script setup>
  import { onMounted, ref } from 'vue';
  import Papa from 'papaparse';
  import L from 'leaflet';
  import 'leaflet/dist/leaflet.css';
  import redMarker from '@/assets/赤色マーカー.jpg';
  
//親コンポーネントから変数の受け取り
const props = defineProps(['drawMapData']);
// console.log('子でのセレクトアイテム',props.drawMapData);

  // Leaflet のマップオブジェクト
  let map;
  const csvData = ref([]);
  
  // CSV 読み込みハンドラー
  const handleFileUpload = (event) => {
	const file = event.target.files[0];
	if (file) {
	  const reader = new FileReader();
	  reader.onload = (e)=> {
		const arrayBuffer = e.target.result;
		const decoder = new TextDecoder('shift-jis');
		const csvText = decoder.decode(arrayBuffer);

		Papa.parse(csvText, {
		header: true,
		skipEmptyLines: true,
		complete: (result) => {
		  csvData.value = result.data;
		  plotMarkersOnMap(); // マーカーを地図上にプロット
		  // console.log(csvData);
		},
		error: (error) => {
		  console.error('CSVの読み込みに失敗しました:', error);
		}
	  });
	  };
	  reader.readAsArrayBuffer(file);
	}
  };
  
// 赤いアイコンの定義
var redIcon = L.icon({
    iconUrl: redMarker,
    shadowUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-shadow.png',
    iconSize: [25, 41],
    iconAnchor: [12, 41],
    popupAnchor: [1, -34],
});

// 地図にマーカーをプロットする関数
const plotMarkersOnMap = () => {
    csvData.value.forEach((item) => {
        const lat = parseFloat(item['緯度']);
        const lng = parseFloat(item['経度']);
        const name = item['観測点名'];
        
	//	console.log("csvの値",name);
	//	console.log("選択データのもの",props.drawMapData[0]);
	//	console.log("真偽", name === props.drawMapData[0]);

        if (props.drawMapData[0] === name) {
            if (!isNaN(lat) && !isNaN(lng)) {
                L.marker([lat, lng], { icon: redIcon })
               .addTo(map)
               .bindPopup(`<b>${name}</b><br>緯度: ${lat}, 経度: ${lng}`);
            }
        } else {
            if (!isNaN(lat) && !isNaN(lng)) {
                L.marker([lat, lng])
                    .addTo(map)
                    .bindPopup(`<b>${name}</b><br>緯度: ${lat}, 経度: ${lng}`);
            }
        }
    });
};

defineExpose({plotMarkersOnMap}); // この条件式を親へ渡す
  
  // マップの初期化
  onMounted(() => {
	map = L.map('map').setView([35.6895, 139.6917], 5); // 初期表示の座標とズームレベル
  
	// タイルレイヤーを追加
	L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
	  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
	}).addTo(map);
  });

  </script>
  
  <style scoped>
  #map {
	height: 500px;
  }
  </style>
  