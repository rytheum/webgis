<script>
  import { onMount, onDestroy } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import {
    basemaps,
    MVT_TILES,
    MVT_SOURCE_LAYER,
    MAP_CENTER,
    MAP_ZOOM
  } from './config.js';

  export let basemap = 'street';

  let container;
  let map;
  let appliedBasemap = basemap;
  let popup;
  let layerMode = 'none';
  export let layerVisible = true;

  const SOURCE_ID = 'suppliers';
  const LAYER_ID = 'suppliers-circles';

  const circlePaint = {
    'circle-radius': ['interpolate', ['linear'], ['zoom'], 6, 3, 12, 7, 16, 10],
    'circle-color': '#10b981',
    'circle-stroke-width': 1.5,
    'circle-stroke-color': 'rgba(255,255,255,0.7)',
    'circle-opacity': ['interpolate', ['linear'], ['zoom'], 6, 0.7, 12, 0.9],
  };

  function escapeHtml(value) {
    return String(value ?? '—')
      .replaceAll('&', '&amp;')
      .replaceAll('<', '&lt;')
      .replaceAll('>', '&gt;');
  }

  function formatCoord(num) {
    return Number(num).toFixed(5);
  }

  function onSupplierClick(event) {
    const feat = event.features?.[0];
    const props = feat?.properties;
    if (!props) return;

    const area = props.plotareaha ?? '—';
    const name = escapeHtml(props.entityname);
    const region = escapeHtml(props.regionlabel);
    const country = escapeHtml(props.countryname);
    const lng = formatCoord(event.lngLat.lng);
    const lat = formatCoord(event.lngLat.lat);

    const html = `
      <div class="popup-header">
        <div class="popup-header-icon">🌿</div>
        <div class="popup-header-text">
          <span class="popup-header-label">Supplier</span>
          <span class="popup-header-name" title="${name}">${name}</span>
        </div>
      </div>
      <div class="popup-body">
        <div class="popup-row">
          <div class="popup-row-icon">📐</div>
          <div class="popup-row-content">
            <span class="popup-row-label">Plot Area</span>
            <span class="popup-row-value">${escapeHtml(area)} ha</span>
          </div>
        </div>
        <div class="popup-row">
          <div class="popup-row-icon">📍</div>
          <div class="popup-row-content">
            <span class="popup-row-label">Region</span>
            <span class="popup-row-value">${region}</span>
          </div>
        </div>
        <div class="popup-row">
          <div class="popup-row-icon">🌍</div>
          <div class="popup-row-content">
            <span class="popup-row-label">Country</span>
            <span class="popup-row-value">${country}</span>
          </div>
        </div>
      </div>
      <div class="popup-footer">
        <span class="popup-coord-icon">📌</span>
        <span class="popup-coord">${lng}, ${lat}</span>
      </div>
    `;

    popup
      .setLngLat(event.lngLat)
      .setHTML(html)
      .addTo(map);
  }

  function onPointer() {
    map.getCanvas().style.cursor = 'pointer';
  }

  function onPointerOut() {
    map.getCanvas().style.cursor = '';
  }

  function clearSupplierLayer() {
    if (!map.getStyle()) return;
    if (map.getLayer(LAYER_ID)) {
      map.off('click', LAYER_ID, onSupplierClick);
      map.off('mouseenter', LAYER_ID, onPointer);
      map.off('mouseleave', LAYER_ID, onPointerOut);
      map.removeLayer(LAYER_ID);
    }
    if (map.getSource(SOURCE_ID)) map.removeSource(SOURCE_ID);
  }

  function bindClicks() {
    map.on('click', LAYER_ID, onSupplierClick);
    map.on('mouseenter', LAYER_ID, onPointer);
    map.on('mouseleave', LAYER_ID, onPointerOut);
  }

  // setStyle drops custom layers. style.load calls this again for the active mode.
  function applyLayer() {
    if (!map.getStyle()) return;
    clearSupplierLayer();
    if (layerMode !== 'mvt') return;
    map.addSource(SOURCE_ID, {
      type: 'vector',
      tiles: [MVT_TILES],
      minzoom: 0,
      maxzoom: 14
    });
    map.addLayer({
      id: LAYER_ID,
      type: 'circle',
      source: SOURCE_ID,
      'source-layer': MVT_SOURCE_LAYER,
      paint: circlePaint,
      layout: {
        visibility: layerVisible ? 'visible' : 'none'
      }
    });
    bindClicks();
  }

  export function useMvt() {
    layerMode = 'mvt';
    applyLayer();
  }

  export function toggleLayer(visible) {
    layerVisible = visible;
    if (!map || !map.getStyle()) return;
    if (map.getLayer(LAYER_ID)) {
      map.setLayoutProperty(LAYER_ID, 'visibility', visible ? 'visible' : 'none');
    }
    if (!visible) {
      popup?.remove();
    }
  }

  onMount(() => {
    popup = new maplibregl.Popup({ closeButton: true, closeOnClick: true, maxWidth: '300px' });
    map = new maplibregl.Map({
      container,
      style: basemaps[basemap],
      center: MAP_CENTER,
      zoom: MAP_ZOOM
    });
    map.addControl(new maplibregl.NavigationControl(), 'top-right');
    map.addControl(new maplibregl.ScaleControl({ maxWidth: 150 }), 'bottom-left');
    map.on('style.load', applyLayer);
  });

  // diff:false makes style.load fire again. The default diff drops custom layers quietly.
  $: if (map && basemap !== appliedBasemap) {
    appliedBasemap = basemap;
    popup?.remove();
    map.setStyle(basemaps[basemap], { diff: false });
  }

  onDestroy(() => {
    popup?.remove();
    map?.remove();
  });
</script>

<div class="map" bind:this={container}></div>

<style>
  .map {
    flex: 1;
    height: 100%;
    min-height: 400px;
  }
</style>
