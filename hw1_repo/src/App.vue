<script setup>
import ChoroplethMap from './components/ChoroplethMap.vue'
</script>

<template>
  <div class="wrap">
    <header>
      <h1>台灣各縣市稻米產量</h1>
      <p class="sub">2019 年糙米產量（公噸）· Vue 3 + D3.js Choropleth</p>
    </header>

    <ChoroplethMap />

    <footer>
      <p>資料來源：行政院農業委員會（農業部）《農產品生產量值統計—稻米產量》，2019 年糙米產量。地圖：g0v twgeojson 縣市界（經 mapshaper 簡化）。</p>
      <p class="note">＊ 南投、高雄、嘉義市為 2018 年官方數值（來源未逐項列出 2019）；基隆市與離島（澎湖、金門、連江）無稻作生產統計，以灰色表示。</p>
    </footer>
  </div>
</template>

<style>
:root { color-scheme: light; }
* { box-sizing: border-box; }
body {
  margin: 0; background: #fff; color: #1a1a1a;
  font-family: "Noto Sans TC", "PingFang TC", "Microsoft JhengHei", system-ui, sans-serif;
  -webkit-font-smoothing: antialiased;
}
.wrap { max-width: 840px; margin: 0 auto; padding: 28px 20px 48px; }
header h1 { font-size: 24px; margin: 0 0 4px; letter-spacing: .5px; }
header .sub { margin: 0 0 18px; color: #6b7280; font-size: 14px; }
footer { margin-top: 16px; color: #6b7280; font-size: 12px; line-height: 1.7; }
footer p { margin: 6px 0 0; }
</style>
