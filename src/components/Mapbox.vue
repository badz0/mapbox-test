<template>
<div>
  <div id="map"></div>
  <div class="scores">
    <p v-for="{val, key} in scores" :key="key">
      {{key}}: {{val}}
    </p>
    <button class="export" @click="exportMarkers">Export</button>
  </div>
</div>
</template>

<script>
import mapboxgl from 'mapbox-gl';
import '../../node_modules/mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_KEY;
const markerColors = ['black', 'gray', 'red', 'orange', 'lime', 'green'];

export default {
  name: 'Mapbox',
  data() {
    return {
      mapbox: null,
      markers: [],
    };
  },
  computed: {
    scores() {
      const scores = [
        { key: 'Zero', val: 0 },
        { key: 'One', val: 0 },
        { key: 'Two', val: 0 },
        { key: 'Three', val: 0 },
        { key: 'Four', val: 0 },
        { key: 'Five', val: 0 },
        { key: 'Total', val: this.markers.length },
      ];

      this.markers.forEach(item => scores[item.score].val++);
      scores.reverse();
      return scores;
    },
  },

  mounted() {
    this.mapbox = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v9',
    });

    this.mapbox.on('click', (ev) => {
      if (ev.originalEvent.target.closest('.mapboxgl-marker')) return;
      this.addMarker(ev.lngLat);
    });
  },

  methods: {
    addMarker(lngLat, newScore) {
      const score = typeof newScore === 'number' ? newScore : this.getRandomScore();

      const marker = new mapboxgl.Marker({
        draggable: true,
        color: markerColors[score],
      }).setLngLat(lngLat)
        .addTo(this.mapbox);

      marker.score = score;
      marker.setPopup(this.createPopup(marker));
      this.markers.push(marker);
    },

    getRandomScore() {
      return Math.floor(Math.random() * markerColors.length);
    },

    createPopup(marker) {
      const content = document.createElement('div');
      content.className = 'popup-content';
      content.innerHTML = this.popupRadioTemplate(marker.score) + this.removeBtnTemplate();

      content.querySelectorAll('.score-radio input').forEach((el) => {
        el.addEventListener('change', (ev) => {
          marker.remove();
          this.markers = this.markers.filter(item => item !== marker);
          this.addMarker(marker._lngLat, +ev.target.value);
        });
      });

      content.querySelector('.remove-marker').addEventListener('click', () => {
        marker.remove();
        this.markers = this.markers.filter(item => item !== marker);
      });

      return new mapboxgl.Popup({ offset: 35 }).setDOMContent(content);
    },

    removeBtnTemplate() {
      return '<button class="remove-marker">Remove</button>';
    },

    popupRadioTemplate(score) {
      return markerColors.map((_, i) => `
        <label class="score-radio">
          <input type="radio" name="score" value="${i}" ${score === i ? 'checked' : ''}>
          ${i}
        </label>
      `).join('');
    },

    exportMarkers() {
      const data = this.markers.map(item => ({
        score: item.score,
        lngLat: item._lngLat,
        color: item._color,
      }));
      const jsonStr = JSON.stringify(data);

      const element = document.createElement('a');
      element.setAttribute('href', `data:text/plain;charset=utf-8, ${encodeURIComponent(jsonStr)}`);
      element.setAttribute('download', 'markers.json');

      element.style.display = 'none';
      document.body.appendChild(element);

      element.click();

      document.body.removeChild(element);
    },
  },
};
</script>

<style scoped>
.scores {
  background: rgba(0, 0, 0, 0.5);
  color: #fff;
  position: absolute;
  top: 20px;
  right: 20px;
  padding: 5px 10px;
  font-size: 14px;
  border-radius: 3px;
}
.export {
  margin-top: 10px;
}
</style>
<style>
.score-radio {
  display: inline-flex;
}
.remove-marker {
  display: block;
  margin: auto;
}
</style>
