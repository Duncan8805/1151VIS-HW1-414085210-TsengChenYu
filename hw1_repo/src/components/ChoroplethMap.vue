<script setup>
// Vue 3 (Composition API) + D3.js —— 以 D3 當 library 繪製 choropleth
import { ref, reactive, onMounted } from 'vue'
import * as d3 from 'd3'
import geo from '../data/taiwan-counties.json'
import { riceProduction, approxCounties, breaks, colors } from '../data/riceData.js'

const svgRef = ref(null)
// tooltip 用 Vue 響應式狀態，由 D3 事件更新（Vue 負責畫面、D3 負責地理運算）
const tip = reactive({ show: false, x: 0, y: 0, name: '', text: '' })

const W = 720, H = 760
const fmt = d3.format(',')

onMounted(() => {
  // 1) 資料結合（join）：把產量寫進每個 feature 的 properties
  geo.features.forEach(f => {
    f.properties.rice = riceProduction[f.properties.name] ?? null
    f.properties.approx = approxCounties.has(f.properties.name)
  })

  // 2) 投影 + 路徑產生器：台灣範圍小，用 geoMercator 自動置中縮放
  const projection = d3.geoMercator().fitExtent([[20, 20], [W - 20, H - 110]], geo)
  const path = d3.geoPath(projection)

  // 3) 色階：scaleThreshold 分級；null 給灰色
  const color = d3.scaleThreshold().domain(breaks).range(colors)
  const fillOf = d => (d == null ? '#d9dce1' : color(d))

  const svg = d3.select(svgRef.value)

  // 4) 畫縣市（data join）
  svg.append('g').selectAll('path')
    .data(geo.features)
    .join('path')
      .attr('class', 'county')
      .attr('d', path)
      .attr('fill', d => fillOf(d.properties.rice))
    .on('mousemove', (event, d) => {
      const p = d.properties
      tip.show = true
      tip.x = event.offsetX
      tip.y = event.offsetY
      tip.name = p.name
      tip.text = p.rice == null
        ? '無生產統計'
        : fmt(p.rice) + ' 公噸' + (p.approx ? '（2018＊）' : '')
    })
    .on('mouseleave', () => { tip.show = false })

  // 5) 主要產區標註（前 8 名）
  const topN = geo.features
    .filter(d => d.properties.rice != null)
    .sort((a, b) => b.properties.rice - a.properties.rice)
    .slice(0, 8)

  svg.append('g').selectAll('text')
    .data(topN)
    .join('text')
      .attr('class', 'label')
      .attr('transform', d => `translate(${path.centroid(d)})`)
      .call(t => t.append('tspan').attr('x', 0).attr('dy', '-0.1em').text(d => d.properties.name))
      .call(t => t.append('tspan').attr('class', 'v').attr('x', 0).attr('dy', '1.1em')
                  .text(d => fmt(d.properties.rice)))

  // 6) 圖例
  const lg = svg.append('g').attr('class', 'legend').attr('transform', `translate(30,${H - 60})`)
  lg.append('text').attr('class', 'caption').attr('y', -10).text('稻米產量（公噸）')
  const bw = 44
  lg.selectAll('g.cell').data(colors).join('g').attr('class', 'cell')
    .attr('transform', (d, i) => `translate(${i * bw},0)`)
    .call(g => g.append('rect').attr('width', bw).attr('height', 12).attr('fill', d => d))
    .call(g => g.append('text').attr('x', 0).attr('y', 26).text((d, i) => {
      if (i === 0) return '< ' + fmt(breaks[0])
      if (i === colors.length - 1) return '≥ ' + fmt(breaks[breaks.length - 1])
      return fmt(breaks[i - 1])
    }))
  const nd = lg.append('g').attr('transform', `translate(${colors.length * bw + 20},0)`)
  nd.append('rect').attr('width', 14).attr('height', 12).attr('fill', '#d9dce1')
  nd.append('text').attr('x', 18).attr('y', 11).text('無資料')
})
</script>

<template>
  <div class="chart">
    <svg ref="svgRef" :viewBox="`0 0 ${W} ${H}`" role="img"
         aria-label="台灣各縣市稻米產量分級著色地圖"></svg>
    <div v-if="tip.show" class="tip" :style="{ left: tip.x + 'px', top: tip.y + 'px' }">
      <b>{{ tip.name }}</b><br>稻米產量：{{ tip.text }}
    </div>
  </div>
</template>

<style scoped>
.chart { position: relative; }
svg {
  display: block; width: 100%; height: auto;
  background: #f7f8fa; border: 1px solid #e5e7eb; border-radius: 12px;
}
:deep(.county) { stroke: #fff; stroke-width: .7; transition: opacity .15s; }
:deep(.county:hover) { stroke: #111; stroke-width: 1.4; }
:deep(.label) {
  font-size: 11px; fill: #111; font-weight: 600; text-anchor: middle;
  paint-order: stroke; stroke: #fff; stroke-width: 2.6px; stroke-linejoin: round;
  pointer-events: none;
}
:deep(.label .v) { font-size: 9px; fill: #374151; font-weight: 400; }
:deep(.legend text) { font-size: 11px; fill: #1a1a1a; }
:deep(.legend .caption) { font-size: 12px; font-weight: 600; }
.tip {
  position: absolute; pointer-events: none; background: #111; color: #fff;
  font-size: 12px; padding: 7px 9px; border-radius: 7px; line-height: 1.4;
  white-space: nowrap; transform: translate(-50%, -120%);
  box-shadow: 0 4px 14px rgba(0,0,0,.25);
}
.tip b { font-size: 13px; }
</style>
