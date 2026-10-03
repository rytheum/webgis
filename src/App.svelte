<script>
  import MapView from './lib/MapView.svelte';

  let basemap = 'street';
  let mapView;
  let layerActive = false;
  let layerVisible = true;

  function toggleMvt() {
    if (!layerActive) {
      mapView.useMvt();
      layerActive = true;
      layerVisible = true;
    }
  }

  function handleLayerToggle() {
    if (!layerActive) {
      // First toggle ON → load the MVT data
      mapView.useMvt();
      layerActive = true;
      layerVisible = true;
    } else {
      // Already loaded → toggle visibility
      layerVisible = !layerVisible;
      mapView.toggleLayer(layerVisible);
    }
  }

  const options = [
    { id: 'street',    label: 'MAPID Street 2D',       icon: '🗺️' },
    { id: 'light',     label: 'MAPID Light',            icon: '☀️' },
    { id: 'satellite', label: 'MAPID Satellite',        icon: '🛰️' },
    { id: 'demo',      label: 'MapLibre Demo',          icon: '🌐' }
  ];
</script>

<div class="app">
  <aside>
    <!-- Brand -->
    <div class="brand">
      <div class="brand-icon">🌍</div>
      <div>
        <p class="kicker">MAPID × BINUS</p>
        <h1>Supplier WebGIS</h1>
      </div>
    </div>

    <div class="divider"></div>

    <!-- Basemap Selector -->
    <div class="section">
      <label class="section-label" for="basemap">
        <span class="label-icon">🗺️</span>
        Basemap
      </label>
      <div class="select-wrapper">
        <select id="basemap" bind:value={basemap}>
          {#each options as option}
            <option value={option.id}>{option.icon} {option.label}</option>
          {/each}
        </select>
        <span class="select-arrow">▾</span>
      </div>
      <button
        class="btn-mvt"
        class:loaded={layerActive}
        on:click={toggleMvt}
        disabled={layerActive}
      >
        <span class="btn-mvt-icon">{layerActive ? '✅' : '⚡'}</span>
        {layerActive ? 'MVT Loaded' : 'Use MVT'}
      </button>
    </div>

    <div class="divider"></div>

    <!-- Layer Controls -->
    <div class="section">
      <p class="section-label">
        <span class="label-icon">📊</span>
        Layers
      </p>
      <div class="layer-card" class:active={layerActive && layerVisible}>
        <div class="layer-info">
          <div class="layer-dot" class:on={layerActive && layerVisible}></div>
          <div class="layer-text">
            <span class="layer-name">Supplier Points</span>
            <span class="layer-desc">MVT • 50K records</span>
          </div>
        </div>
        <button
          class="toggle"
          class:on={layerActive && layerVisible}
          on:click={handleLayerToggle}
          aria-label="Toggle supplier points layer"
        >
          <span class="toggle-thumb"></span>
        </button>
      </div>
    </div>

    <div class="divider"></div>

    <!-- Status -->
    <div class="section">
      <p class="section-label">
        <span class="label-icon">ℹ️</span>
        Status
      </p>
      {#if !layerActive}
        <div class="status-card idle">
          <span class="status-dot idle"></span>
          <span>Layer belum aktif. Toggle layer untuk memuat data.</span>
        </div>
      {:else if layerVisible}
        <div class="status-card active">
          <span class="status-dot active"></span>
          <span>Data ditampilkan. Klik titik hijau untuk info detail.</span>
        </div>
      {:else}
        <div class="status-card hidden">
          <span class="status-dot hidden"></span>
          <span>Layer disembunyikan. Toggle kembali untuk menampilkan.</span>
        </div>
      {/if}
    </div>

    <!-- Footer -->
    <div class="footer">
      <span class="footer-text">Built with MapLibre GL JS</span>
    </div>
  </aside>

  <MapView bind:this={mapView} {basemap} bind:layerVisible />
</div>

<style>
  .app {
    display: flex;
    height: 100%;
    min-height: 100vh;
  }

  /* ── Sidebar ── */
  aside {
    width: 300px;
    flex-shrink: 0;
    padding: 24px 20px;
    background: linear-gradient(180deg, #13151e 0%, #0f1117 100%);
    border-right: 1px solid rgba(255, 255, 255, 0.06);
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    overflow-y: auto;
  }

  /* ── Brand ── */
  .brand {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 4px;
  }

  .brand-icon {
    width: 40px;
    height: 40px;
    border-radius: 10px;
    background: linear-gradient(135deg, #10b981, #059669);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.25);
  }

  .kicker {
    margin: 0;
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    color: #10b981;
  }

  h1 {
    margin: 2px 0 0;
    font-size: 18px;
    line-height: 1.2;
    font-weight: 700;
    color: #f1f5f9;
  }

  /* ── Dividers ── */
  .divider {
    height: 1px;
    background: rgba(255, 255, 255, 0.06);
    margin: 18px 0;
  }

  /* ── Section ── */
  .section {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .section-label {
    margin: 0;
    font-size: 11px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: #64748b;
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .label-icon {
    font-size: 13px;
  }

  /* ── Select ── */
  .select-wrapper {
    position: relative;
  }

  select {
    width: 100%;
    padding: 10px 32px 10px 12px;
    font: inherit;
    font-size: 13px;
    font-weight: 500;
    background: rgba(255, 255, 255, 0.04);
    color: #e2e8f0;
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 8px;
    appearance: none;
    cursor: pointer;
    transition: all 0.2s ease;
    outline: none;
  }

  select:hover {
    background: rgba(255, 255, 255, 0.06);
    border-color: rgba(255, 255, 255, 0.12);
  }

  select:focus {
    border-color: #10b981;
    box-shadow: 0 0 0 3px rgba(16, 185, 129, 0.15);
  }

  select option {
    background: #1a1d28;
    color: #e2e8f0;
  }

  .select-arrow {
    position: absolute;
    right: 12px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 11px;
    color: #64748b;
    pointer-events: none;
  }

  /* ── MVT Button ── */
  .btn-mvt {
    width: 100%;
    margin-top: 10px;
    padding: 10px 14px;
    font: inherit;
    font-size: 13px;
    font-weight: 600;
    color: #fff;
    background: linear-gradient(135deg, #10b981, #059669);
    border: none;
    border-radius: 8px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: all 0.25s ease;
    box-shadow: 0 2px 10px rgba(16, 185, 129, 0.25);
  }

  .btn-mvt:hover:not(:disabled) {
    transform: translateY(-1px);
    box-shadow: 0 4px 16px rgba(16, 185, 129, 0.35);
  }

  .btn-mvt:active:not(:disabled) {
    transform: translateY(0);
  }

  .btn-mvt.loaded {
    background: rgba(255, 255, 255, 0.06);
    color: #86efac;
    box-shadow: none;
    cursor: default;
    border: 1px solid rgba(16, 185, 129, 0.2);
  }

  .btn-mvt-icon {
    font-size: 14px;
  }

  /* ── Layer Card ── */
  .layer-card {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 12px 14px;
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid rgba(255, 255, 255, 0.06);
    border-radius: 10px;
    transition: all 0.25s ease;
  }

  .layer-card:hover {
    background: rgba(255, 255, 255, 0.05);
  }

  .layer-card.active {
    border-color: rgba(16, 185, 129, 0.25);
    background: rgba(16, 185, 129, 0.05);
  }

  .layer-info {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .layer-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #475569;
    transition: all 0.3s ease;
  }

  .layer-dot.on {
    background: #10b981;
    box-shadow: 0 0 8px rgba(16, 185, 129, 0.5);
  }

  .layer-text {
    display: flex;
    flex-direction: column;
    gap: 2px;
  }

  .layer-name {
    font-size: 13px;
    font-weight: 600;
    color: #e2e8f0;
  }

  .layer-desc {
    font-size: 10px;
    color: #64748b;
    font-weight: 500;
  }

  /* ── Toggle Switch ── */
  .toggle {
    position: relative;
    width: 40px;
    height: 22px;
    border-radius: 12px;
    background: #2a2d3a;
    border: 1px solid rgba(255, 255, 255, 0.08);
    cursor: pointer;
    padding: 0;
    transition: all 0.3s ease;
    flex-shrink: 0;
  }

  .toggle:hover {
    background: #333646;
  }

  .toggle.on {
    background: #10b981;
    border-color: #059669;
    box-shadow: 0 2px 8px rgba(16, 185, 129, 0.3);
  }

  .toggle-thumb {
    position: absolute;
    top: 2px;
    left: 2px;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #fff;
    transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
  }

  .toggle.on .toggle-thumb {
    left: 20px;
  }

  /* ── Status Card ── */
  .status-card {
    padding: 10px 12px;
    border-radius: 8px;
    font-size: 12px;
    line-height: 1.5;
    display: flex;
    align-items: flex-start;
    gap: 8px;
    color: #94a3b8;
  }

  .status-card.idle {
    background: rgba(100, 116, 139, 0.08);
    border: 1px solid rgba(100, 116, 139, 0.12);
  }

  .status-card.active {
    background: rgba(16, 185, 129, 0.06);
    border: 1px solid rgba(16, 185, 129, 0.15);
    color: #86efac;
  }

  .status-card.hidden {
    background: rgba(234, 179, 8, 0.06);
    border: 1px solid rgba(234, 179, 8, 0.15);
    color: #fde68a;
  }

  .status-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    flex-shrink: 0;
    margin-top: 5px;
  }

  .status-dot.idle {
    background: #64748b;
  }

  .status-dot.active {
    background: #10b981;
    box-shadow: 0 0 6px rgba(16, 185, 129, 0.5);
    animation: pulse 2s ease-in-out infinite;
  }

  .status-dot.hidden {
    background: #eab308;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.4; }
  }

  /* ── Footer ── */
  .footer {
    margin-top: auto;
    padding-top: 20px;
  }

  .footer-text {
    font-size: 10px;
    color: #334155;
    font-weight: 500;
  }
</style>
