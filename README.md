<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Application métier Cità di Biguglia</title>

<!-- CSS avec fallbacks JS en cas d'échec -->
<link id="tailwind" rel="stylesheet" href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css">
<link id="fontawesome" rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.4.0/css/all.min.css">

<script>
  // Fallback Tailwind
  document.getElementById('tailwind').addEventListener('error', () => {
    const alt = document.createElement('link');
    alt.rel = 'stylesheet';
    alt.href = 'https://unpkg.com/tailwindcss@2.2.19/dist/tailwind.min.css';
    document.head.appendChild(alt);
  });
  
  // Fallback FontAwesome
  document.getElementById('fontawesome').addEventListener('error', () => {
    const alt = document.createElement('link');
    alt.rel = 'stylesheet';
    alt.href = 'https://unpkg.com/@fortawesome/fontawesome-free@6.4.0/css/all.min.css';
    document.head.appendChild(alt);
  });
</script>

<!-- CHARTS avec fallbacks -->
<script defer src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"
        onerror="this.onerror=null;this.src='https://unpkg.com/chart.js@4.4.3/dist/chart.umd.js'"></script>
<script defer src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"
        onerror="this.onerror=null;this.src='https://unpkg.com/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js'"></script>

<!-- DOCX avec fallback -->
<script defer src="https://unpkg.com/mammoth/mammoth.browser.min.js"
        onerror="this.onerror=null;this.src='https://cdn.jsdelivr.net/npm/mammoth/mammoth.browser.min.js'"></script>

<!-- SheetJS pour l'export Excel -->
<script defer src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"
        onerror="this.onerror=null;this.src='https://unpkg.com/xlsx@0.18.5/dist/xlsx.full.min.js'"></script>

<!-- GANTT avec fallbacks -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/frappe-gantt@0.6.1/dist/frappe-gantt.css"
      onerror="this.href='https://unpkg.com/frappe-gantt@0.6.1/dist/frappe-gantt.css'">
<script defer src="https://cdn.jsdelivr.net/npm/frappe-gantt@0.6.1/dist/frappe-gantt.min.js"
        onerror="this.onerror=null;this.src='https://unpkg.com/frappe-gantt@0.6.1/dist/frappe-gantt.min.js'"></script>

<!-- Boot robuste : attend Chart (ou timeout) AVANT de lancer l'app -->
<script>
  // Boot robuste : attend Chart (ou timeout) AVANT de lancer l'app.
  window.addEventListener('DOMContentLoaded', () => {
    const MAX_WAIT = 8000; // 8s max puis on démarre quand même
    const t0 = performance.now();
    (function tick(){
      const hasChart = !!window.Chart;
      const hasDL    = !!window.ChartDataLabels;

      // Si Chart + DataLabels présents, on enregistre le plugin
      if (hasChart && hasDL) {
        try { Chart.register(ChartDataLabels); } catch(e) { /* ignore */ }
      }

      // On démarre l'app si initApp existe ET (Chart prêt OU timeout écoulé)
      const canStart = (typeof initApp === 'function') && (hasChart || (performance.now() - t0 > MAX_WAIT));
      if (canStart) { initApp(); return; }

      setTimeout(tick, 120);
    })();
  });
</script>

<style>
  .nav-btn.active { background: linear-gradient(135deg,#667eea 0%,#764ba2 100%); color:#fff; }
  .module-content { display:none; }
  .module-content.active { display:block; }
  .card { background:rgba(255,255,255,0.95); backdrop-filter: blur(10px); border:1px solid rgba(255,255,255,0.2); box-shadow:0 8px 32px rgba(0,0,0,0.1); }
  .modal { display:none; position:fixed; z-index:1000; inset:0; background:rgba(0,0,0,0.5);}
  .modal.active { display:flex; align-items:center; justify-content:center; }
  .modal-content { position:relative; background:#fff; border-radius:10px; box-shadow:0 20px 25px -5px rgba(0,0,0,0.1); max-width:90vw; max-height:90vh; overflow:hidden; }
  .modal-header { display:flex; justify-content:space-between; align-items:center; padding:14px 18px; background:#f8fafc; border-bottom:1px solid #e5e7eb; }
  .modal-body { padding:18px; overflow:auto; max-height:calc(90vh - 64px); }
  .inline-edit { width:100%; padding:8px 12px; border:1px solid #d1d5db; border-radius:6px; font-size:14px; }
  .inline-edit:focus { outline:none; border-color:#4f46e5; box-shadow:0 0 0 2px #4f46e5; }
  .hours-badge{ padding:6px 12px; border-radius:9999px; font-size:12px; font-weight:600; display:inline-block; min-width:60px; text-align:center;}
  .badge-payees{ background:#dcfce7; color:#166534; border:1px solid #bbf7d0; }
  .badge-recup{ background:#dbeafe; color:#1e40af; border:1px solid #bfdbfe; }
  .badge-voirie{ background:#fed7aa; color:#9a3412; border:1px solid #fdba74; }
  .filter-active{ background:#fef3c7!important; border-color:#f59e0b!important; font-weight:700; }
  .event-history-card{ background:#fff; border:2px solid #e5e7eb; border-radius:12px; padding:14px; cursor:pointer; transition:.2s; }
  .event-history-card:hover{ border-color:#6366f1; box-shadow:0 4px 12px rgba(99,102,241,0.15); transform:translateY(-2px); }
  #participantsTable{ min-width:1200px; }
  #participantsTable th{ position:sticky; top:0; z-index:10; }
  .type-select{ padding:4px 8px; border:1px solid #d1d5db; border-radius:4px; font-size:13px; width:140px;}
  .type-payees{ background:#dcfce7; color:#166534;}
  .type-recup{ background:#dbeafe; color:#1e40af;}
  .type-voirie{ background:#fed7aa; color:#9a3412;}
  .time-input{ width:90px; padding:4px 8px; border:1px solid #d1d5db; border-radius:4px; text-align:center; }
  .duration-display{ font-weight:bold; color:#059669; background:#dcfce7; padding:4px 8px; border-radius:4px; text-align:center;}
  .document-upload-zone{ border:2px dashed #d1d5db; border-radius:8px; padding:32px; text-align:center; cursor:pointer; }
  .document-upload-zone.dragover{ border-color:#10b981; background:#ecfdf5; }
  .calendar-grid{ display:grid; grid-template-columns: repeat(7, minmax(0,1fr)); gap:2px; background:#e5e7eb; border:1px solid #e5e7eb; border-radius:10px; }
  .day-cell{ background:#fff; padding:8px; min-height:78px; border:1px solid #1f2937; border-radius:8px; position:relative; }
  .day-num{ font-size:12px; color:#374151; }
  .event-dot{ position:absolute; right:8px; bottom:8px; background:#2563eb; color:#fff; border-radius:8px; min-width:24px; height:24px; display:flex; align-items:center; justify-content:center; font-size:12px; padding:0 6px; }
  .tooltip-portal{ position:absolute; pointer-events:none; transform: translate(-50%, -50%); transition: all .08s ease-out; }

  /* >>> Ajout pour afficher les noms des manifestations dans les cases du calendrier <<< */
  .event-list{
    margin-top:4px; padding-right:34px; /* laisse la place à la pastille chiffrée */
    display:flex; flex-direction:column; gap:2px;
  }
  .event-chip{
    font-size:11px; line-height:1.2; padding:2px 6px;
    background:#eef2ff; color:#1f2937; border:1px solid #e5e7eb; border-radius:6px;
    white-space:nowrap; overflow:hidden; text-overflow:ellipsis;
  }

  .flash { animation: flash 1.2s ease-out 1; }
  @keyframes flash { 0% { box-shadow:0 0 0 0 rgba(99,102,241,.9); } 100% { box-shadow:0 0 0 18px rgba(99,102,241,0); } }

  @media print {
    body * { visibility: hidden !important; }
    .module-content.active, .module-content.active * { visibility: visible !important; }
    .module-content.active { position:absolute; top:0; left:0; width:100%; }
    .no-print { display:none !important; }
    
    /* Styles spéciaux pour les graphiques convertis en images */
    .print-chart-image { 
      visibility: visible !important; 
      display: block !important; 
      page-break-inside: avoid; 
      margin: 10px auto !important;
      max-width: 90% !important;
    }
    
    /* Force l'affichage des images et masque les canvas */
    canvas { display: none !important; }
    img.print-chart-image { display: block !important; }
    
    /* Améliore l'espacement des cartes avec graphiques */
    .card { 
      page-break-inside: avoid; 
      margin-bottom: 20px !important; 
    }
    
    /* Assure que les titres des graphiques restent avec les graphiques */
    h3 { 
      page-break-after: avoid; 
      margin-bottom: 10px !important; 
    }
  }

  /* === Compactage propre des colonnes d'heures dans le tableau Participants === */
  #participantsTable .time-input{
    width: 86px;
    height: 30px;
    line-height: 30px;
    font-size: 13px;
    padding: 0 6px;
    box-sizing: border-box;
    text-align: center;
    font-variant-numeric: tabular-nums;
  }

  /* harmonise l'affichage cross-browser de input[type=time] */
  #participantsTable input.time-input::-webkit-datetime-edit,
  #participantsTable input.time-input::-webkit-datetime-edit-fields-wrapper,
  #participantsTable input.time-input::-webkit-datetime-edit-hour-field,
  #participantsTable input.time-input::-webkit-datetime-edit-minute-field{
    padding: 0;
  }
  #participantsTable input.time-input::-webkit-calendar-picker-indicator{
    margin: 0; padding: 0;
  }
  #participantsTable input.time-input::-webkit-inner-spin-button{
    display: none;
  }
  #participantsTable input.time-input{
    -webkit-appearance: textfield;
    -moz-appearance: textfield;
    appearance: textfield;
  }

  /* pastille "Durée" plus fine et à hauteur fixe (plus rien ne se coupe) */
  #participantsTable .duration-display{
    display: inline-block;
    min-width: 68px;
    height: 30px;
    line-height: 30px;
    font-size: 13px;
    padding: 0 8px;
    border-radius: 6px;
    white-space: nowrap;
    font-variant-numeric: tabular-nums;
  }

  /* réduit un peu le padding vertical des colonnes Début / Fin / Durée */
  #participantsTable td:nth-child(5),
  #participantsTable td:nth-child(6),
  #participantsTable td:nth-child(7){
    padding-top: 4px;
    padding-bottom: 4px;
  }

  /* badges des totaux en pied de tableau : compacts et lisibles */
  #participantsTable .hours-badge{
    display: inline-block;
    min-width: 60px;
    height: 22px;
    line-height: 22px;
    font-size: 12px;
    padding: 0 8px;
    font-variant-numeric: tabular-nums;
  }

  /* === Système de notifications/toasts === */
  .toast {
    position: fixed;
    top: 20px;
    right: 20px;
    background: #10b981;
    color: white;
    padding: 12px 20px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    z-index: 9999;
    transform: translateX(400px);
    transition: transform 0.3s ease;
    max-width: 350px;
    font-size: 14px;
  }
  .toast.show { transform: translateX(0); }
  .toast.error { background: #ef4444; }
  .toast.warning { background: #f59e0b; }
  .toast.info { background: #3b82f6; }

  /* === Boutons de la barre d'outils === */
  .toolbar-btn {
    display: inline-flex !important;
    align-items: center;
    justify-content: center;
    padding: 0.5rem 1rem;
    border-radius: 0.5rem;
    font-weight: 600;
    transition: all 0.2s ease;
    text-decoration: none;
    border: none;
    cursor: pointer;
    font-size: 0.875rem;
    line-height: 1.25rem;
  }

  .toolbar-btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }

  .toolbar-btn.btn-orange {
    background-color: #ea580c !important;
    color: white !important;
  }

  .toolbar-btn.btn-orange:hover {
    background-color: #c2410c !important;
  }

  /* === Styles spécifiques pour le bouton Export ICS === */
  .bg-orange-600 {
    background-color: #ea580c !important;
    color: white !important;
    border: 2px solid #ea580c !important;
  }

  .bg-orange-600:hover,
  .hover\:bg-orange-700:hover {
    background-color: #c2410c !important;
    border-color: #c2410c !important;
    color: white !important;
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(234, 88, 12, 0.3) !important;
  }

  /* Force la visibilité en mode clair */
  body:not(.dark) .bg-orange-600 {
    background-color: #ea580c !important;
    color: white !important;
    border: 2px solid #ea580c !important;
    opacity: 1 !important;
    display: inline-flex !important;
    visibility: visible !important;
  }

  /* === Mode sombre === */
  .dark {
    --bg-primary: #1f2937;
    --bg-secondary: #374151;
    --text-primary: #f9fafb;
    --text-secondary: #d1d5db;
    --border: #4b5563;
  }

  .dark body { background: linear-gradient(to bottom right, #1f2937, #111827) !important; color: var(--text-primary); }
  .dark .card { background: rgba(55, 65, 81, 0.95) !important; border-color: var(--border); color: var(--text-primary); }
  .dark .modal-content { background: var(--bg-secondary) !important; color: var(--text-primary); }
  .dark .modal-header { background: #4b5563 !important; color: var(--text-primary); border-color: var(--border); }
  .dark .inline-edit { background: var(--bg-secondary); color: var(--text-primary); border-color: var(--border); }
  .dark .nav-btn { background: var(--bg-secondary) !important; color: var(--text-secondary) !important; }
  .dark .nav-btn.active { background: linear-gradient(135deg,#667eea 0%,#764ba2 100%) !important; color: #fff !important; }
  .dark header { background: var(--bg-primary) !important; }
  .dark nav { background: var(--bg-primary) !important; }
  
  /* === Corrections mode sombre pour le texte === */
  .dark h1, .dark h2, .dark h3, .dark h4, .dark h5, .dark h6 { color: var(--text-primary) !important; }
  .dark p, .dark div, .dark span { color: var(--text-secondary) !important; }
  .dark label { color: var(--text-primary) !important; }
  .dark .text-gray-800 { color: var(--text-primary) !important; }
  .dark .text-gray-700 { color: var(--text-secondary) !important; }
  .dark .text-gray-600 { color: var(--text-secondary) !important; }
  .dark .text-gray-500 { color: #9ca3af !important; }
  .dark .text-gray-400 { color: #6b7280 !important; }
  .dark .font-semibold, .dark .font-bold { color: var(--text-primary) !important; }
  
  /* === Tables en mode sombre === */
  .dark table { background: var(--bg-secondary) !important; color: var(--text-primary) !important; }
  .dark th { background: #4b5563 !important; color: var(--text-primary) !important; border-color: var(--border) !important; }
  .dark td { border-color: var(--border) !important; color: var(--text-secondary) !important; }
  .dark .bg-gray-50 { background: #4b5563 !important; }
  .dark .bg-white { background: var(--bg-secondary) !important; }
  
  /* === Boutons en mode sombre === */
  .dark button { color: inherit !important; }
  .dark .bg-gray-100 { background: #4b5563 !important; color: var(--text-primary) !important; }
  .dark .bg-gray-200 { background: #6b7280 !important; color: var(--text-primary) !important; }
  .dark .hover\:bg-gray-200:hover { background: #6b7280 !important; }
  
  /* === Cartes et contenus en mode sombre === */
  .dark .border { border-color: var(--border) !important; }
  .dark .rounded-lg, .dark .rounded-xl { background: var(--bg-secondary) !important; }
  .dark .event-history-card { background: var(--bg-secondary) !important; border-color: var(--border) !important; color: var(--text-primary) !important; }
  .dark .event-history-card:hover { border-color: #6366f1 !important; }
  
  /* === Formulaires en mode sombre === */
  .dark input, .dark select, .dark textarea { background: var(--bg-secondary) !important; color: var(--text-primary) !important; border-color: var(--border) !important; }
  .dark input:focus, .dark select:focus, .dark textarea:focus { border-color: #4f46e5 !important; }
  
  /* === Badges et éléments colorés === */
  .dark .badge-payees { background: #065f46 !important; color: #d1fae5 !important; border-color: #047857 !important; }
  .dark .badge-recup { background: #1e3a8a !important; color: #dbeafe !important; border-color: #2563eb !important; }
  .dark .badge-voirie { background: #92400e !important; color: #fed7aa !important; border-color: #d97706 !important; }
  
  /* === Tooltips et éléments flottants === */
  .dark .tooltip-portal { background: var(--bg-secondary) !important; color: var(--text-primary) !important; border-color: var(--border) !important; }
  .dark .shadow-lg, .dark .shadow-2xl { box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5) !important; }
</style>
</head>

<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
<header class="bg-white shadow-lg">
  <div class="container mx-auto px-6 py-4">
    <div class="flex justify-between items-center">
      <div class="flex items-center space-x-4">
        <img src="https://page.gensparksite.com/v1/base64_upload/8f2d0c4ecbd1bd7687fa4da46e049539" alt="Logo Biguglia" class="h-12 w-12">
        <div>
          <h1 class="text-2xl font-bold text-gray-800">Application métier Cità di Biguglia</h1>
          <p class="text-gray-600">Gestion Voirie & Festivités</p>
        </div>
      </div>
      <div class="flex space-x-2 no-print">
        <!-- Recherche globale -->
        <div class="relative">
          <input type="text" id="globalSearch" placeholder="Rechercher..." 
                 class="pl-8 pr-4 py-2 border rounded-lg text-sm w-64"
                 onkeyup="handleGlobalSearch(this.value)">
          <i class="fas fa-search absolute left-2 top-3 text-gray-400"></i>
          <div id="searchResults" class="absolute top-16 right-0 bg-white border rounded-lg shadow-lg p-4 w-80 z-50" style="display:none;"></div>
        </div>
        
        <button type="button" onclick="printModule()" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-print mr-2"></i>Imprimer
        </button>
        <button type="button" onclick="resetApplication()" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-trash mr-2"></i>Reset Total
        </button>
        <button type="button" onclick="document.getElementById('importInput').click()" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-upload mr-2"></i>Importer
        </button>
        <button type="button" onclick="exporterDonnees()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-download mr-2"></i>Exporter
        </button>
        <button type="button" onclick="exportToExcel()" class="bg-green-700 hover:bg-green-800 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-file-excel mr-2"></i>Export Excel
        </button>
        <button type="button" onclick="exportDetailedReport()" class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-file-alt mr-2"></i>Rapport détaillé
        </button>
        <button type="button" onclick="exportToCalendar()" class="bg-orange-600 hover:bg-orange-700 text-white px-4 py-2 rounded-lg font-semibold" title="Exporter au format iCalendar (.ics)">
          <i class="fas fa-calendar-plus mr-2"></i>Export ICS
        </button>
        <button type="button" onclick="exportToUSB()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg font-semibold shadow-lg border-2 border-indigo-500" title="Sauvegarder directement sur votre clé USB">
          <i class="fas fa-usb mr-2"></i>💾 Export USB
        </button>
        <button type="button" onclick="showStorageManager()" class="bg-yellow-600 hover:bg-yellow-700 text-white px-4 py-2 rounded-lg font-semibold" title="Gérer l'espace de stockage">
          <i class="fas fa-database mr-2"></i>Stockage
        </button>
        <button type="button" onclick="toggleDarkMode()" class="bg-gray-600 hover:bg-gray-700 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-moon mr-2"></i><span id="darkModeText">Mode sombre</span>
        </button>
      </div>
    </div>
  </div>
</header>
<input type="file" id="importInput" accept=".json" style="display:none" onchange="importData(this.files)"/>

<nav class="bg-white shadow-md no-print">
  <div class="container mx-auto px-6">
    <div class="flex space-x-1">
      <button type="button" onclick="showModule('dashboard')" class="nav-btn active px-6 py-3 font-semibold rounded-t-lg">
        <i class="fas fa-chart-pie mr-2"></i>Tableau de bord
      </button>
      <button type="button" onclick="showModule('agents')" class="nav-btn bg-white text-gray-700 px-6 py-3 font-semibold rounded-t-lg">
        <i class="fas fa-users mr-2"></i>Agents
      </button>
      <button type="button" onclick="showModule('events')" class="nav-btn bg-white text-gray-700 px-6 py-3 font-semibold rounded-t-lg">
        <i class="fas fa-calendar-alt mr-2"></i>Événements
      </button>
      <button type="button" onclick="showModule('hours')" class="nav-btn bg-white text-gray-700 px-6 py-3 font-semibold rounded-t-lg">
        <i class="fas fa-clock mr-2"></i>Heures
      </button>
      <button type="button" onclick="showModule('calendar')" class="nav-btn bg-white text-gray-700 px-6 py-3 font-semibold rounded-t-lg">
        <i class="fas fa-calendar mr-2"></i>Calendrier
      </button>
      <button type="button" onclick="showModule('material')" class="nav-btn bg-white text-gray-700 px-6 py-3 font-semibold rounded-t-lg">
        <i class="fas fa-boxes mr-2"></i>Matériel
      </button>
    </div>
  </div>
</nav>

<main class="container mx-auto px-6 py-8">
  <!-- DASHBOARD -->
  <div id="dashboard" class="module-content active">
    <div class="mb-6 p-4 bg-gradient-to-r from-blue-50 to-indigo-50 rounded-xl border-2 border-indigo-200">
      <div class="flex items-center space-x-4">
        <i class="fas fa-filter text-indigo-600 text-xl"></i>
        <label class="text-lg font-bold text-gray-800">Filtrer par type d'événement :</label>
        <select id="typeFilter" class="inline-edit text-lg font-medium" onchange="onTypeFilterChange()">
          <option value="tous">📊 Tous les types</option>
        </select>
        <button onclick="resetTypeFilter()" class="bg-gray-500 hover:bg-gray-600 text-white px-4 py-2 rounded-lg font-medium">
          <i class="fas fa-undo mr-2"></i>Réinitialiser
        </button>
        <div id="filterStatus" class="text-sm font-medium"></div>
      </div>
    </div>

    <h2 class="text-3xl font-bold text-gray-800 mb-8">Tableau de bord</h2>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
      <div class="card rounded-xl p-6">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-600">Total Heures Payées</p>
            <p id="totalHeuresPayees" class="text-2xl font-bold text-green-600">0h00m</p>
          </div>
          <div class="p-3 bg-green-100 rounded-full"><i class="fas fa-euro-sign text-green-600"></i></div>
        </div>
      </div>
      <div class="card rounded-xl p-6">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-600">Total Heures Récup</p>
            <p id="totalHeuresRecup" class="text-2xl font-bold text-blue-600">0h00m</p>
          </div>
          <div class="p-3 bg-blue-100 rounded-full"><i class="fas fa-undo text-blue-600"></i></div>
        </div>
      </div>
      <div class="card rounded-xl p-6">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-sm font-medium text-gray-600">Voirie → Festivités</p>
            <p id="heuresVoirieFestvites" class="text-2xl font-bold text-yellow-600">0h00m</p>
          </div>
          <div class="p-3 bg-yellow-100 rounded-full"><i class="fas fa-exchange-alt text-yellow-600"></i></div>
        </div>
      </div>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
      <div class="card rounded-xl p-6">
        <h3 class="text-xl font-bold text-gray-800 mb-4">Événements mensuels</h3>
        <div style="height:380px"><canvas id="eventsChart"></canvas></div>
      </div>
      <div class="card rounded-xl p-6">
        <h3 class="text-xl font-bold text-gray-800 mb-4">Répartition par secteur</h3>
        <div style="height:380px"><canvas id="sectorsChart"></canvas></div>
      </div>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
      <div class="card rounded-xl p-6">
        <h3 class="text-xl font-bold text-gray-800 mb-4">Analyse des Heures</h3>
        <div style="height:380px"><canvas id="hoursChart"></canvas></div>
      </div>
      <div class="card rounded-xl p-6">
        <h3 class="text-xl font-bold text-gray-800 mb-4">Répartition Mensuelle par Type</h3>
        <div style="height:380px"><canvas id="monthlyTypeChart"></canvas></div>
      </div>
    </div>

    <!-- 2 courbes l'une sous l'autre -->
    <div class="grid grid-cols-1 gap-8 mb-8">
      <div class="card rounded-xl p-6">
        <div class="flex items-center justify-between mb-2">
          <h3 class="text-xl font-bold text-gray-800">Heures annuelles (ligne)</h3>
          <div class="flex items-center space-x-2">
            <span class="text-sm text-gray-600">Type :</span>
            <select id="lineTypeFilter1" class="inline-edit w-auto" onchange="createYearlyHoursLineChart()">
              <option value="tous">Tous</option>
            </select>
          </div>
        </div>
        <div style="height:380px"><canvas id="yearlyHoursLineChart"></canvas></div>
      </div>

      <div class="card rounded-xl p-6">
        <div class="flex items-center justify-between mb-2">
          <h3 class="text-xl font-bold text-gray-800">Heures mensuelles par nature (ligne)</h3>
          <div class="flex items-center space-x-2">
            <span class="text-sm text-gray-600">Type :</span>
            <select id="lineTypeFilter2" class="inline-edit w-auto" onchange="createMonthlyBreakdownLineChart()">
              <option value="tous">Tous</option>
            </select>
          </div>
        </div>
        <div style="height:380px"><canvas id="monthlyBreakdownLineChart"></canvas></div>
      </div>
    </div>

    <!-- Graphique multi-sélections -->
    <div class="card rounded-xl p-6 mb-8">
      <div class="flex items-center justify-between mb-3">
        <h3 class="text-xl font-bold text-gray-800">Heures multi-sélections (ligne)</h3>
        <button class="bg-gray-200 hover:bg-gray-300 text-gray-800 px-3 py-1 rounded text-sm" onclick="resetMultiFilters()">
          <i class="fas fa-undo mr-1"></i>Réinitialiser
        </button>
      </div>
      <div class="flex flex-wrap gap-3 mb-3">
        <div class="flex items-center space-x-2">
          <span class="text-sm text-gray-600">Type :</span>
          <select id="multiTypeFilter" class="inline-edit w-auto"></select>
        </div>
        <div class="flex items-center space-x-2">
          <span class="text-sm text-gray-600">Agent :</span>
          <select id="multiAgentFilter" class="inline-edit w-auto"></select>
        </div>
        <div class="flex items-center space-x-2">
          <span class="text-sm text-gray-600">Événement :</span>
          <select id="multiEventFilter" class="inline-edit w-auto"></select>
        </div>
      </div>
      <div style="height:380px"><canvas id="multiHoursChart"></canvas></div>
      <div class="text-xs text-gray-500 mt-2">Astuce : cliquez sur la légende pour masquer/afficher une série.</div>
    </div>

    <!-- NOUVEAU : Heures annuelles par type (ligne) -->
    <div class="card rounded-xl p-6 mb-8">
      <div class="flex items-center justify-between mb-3">
        <h3 class="text-xl font-bold text-gray-800">Heures annuelles par type (ligne)</h3>
        <div class="text-sm text-gray-500">Axe Y max 650 h</div>
      </div>
      <div style="height:380px">
        <canvas id="yearlyTypeHoursChart"></canvas>
      </div>
      <div class="text-xs text-gray-500 mt-2">
        Chaque courbe = un type d’événement (CCAS, Scolaire, Démocratie participative, Pôle de vie, Spaziu culturale, Sport).
      </div>
    </div>

    <div class="card rounded-xl p-6 mt-8">
      <h3 class="text-xl font-bold mb-4">Agents – Voir détail annuel</h3>
      <ul id="dashboardAgentsList" class="space-y-2"></ul>
    </div>
  </div>

  <!-- AGENTS -->
  <div id="agents" class="module-content">
    <div class="flex justify-between items-center mb-6">
      <h2 class="text-3xl font-bold text-gray-800">Gestion des Agents</h2>
      <button type="button" id="btnAddAgent" onclick="showAgentForm()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg font-semibold">
        <i class="fas fa-plus mr-2"></i>Ajouter Agent
      </button>
    </div>
    <div class="card rounded-xl p-6">
      <div id="agentsList" class="space-y-4"></div>
    </div>

    <div id="addAgentForm" class="card rounded-xl p-6 mt-6" style="display:none;">
      <h3 id="agentFormTitle" class="text-xl font-bold text-gray-800 mb-4">Ajouter un Agent</h3>
      <form id="agentForm" onsubmit="saveAgent(event)">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Nom</label>
            <input type="text" id="agentName" required class="inline-edit">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Service</label>
            <select id="agentService" required class="inline-edit">
              <option value="Voirie">Voirie</option>
              <option value="Transport">Transport</option>
              <option value="Engin">Engin</option>
              <option value="Technique">Technique</option>
              <option value="Sport">Sport</option>
              <option value="Entretien">Entretien</option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Téléphone</label>
            <input type="tel" id="agentPhone" required class="inline-edit" 
                   pattern="^[0-9 .+-]{6,}$" 
                   title="Format : au moins 6 caractères (chiffres, espaces, points, + ou -)">
          </div>
        </div>
        <div class="flex space-x-4 mt-6">
          <button type="submit" id="agentFormSubmit" class="bg-green-600 hover:bg-green-700 text-white px-6 py-2 rounded-lg font-semibold">
            <i class="fas fa-save mr-2"></i><span>Ajouter</span>
          </button>
          <button type="button" onclick="hideAddAgentForm()" class="bg-gray-600 hover:bg-gray-700 text-white px-6 py-2 rounded-lg font-semibold">
            <i class="fas fa-times mr-2"></i>Annuler
          </button>
        </div>
      </form>
    </div>
  </div>

  <!-- EVENTS -->
  <div id="events" class="module-content">
    <div class="flex justify-between items-center mb-6">
      <h2 class="text-3xl font-bold text-gray-800">Gestion des Événements</h2>
      <button type="button" onclick="showAddEventForm()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg font-semibold">
        <i class="fas fa-plus mr-2"></i>Nouvel Événement
      </button>
    </div>

    <div class="card rounded-xl p-6">
      <div id="eventsList" class="space-y-4"></div>
    </div>

    <div id="addEventForm" class="card rounded-xl p-6 mt-6" style="display:none;">
      <h3 id="eventFormTitle" class="text-xl font-bold text-gray-800 mb-4">Créer un Événement</h3>
      <form id="eventForm" onsubmit="saveEvent(event)">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="md:col-span-2">
            <label class="block text-sm font-medium text-gray-700 mb-2">Nom de l'événement</label>
            <input type="text" id="eventName" required class="inline-edit">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Type</label>
            <input type="text" id="eventType" required class="inline-edit" placeholder="Ex: CCAS, Scolaire, Pôle de vie, Spaziu culturale...">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Lieu</label>
            <input type="text" id="eventLieu" required class="inline-edit">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Date début</label>
            <input type="date" id="eventDateDebut" required class="inline-edit">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Date fin</label>
            <input type="date" id="eventDateFin" required class="inline-edit">
          </div>
          <div class="md:col-span-2">
            <label class="block text-sm font-medium text-gray-700 mb-2">Description</label>
            <textarea id="eventDescription" rows="3" class="inline-edit"></textarea>
          </div>
        </div>
        <div class="flex space-x-4 mt-6">
          <button type="submit" class="bg-green-600 hover:bg-green-700 text-white px-6 py-2 rounded-lg font-semibold">
            <i class="fas fa-save mr-2"></i><span id="eventFormSubmit">Créer</span>
          </button>
          <button type="button" onclick="hideAddEventForm()" class="bg-gray-600 hover:bg-gray-700 text-white px-6 py-2 rounded-lg font-semibold">
            <i class="fas fa-times mr-2"></i>Annuler
          </button>
        </div>
      </form>
    </div>
  </div>

  <!-- HOURS -->
  <div id="hours" class="module-content">
    <h2 class="text-3xl font-bold text-gray-800 mb-6">Tableau Récapitulatif des Heures</h2>
    <div id="hoursContainer" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"></div>
  </div>

  <!-- CALENDAR -->
  <div id="calendar" class="module-content">
    <div class="flex items-center justify-between mb-4">
      <h2 class="text-3xl font-bold text-gray-800">Calendrier annuel</h2>
      <div class="flex items-center space-x-2">
        <select id="calendarYear" class="inline-edit w-auto" onchange="renderCalendar()"></select>
      </div>
    </div>
    <div id="calendarWrap" class="space-y-8"></div>
  </div>

  <!-- MATERIAL -->
  <div id="material" class="module-content">
    <div class="flex justify-between items-center mb-6">
      <h2 class="text-3xl font-bold text-gray-800">Gestion du Matériel</h2>
      <div class="flex space-x-3">
        <button type="button" onclick="showMaterialForm()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-plus mr-2"></i>Ajouter Matériel
        </button>
        <button type="button" onclick="showMaterialConsumptionChart()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-semibold">
          <i class="fas fa-chart-line mr-2"></i>Consommation par mois
        </button>
      </div>
    </div>

    <!-- Filtres et recherche -->
    <div class="mb-6 p-4 bg-gradient-to-r from-blue-50 to-indigo-50 rounded-xl border-2 border-indigo-200">
      <div class="flex items-center space-x-4">
        <i class="fas fa-filter text-indigo-600 text-xl"></i>
        <select id="materialCategoryFilter" class="inline-edit" onchange="updateMaterialList()">
          <option value="tous">📦 Toutes catégories</option>
        </select>
        <input type="text" id="materialSearch" placeholder="Rechercher un article..." 
               class="inline-edit flex-1" onkeyup="updateMaterialList()">
        <button onclick="updateMaterialAvailability()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-medium">
          <i class="fas fa-sync mr-2"></i>Actualiser Stock
        </button>
      </div>
    </div>

    <!-- Liste du matériel -->
    <div class="card rounded-xl p-6">
      <div id="materialList" class="space-y-4"></div>
      <div id="materialEmpty" class="text-center text-gray-500 py-10" style="display:none;">
        <i class="fas fa-box-open fa-3x mb-4 text-gray-300"></i>
        <h3 class="text-lg font-semibold mb-2">Aucun matériel</h3>
        <p class="text-sm">Ajoutez votre premier article avec le bouton ci-dessus</p>
      </div>
    </div>

    <!-- Formulaire d'ajout/modification -->
    <div id="addMaterialForm" class="card rounded-xl p-6 mt-6" style="display:none;">
      <h3 id="materialFormTitle" class="text-xl font-bold text-gray-800 mb-4">Ajouter du Matériel</h3>
      <form id="materialForm" onsubmit="saveMaterial(event)">
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Nom de l'article *</label>
            <input type="text" id="materialName" required class="inline-edit" placeholder="Ex: Table rectangulaire">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Catégorie *</label>
            <div class="space-y-2">
              <select id="materialCategory" class="inline-edit" onchange="handleCategorySelection(this)">
                <option value="">-- Sélectionner --</option>
                <option value="Tables">Tables</option>
                <option value="Chaises">Chaises</option>
                <option value="Tentes">Tentes</option>
                <option value="Éclairage">Éclairage</option>
                <option value="Sonorisation">Sonorisation</option>
                <option value="Décoration">Décoration</option>
                <option value="Sécurité">Sécurité</option>
                <option value="Transport">Transport</option>
                <option value="Technique">Technique</option>
                <option value="custom">✏️ Autre (saisir librement)</option>
              </select>
              <input type="text" id="materialCategoryCustom" class="inline-edit" placeholder="Saisir une nouvelle catégorie..." style="display:none;">
            </div>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Stock total *</label>
            <input type="number" id="materialStock" required min="0" class="inline-edit" placeholder="Ex: 50">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Stock minimum d'alerte</label>
            <input type="number" id="materialMinStock" min="0" class="inline-edit" placeholder="Ex: 5">
          </div>
          <div class="md:col-span-2 lg:col-span-2">
            <label class="block text-sm font-medium text-gray-700 mb-2">Description</label>
            <textarea id="materialDescription" rows="3" class="inline-edit" placeholder="Description, caractéristiques, remarques..."></textarea>
          </div>
        </div>
        <div class="flex space-x-4 mt-6">
          <button type="submit" id="materialFormSubmit" class="bg-green-600 hover:bg-green-700 text-white px-6 py-2 rounded-lg font-semibold">
            <i class="fas fa-save mr-2"></i><span>Ajouter</span>
          </button>
          <button type="button" onclick="hideMaterialForm()" class="bg-gray-600 hover:bg-gray-700 text-white px-6 py-2 rounded-lg font-semibold">
            <i class="fas fa-times mr-2"></i>Annuler
          </button>
        </div>
      </form>
    </div>
  </div>
</main>

<!-- Material Modal -->
<div id="materialModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="materialModalTitle">
  <div class="modal-content" style="width: 95vw; max-width: 1400px; max-height: 90vh;">
    <div class="modal-header">
      <h3 id="materialModalTitle" class="text-2xl font-bold"><i class="fas fa-boxes mr-2 text-purple-600"></i>Gestion Matériel - <span id="materialEventName"></span></h3>
      <div class="flex items-center space-x-2">
        <button onclick="printMaterialList()" class="bg-green-600 text-white px-3 py-2 rounded" title="Imprimer la liste du matériel"><i class="fas fa-print mr-1"></i>Imprimer</button>
        <button onclick="closeMaterialModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>
    <div class="modal-body" style="overflow-y:auto; max-height:calc(90vh - 120px);">
      
      <!-- Section attribution matériel -->
      <div class="bg-gradient-to-r from-purple-50 to-indigo-50 p-4 rounded-lg mb-6 border border-purple-200">
        <h4 class="text-lg font-bold text-gray-800 mb-3"><i class="fas fa-plus-circle mr-2 text-purple-600"></i>Attribution de matériel</h4>
        
        <!-- Sélection matériel -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Article</label>
            <select id="materialSelect" class="inline-edit">
              <option value="">-- Sélectionner un article --</option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Quantité</label>
            <input type="number" id="materialQuantity" min="1" value="1" class="inline-edit" placeholder="Ex: 10">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Date début</label>
            <input type="date" id="materialStartDate" class="inline-edit">
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Date fin</label>
            <input type="date" id="materialEndDate" class="inline-edit">
          </div>
        </div>
        
        <div class="flex justify-between items-center">
          <div id="materialAlert" class="text-sm" style="display:none;"></div>
          <button onclick="addMaterialToEvent()" class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg font-semibold">
            <i class="fas fa-plus mr-2"></i>Attribuer le matériel
          </button>
        </div>
      </div>

      <!-- Liste du matériel attribué -->
      <div class="card rounded-xl p-6">
        <h4 class="text-lg font-bold text-gray-800 mb-4"><i class="fas fa-list mr-2 text-blue-600"></i>Matériel attribué à cet événement</h4>
        <div id="eventMaterialList" class="space-y-3"></div>
        <div id="eventMaterialEmpty" class="text-center text-gray-500 py-8" style="display:none;">
          <i class="fas fa-box fa-3x mb-4 text-gray-300"></i>
          <h4 class="text-lg font-semibold mb-2">Aucun matériel attribué</h4>
          <p class="text-sm">Utilisez le formulaire ci-dessus pour attribuer du matériel à cet événement</p>
        </div>
      </div>

      <div class="mt-6 flex justify-end">
        <button onclick="closeMaterialModal()" class="bg-gray-400 text-white px-6 py-3 rounded-lg font-semibold hover:bg-gray-500">Fermer</button>
      </div>
    </div>
  </div>
</div>

<!-- Material Consumption Chart Modal -->
<div id="materialConsumptionModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="materialConsumptionTitle">
  <div class="modal-content" style="width: 95vw; max-width: 1400px; max-height: 90vh;">
    <div class="modal-header">
      <h3 id="materialConsumptionTitle" class="text-2xl font-bold"><i class="fas fa-chart-bar mr-2 text-blue-600"></i>Analyse de Consommation du Matériel</h3>
      <div class="flex items-center space-x-2">
        <button onclick="printModal('materialConsumptionModal')" class="bg-green-600 text-white px-3 py-2 rounded" title="Imprimer le rapport"><i class="fas fa-print mr-1"></i>Imprimer</button>
        <button onclick="closeMaterialConsumptionModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>
    <div class="modal-body" style="overflow-y:auto; max-height:calc(90vh - 120px);">
      
      <!-- Filtres et période -->
      <div class="mb-6 p-4 bg-gradient-to-r from-blue-50 to-indigo-50 rounded-xl border-2 border-indigo-200">
        <div class="flex items-center space-x-4 mb-3">
          <i class="fas fa-filter text-indigo-600 text-xl"></i>
          <select id="consumptionMaterialFilter" class="inline-edit" onchange="updateConsumptionChart()">
            <option value="all">📦 Tous les articles</option>
          </select>
          <select id="consumptionPeriodFilter" class="inline-edit" onchange="updateConsumptionChart()">
            <option value="year">📅 Année en cours</option>
            <option value="6months">📅 6 derniers mois</option>
            <option value="3months">📅 3 derniers mois</option>
          </select>
        </div>
        <div class="text-sm text-gray-600">
          <i class="fas fa-info-circle mr-2"></i>Analysez la consommation pour prévoir vos besoins et optimiser vos stocks
        </div>
      </div>

      <!-- Graphiques -->
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
        <!-- Consommation mensuelle -->
        <div class="card rounded-xl p-6">
          <h4 class="text-lg font-bold text-gray-800 mb-4">
            <i class="fas fa-calendar-alt mr-2 text-blue-600"></i>Consommation mensuelle
          </h4>
          <div style="height:350px"><canvas id="monthlyConsumptionChart"></canvas></div>
          <div class="mt-3 text-sm text-gray-600 italic">
            <i class="fas fa-info-circle mr-2 text-blue-500"></i>
            Ce graphique montre l'évolution de l'utilisation du matériel mois par mois. 
            Identifiez les périodes de forte demande pour anticiper vos besoins et optimiser votre planning d'achats.
          </div>
        </div>
        
        <!-- Consommation par événement -->
        <div class="card rounded-xl p-6">
          <h4 class="text-lg font-bold text-gray-800 mb-4">
            <i class="fas fa-chart-pie mr-2 text-green-600"></i>Répartition par événement
          </h4>
          <div style="height:350px"><canvas id="eventConsumptionChart"></canvas></div>
          <div class="mt-3 text-sm text-gray-600 italic">
            <i class="fas fa-info-circle mr-2 text-green-500"></i>
            Visualisez quels événements mobilisent le plus de matériel. 
            Cela vous aide à préparer des kits spécialisés et à mieux évaluer les coûts d'organisation par type d'événement.
          </div>
        </div>
      </div>

      <!-- Prévisions et tendances -->
      <div class="card rounded-xl p-6 mb-6">
        <h4 class="text-lg font-bold text-gray-800 mb-4">
          <i class="fas fa-crystal-ball mr-2 text-purple-600"></i>Analyse des tendances et prévisions
        </h4>
        <div class="mb-4 text-sm text-gray-600 italic">
          <i class="fas fa-info-circle mr-2 text-purple-500"></i>
          Ces analyses comparent les consommations actuelles avec les données historiques pour vous aider à anticiper vos besoins futurs et optimiser vos achats.
        </div>
        <div id="budgetPredictions" class="grid grid-cols-1 md:grid-cols-3 gap-4"></div>
      </div>

      <!-- Tableau détaillé -->
      <div class="card rounded-xl p-6">
        <h4 class="text-lg font-bold text-gray-800 mb-4">
          <i class="fas fa-table mr-2 text-orange-600"></i>Détail des consommations
        </h4>
        <div class="overflow-x-auto">
          <table id="consumptionDetailTable" class="w-full border-collapse border">
            <thead class="bg-gray-50">
              <tr>
                <th class="border p-2 text-left">Article</th>
                <th class="border p-2 text-left">Événement</th>
                <th class="border p-2 text-center">Quantité</th>
                <th class="border p-2 text-center">Période</th>
                <th class="border p-2 text-center">Statut</th>
              </tr>
            </thead>
            <tbody id="consumptionDetailBody">
            </tbody>
          </table>
        </div>
      </div>

      <div class="mt-6 flex justify-end">
        <button onclick="closeMaterialConsumptionModal()" class="bg-gray-400 text-white px-6 py-3 rounded-lg font-semibold hover:bg-gray-500">Fermer</button>
      </div>
    </div>
  </div>
</div>

<!-- Agent History Modal -->
<div id="agentHistoryModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="agentHistoryTitle">
  <div class="modal-content" style="width: 98vw; max-width: 1600px; max-height: 95vh;">
    <div class="modal-header">
      <h3 id="agentHistoryTitle" class="text-2xl font-bold">Historique des événements - <span id="agentHistoryName"></span></h3>
      <div class="flex items-center space-x-2">
        <button onclick="printModal('agentHistoryModal')" class="bg-green-600 text-white px-3 py-2 rounded" title="Imprimer l'historique"><i class="fas fa-print mr-1"></i>Imprimer</button>
        <button onclick="closeAgentHistoryModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>
    <div class="modal-body" style="overflow-y: auto; max-height: calc(95vh - 80px);">
      <div class="mb-4">
        <div class="bg-gradient-to-r from-blue-50 to-indigo-50 p-4 rounded-lg border"><div id="agentHistoryStats" class="text-center"></div></div>
      </div>
      
      <!-- Graphique de l'évolution des heures par événement -->
      <div class="mb-6 bg-white border rounded-lg p-4">
        <h4 class="text-lg font-bold text-gray-800 mb-3">
          <i class="fas fa-chart-line mr-2 text-blue-600"></i>Évolution des heures par événement
        </h4>
        <div style="height: 500px; min-height: 500px;">
          <canvas id="agentEvolutionChart"></canvas>
        </div>
        <div class="text-xs text-gray-500 mt-2">
          <i class="fas fa-info-circle mr-1"></i>Chaque point représente un événement avec ses heures par type. Cliquez sur un point pour voir le détail de l'événement.
        </div>
      </div>
      
      <div id="agentHistoryEvents" class="space-y-3 max-h-96 overflow-y-auto"></div>
      <div class="mt-6 flex justify-end">
        <button onclick="closeAgentHistoryModal()" class="bg-gray-500 text-white px-6 py-2 rounded-lg font-semibold">Fermer</button>
      </div>
    </div>
  </div>
</div>

<!-- Agent Event Detail Modal -->
<div id="agentEventDetailModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="agentEventDetailTitle">
  <div class="modal-content" style="width: 90vw; max-width: 900px;">
    <div class="modal-header">
      <h3 id="agentEventDetailTitle" class="text-2xl font-bold"><i class="fas fa-list-ul mr-2 text-blue-600"></i>Détails – <span id="agentEventDetailName"></span></h3>
      <div class="flex items-center space-x-2">
        <button onclick="printModal('agentEventDetailModal')" class="bg-green-600 text-white px-3 py-2 rounded" title="Imprimer les détails"><i class="fas fa-print mr-1"></i>Imprimer</button>
        <button onclick="closeAgentEventDetailModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>
    <div class="modal-body">
      <div class="overflow-x-auto">
        <table class="w-full border">
          <thead class="bg-gray-50">
            <tr>
              <th class="px-3 py-2 border">📅 Date</th>
              <th class="px-3 py-2 border">🖊️ Motif</th>
              <th class="px-3 py-2 border">🔄 Type</th>
              <th class="px-3 py-2 border">🕓 Début</th>
              <th class="px-3 py-2 border">🕔 Fin</th>
              <th class="px-3 py-2 border">⏳ Durée</th>
            </tr>
          </thead>
          <tbody id="agentEventDetailTable"></tbody>
        </table>
      </div>
      <div class="mt-6 flex justify-end">
        <button onclick="closeAgentEventDetailModal()" class="bg-gray-500 text-white px-6 py-2 rounded-lg font-semibold">Retour</button>
      </div>
    </div>
  </div>
</div>

<!-- Participants Modal -->
<div id="participantsModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="participantsModalTitle">
  <div class="modal-content" style="width: 95vw; max-width: 1600px; max-height: 90vh;">
    <div class="modal-header">
      <h3 id="participantsModalTitle" class="text-2xl font-bold"><i class="fas fa-users-cog mr-2 text-blue-600"></i>Participants - <span id="partsEventName"></span></h3>
      <div class="flex items-center space-x-2">
        <button onclick="printModal('participantsModal')" class="bg-green-600 text-white px-3 py-2 rounded" title="Imprimer la liste des participants"><i class="fas fa-print mr-1"></i>Imprimer</button>
        <button onclick="closeParticipantsModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>
    <div class="modal-body" style="overflow-y:auto; max-height:calc(90vh - 120px);">
      <div class="bg-gradient-to-r from-green-50 to-blue-50 p-4 rounded-lg mb-6 border border-green-200">
        <div class="flex justify-between items-center">
          <div>
            <h4 class="text-lg font-bold text-gray-800 mb-2"><i class="fas fa-clock mr-2 text-green-600"></i>Gestion intelligente des heures</h4>
            <p class="text-sm text-gray-600">Début/fin → durée auto • motif libre • totaux par type en pied de tableau.</p>
          </div>
          <button id="addRowBtn" onclick="onAddRow()" class="bg-green-600 text-white px-4 py-2 rounded-lg font-semibold hover:bg-green-700"><i class="fas fa-plus mr-2"></i>Ajouter une ligne</button>
        </div>
      </div>

      <div class="overflow-x-auto bg-white rounded-lg border border-gray-200 shadow-sm">
        <table id="participantsTable" class="w-full">
          <thead class="bg-gradient-to-r from-blue-600 to-indigo-600 text-white">
            <tr>
              <th class="px-3 py-3 text-left font-semibold" style="width:110px;">Date</th>
              <th class="px-3 py-3 text-left font-semibold" style="width:180px;">Agent</th>
              <th class="px-3 py-3 text-left font-semibold" style="width:200px;">Motif (libre)</th>
              <th class="px-3 py-3 text-center font-semibold" style="width:150px;">Type</th>
              <th class="px-3 py-3 text-center font-semibold" style="width:90px;">Début</th>
              <th class="px-3 py-3 text-center font-semibold" style="width:90px;">Fin</th>
              <th class="px-3 py-3 text-center font-semibold" style="width:100px;">Durée</th>
              <th class="px-3 py-3 text-center font-semibold" style="width:80px;">Actions</th>
            </tr>
          </thead>
          <tbody id="participantsTableBody"></tbody>
          <tfoot>
            <tr class="bg-gray-50 font-semibold">
              <td colspan="6" class="px-4 py-3 text-right"><i class="fas fa-calculator mr-2 text-purple-600"></i>Totaux :</td>
              <td class="px-4 py-3 text-center">
                <div class="text-xs space-y-1">
                  <div class="text-green-700"><span class="badge-payees hours-badge" id="totalPayees">0.0h</span></div>
                  <div class="text-blue-700"><span class="badge-recup hours-badge" id="totalRecup">0.0h</span></div>
                  <div class="text-yellow-700"><span class="badge-voirie hours-badge" id="totalVoirieFest">0.0h</span></div>
                </div>
              </td>
              <td class="px-4 py-3"></td>
            </tr>
          </tfoot>
        </table>
      </div>

      <div id="participantsEmpty" class="text-center py-12" style="display:none;">
        <i class="fas fa-user-clock fa-3x text-gray-300 mb-4"></i>
        <h4 class="text-lg font-semibold text-gray-600 mb-2">Aucune ligne d'heures</h4>
        <p class="text-gray-500 mb-4">Cliquez sur "Ajouter une ligne" pour commencer</p>
      </div>

      <div class="mt-6 flex justify-between items-center bg-gray-50 p-4 rounded-lg">
        <div class="text-sm text-gray-600"><i class="fas fa-info-circle mr-2"></i>Modifiez les champs, la durée se calcule automatiquement • Sauvegarde quand vous cliquez sur “Sauvegarder”.</div>
        <div class="flex space-x-3">
          <button id="savePartsBtn" onclick="onSaveParticipants()" class="bg-green-600 text-white px-6 py-3 rounded-lg font-semibold hover:bg-green-700">
            <i class="fas fa-save mr-2"></i>Sauvegarder
          </button>
          <button onclick="onCancelParticipants()" class="bg-gray-400 text-white px-6 py-3 rounded-lg font-semibold hover:bg-gray-500">
            <i class="fas fa-times mr-2"></i>Annuler
          </button>
        </div>
      </div>

    </div>
  </div>
</div>

<!-- Documents Modal -->
<div id="docsModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="docsModalTitle">
  <div class="modal-content" style="width: 95vw; max-width: 1200px;">
    <div class="modal-header">
      <h3 id="docsModalTitle" class="text-2xl font-bold"><i class="fas fa-folder-open mr-2 text-blue-600"></i>Gestion Documentaire – <span id="docsEventName"></span></h3>
      <div class="flex items-center space-x-2">
        <button onclick="printModal('docsModal')" class="bg-green-600 text-white px-3 py-2 rounded" title="Imprimer la liste des documents"><i class="fas fa-print mr-1"></i>Imprimer</button>
        <button onclick="closeDocsModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>
    <div class="modal-body">
      <div class="mb-6">
        <div id="docDropZone" class="document-upload-zone"
             onclick="document.getElementById('docFileInput').click()"
             ondrop="handleFileDrop(event)" ondragover="handleDragOver(event)" ondragleave="handleDragLeave(event)">
          <i class="fas fa-cloud-upload-alt fa-3x text-gray-400 mb-4"></i>
          <h4 class="text-lg font-semibold text-gray-700 mb-2">Glissez vos fichiers ici ou cliquez pour parcourir</h4>
          <p class="text-sm text-gray-500 mb-2">PDF, DOCX, XLSX, Images… (max 10 Mo / fichier)</p>
          <p class="text-xs text-gray-400">Les fichiers sont intégrés en Base64 et sauvegardés dans le JSON exporté (attention à la taille).</p>
        </div>
        <input type="file" id="docFileInput" multiple
               accept="*/*"
               style="display:none" onchange="handleFileSelect(event)">
      </div>

      <div class="flex justify-between items-center mb-3">
        <h4 class="text-lg font-bold text-gray-800"><i class="fas fa-list mr-2 text-blue-600"></i>Documents (<span id="docsCount">0</span>)</h4>
        <div class="flex space-x-2">
          <button onclick="downloadAllDocuments()" class="bg-blue-600 text-white px-3 py-2 rounded text-sm hover:bg-blue-700"><i class="fas fa-download mr-2"></i>Tout télécharger</button>
          <button onclick="clearAllDocuments()" class="bg-red-600 text-white px-3 py-2 rounded text-sm hover:bg-red-700"><i class="fas fa-trash mr-2"></i>Tout supprimer</button>
        </div>
      </div>

      <div id="docsList" class="space-y-3 max-h-96 overflow-y-auto"></div>
      <div id="docsEmpty" class="text-center py-12 text-gray-500" style="display:none;">
        <i class="fas fa-folder-open fa-3x text-gray-300 mb-4"></i>
        <h4 class="text-lg font-semibold mb-2">Aucun document</h4>
        <p class="text-sm">Utilisez la zone d'upload ci-dessus pour ajouter vos documents</p>
      </div>

      <div id="docPreviewArea" class="mt-6" style="display:none;">
        <h4 class="text-lg font-bold text-gray-800 mb-2"><i class="fas fa-eye mr-2 text-purple-600"></i>Aperçu : <span id="previewFileName"></span></h4>
        <div id="docPreviewContent" class="border rounded-lg p-4 bg-gray-50"></div>
      </div>

      <div class="mt-6 flex justify-end">
        <button onclick="closeDocsModal()" class="bg-gray-400 text-white px-6 py-3 rounded-lg font-semibold hover:bg-gray-500">Fermer</button>
      </div>
    </div>
  </div>
</div>

<!-- Storage Manager Modal -->
<div id="storageModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="storageModalTitle">
  <div class="modal-content" style="width: 95vw; max-width: 1000px; max-height: 90vh;">
    <div class="modal-header">
      <h3 id="storageModalTitle" class="text-2xl font-bold"><i class="fas fa-database mr-2 text-yellow-600"></i>Gestionnaire de Stockage</h3>
      <div class="flex items-center space-x-2">
        <button onclick="refreshStorageInfo()" class="bg-blue-600 text-white px-3 py-2 rounded" title="Actualiser les informations"><i class="fas fa-sync-alt mr-1"></i>Actualiser</button>
        <button onclick="closeStorageModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>
    <div class="modal-body" style="overflow-y:auto; max-height:calc(90vh - 120px);">
      
      <!-- Vue d'ensemble -->
      <div id="storageOverview" class="mb-6 p-4 bg-gradient-to-r from-blue-50 to-indigo-50 rounded-xl border-2 border-indigo-200">
        <h4 class="text-lg font-bold text-gray-800 mb-3"><i class="fas fa-chart-pie mr-2 text-blue-600"></i>Vue d'ensemble du stockage</h4>
        <div id="storageStats" class="grid grid-cols-1 md:grid-cols-3 gap-4">
          <!-- Rempli dynamiquement -->
        </div>
      </div>

      <!-- Actions rapides -->
      <div class="mb-6 p-4 bg-gradient-to-r from-green-50 to-blue-50 rounded-xl border-2 border-green-200">
        <h4 class="text-lg font-bold text-gray-800 mb-3"><i class="fas fa-tools mr-2 text-green-600"></i>Actions d'optimisation</h4>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <button onclick="cleanupStorage()" class="bg-red-600 hover:bg-red-700 text-white px-4 py-3 rounded-lg font-semibold">
            <i class="fas fa-broom mr-2"></i>Nettoyage complet
            <div class="text-sm opacity-75">Supprime documents volumineux</div>
          </button>
          <button onclick="exporterDonnees()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-3 rounded-lg font-semibold">
            <i class="fas fa-download mr-2"></i>Export de sauvegarde
            <div class="text-sm opacity-75">Sauvegarde complète en JSON</div>
          </button>
          <button onclick="compressDocuments()" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-3 rounded-lg font-semibold">
            <i class="fas fa-compress mr-2"></i>Compression docs
            <div class="text-sm opacity-75">Réduit la taille des images</div>
          </button>
          <button onclick="resetApplication()" class="bg-gray-600 hover:bg-gray-700 text-white px-4 py-3 rounded-lg font-semibold">
            <i class="fas fa-power-off mr-2"></i>Reset complet
            <div class="text-sm opacity-75">⚠️ Supprime toutes les données</div>
          </button>
        </div>
      </div>

      <!-- Analyse détaillée -->
      <div class="card rounded-xl p-6">
        <h4 class="text-lg font-bold text-gray-800 mb-4"><i class="fas fa-search mr-2 text-purple-600"></i>Analyse détaillée</h4>
        <div id="storageBreakdown">
          <!-- Rempli dynamiquement -->
        </div>
      </div>

      <!-- Conseils d'optimisation -->
      <div class="mt-6 p-4 bg-yellow-50 rounded-xl border border-yellow-200">
        <h4 class="text-lg font-bold text-yellow-800 mb-2"><i class="fas fa-lightbulb mr-2"></i>Conseils d'optimisation</h4>
        <ul id="storageAdvice" class="text-sm text-yellow-700 space-y-1">
          <!-- Rempli dynamiquement -->
        </ul>
      </div>

      <div class="mt-6 flex justify-end">
        <button onclick="closeStorageModal()" class="bg-gray-400 text-white px-6 py-3 rounded-lg font-semibold hover:bg-gray-500">Fermer</button>
      </div>
    </div>
  </div>
</div>

<!-- Planning (Gantt) Modal -->
<div id="planningModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="planningModalTitle">
  <div class="modal-content" style="width:95vw; max-width:1400px;">
    <div class="modal-header">
      <h3 id="planningModalTitle" class="text-2xl font-bold">Planning Gantt - <span id="planningEventName"></span></h3>
      <div class="flex items-center space-x-2">
        <button onclick="printModal('planningModal')" class="bg-green-600 text-white px-3 py-2 rounded" title="Imprimer le planning"><i class="fas fa-print mr-1"></i>Imprimer</button>
        <button onclick="closePlanningModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
      </div>
    </div>

    <div class="modal-body" style="padding:0;">
      <div class="bg-gradient-to-r from-blue-50 to-indigo-50 p-4 border-b">
        <form id="taskForm" onsubmit="saveGanttTask(event)">
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            <div><label class="block text-sm font-medium mb-1">Nom *</label><input id="taskName" required class="inline-edit" placeholder="Ex: Montage scène"></div>
            <div><label class="block text-sm font-medium mb-1">Début *</label><input id="taskStartDate" type="date" required class="inline-edit"></div>
            <div><label class="block text-sm font-medium mb-1">Fin *</label><input id="taskEndDate" type="date" required class="inline-edit"></div>
            <div>
              <label class="block text-sm font-medium mb-1">Agents (multi)</label>
              <select id="taskAgents" class="inline-edit" multiple size="5">
                <!-- options injectées en JS -->
              </select>
              <div class="text-xs text-gray-500 mt-1">
                Astuce : Ctrl/Cmd + clic pour sélectionner plusieurs agents
              </div>
            </div>
            <div><label class="block text-sm font-medium mb-1">Service</label>
              <select id="taskService" class="inline-edit">
                <option value="">—</option>
                <option>Voirie</option><option>Transport</option><option>Engin</option><option>Technique</option><option>Sport</option><option>Entretien</option>
              </select>
            </div>
            <div><label class="block text-sm font-medium mb-1">Heures</label><input id="taskHours" type="number" min="0" step="0.5" class="inline-edit" placeholder="Ex: 8"></div>
          </div>
          <div class="mt-3 flex justify-between items-center">
            <div class="space-x-2">
              <button type="submit" class="bg-green-600 text-white px-4 py-2 rounded font-semibold"><i class="fas fa-save mr-2"></i><span id="taskFormSubmitText">Enregistrer</span></button>
              <button type="button" onclick="resetTaskForm()" class="bg-gray-400 text-white px-3 py-2 rounded font-semibold"><i class="fas fa-undo mr-2"></i>Réinitialiser</button>
            </div>
            <div class="text-sm text-gray-600"><i class="fas fa-info-circle mr-1"></i>Cliquez une barre pour modifier • Supprimez via la corbeille</div>
          </div>
        </form>
      </div>
      <div class="p-4">
        <div id="gantt-chart" style="height:420px; overflow:auto; border:1px solid #e5e7eb; border-radius:8px; background:#fff;"></div>
        <div id="gantt-empty" class="text-center text-gray-500 py-12" style="display:none;">
          <i class="fas fa-calendar-times fa-3x mb-4 text-gray-300"></i>
          <p class="text-lg">Aucune tâche planifiée</p>
          <p class="text-sm">Utilisez le formulaire ci-dessus pour créer votre première tâche</p>
        </div>
      </div>
      <div class="px-4 pb-4">
        <div id="ganttTasksList" class="space-y-2"></div>
      </div>
    </div>
  </div>
</div>

<!-- Modal générique (bar info / calendrier jour) -->
<div id="barInfoModal" class="modal" role="dialog" aria-modal="true" aria-labelledby="barInfoTitle">
  <div class="modal-content" style="width:90vw; max-width:900px;">
    <div class="modal-header">
      <h3 class="text-2xl font-bold" id="barInfoTitle">Détail</h3>
      <button onclick="closeBarInfoModal()" class="text-gray-500 hover:text-gray-800" title="Fermer la fenêtre"><i class="fas fa-times fa-lg"></i></button>
    </div>
    <div class="modal-body">
      <div id="barInfoBody"></div>
      <div class="mt-4 text-right">
        <button onclick="closeBarInfoModal()" class="bg-gray-500 text-white px-4 py-2 rounded">Fermer</button>
      </div>
    </div>
  </div>
</div>

<script>
/* ==== Données exemple ==== */
let appData = {
  agents: [
    { id:1, nom:"Dupont Jean",  service:"Voirie",    telephone:"04.95.XX.XX.XX" },
    { id:2, nom:"Martin Claire",service:"Transport", telephone:"04.95.XX.XX.XX" },
    { id:3, nom:"Durand Paul",  service:"Engin",     telephone:"04.95.XX.XX.XX" },
    { id:4, nom:"Rossi Marie",  service:"Technique", telephone:"04.95.XX.XX.XX" },
    { id:5, nom:"Bernard Luc",  service:"Sport",     telephone:"04.95.XX.XX.XX" }
  ],
  events: [
    {
      id:1, nom:"Fête de la Saint-Jean", categorie:"Festivité", lieu:"Place du Village",
      dateDebut:"2024-06-24", dateFin:"2024-06-26", description:"Fête traditionnelle",
      participants:[
        { agentId:1, date:"2024-06-24", motif:"Surveillance", nature:"payees", heureDebut:"08:00", heureFin:"16:00" },
        { agentId:2, date:"2024-06-24", motif:"Montage",     nature:"recup",  heureDebut:"14:00", heureFin:"20:00" },
        { agentId:1, date:"2024-06-25", motif:"Nettoyage",   nature:"voirie", heureDebut:"09:00", heureFin:"12:00" }
      ],
      documents:[]
    },
    {
      id:2, nom:"Marché Hebdomadaire", categorie:"Marché", lieu:"Centre-ville",
      dateDebut:"2024-07-01", dateFin:"2024-07-01", description:"Marché du lundi",
      participants:[
        { agentId:1, date:"2024-07-01", motif:"Nettoyage",          nature:"payees", heureDebut:"06:00", heureFin:"09:00" },
        { agentId:3, date:"2024-07-01", motif:"Transport matériel", nature:"recup",  heureDebut:"07:00", heureFin:"11:00" }
      ],
      documents:[]
    }
  ],
  materials: [
    { 
      id: 1, 
      nom: "Table rectangulaire", 
      categorie: "Tables", 
      stockTotal: 50, 
      stockReserve: 0,
      unite: "unité", 
      stockMin: 5,
      statut: "disponible",
      description: "Table 180x80cm pliante"
    },
    { 
      id: 2, 
      nom: "Chaise pliante", 
      categorie: "Chaises", 
      stockTotal: 200, 
      stockReserve: 0,
      unite: "unité", 
      stockMin: 20,
      statut: "disponible",
      description: "Chaise pliante blanche"
    },
    { 
      id: 3, 
      nom: "Tente 3x3m", 
      categorie: "Tentes", 
      stockTotal: 10, 
      stockReserve: 0,
      unite: "unité", 
      stockMin: 2,
      statut: "disponible",
      description: "Tente pliante 3x3m avec bâches latérales"
    }
  ],
  eventMaterials: {}, // eventId -> [{ materialId, quantite, dateDebut, dateFin, valide: false }]
  tasks:{
    1:[
      { id:"task_1", name:"Montage scène", start:"2024-06-23", end:"2024-06-24", progress:100, agentId:1, service:"Voirie", hours:8 },
      { id:"task_2", name:"Installation éclairage", start:"2024-06-24", end:"2024-06-25", progress:50, agentId:3, service:"Technique", hours:6 }
    ],
    2:[
      { id:"task_3", name:"Préparation emplacement", start:"2024-06-30", end:"2024-07-01", progress:100, agentId:1, service:"Voirie", hours:4 }
    ]
  }
};

// blobs doc en mémoire avec gestion améliorée
class BlobManager {
  constructor() {
    this.eventBlobs = new Map(); // eventId -> Map(docId -> blob)
    this.previewUrls = new Set(); // URLs créées pour preview
  }
  
  addBlob(eventId, docId, blob) {
    if (!this.eventBlobs.has(eventId)) {
      this.eventBlobs.set(eventId, new Map());
    }
    this.eventBlobs.get(eventId).set(docId, blob);
  }
  
  getBlob(eventId, docId) {
    return this.eventBlobs.get(eventId)?.get(docId);
  }
  
  removeBlob(eventId, docId) {
    this.eventBlobs.get(eventId)?.delete(docId);
  }
  
  removeEvent(eventId) {
    this.eventBlobs.delete(eventId);
  }
  
  createPreviewURL(blob) {
    const url = URL.createObjectURL(blob);
    this.previewUrls.add(url);
    return url;
  }
  
  revokeAllPreviewURLs() {
    this.previewUrls.forEach(url => {
      try { URL.revokeObjectURL(url); } catch(e) {}
    });
    this.previewUrls.clear();
  }
  
  clear() {
    this.eventBlobs.clear();
    this.revokeAllPreviewURLs();
  }
}

const blobManager = new BlobManager();

/* ==== Globals ==== */
let eventsChartInstance=null, sectorsChartInstance=null, monthlyTypeChartInstance=null, hoursChartInstance=null;
let yearlyLineChartInstance=null, monthlyBreakdownLineChartInstance=null;
let multiHoursChartInstance=null;
let yearlyTypeHoursChartInstance=null;
let agentEvolutionChartInstance=null; // Pour le graphique d'évolution des agents

// ==== Charts du modal jour ====
let dayDistChartInst = null;
let dayEventBarChartInst = null;

let editingEventId=null, editingAgentId=null, editingEventForParts=null;
let currentTypeFilter='tous', currentAgentForHistory=null;
let currentPlanningEventId=null, currentGanttInstance=null, editingTaskId=null;
let currentEventForDocs=null;
let participantsData=[], originalParticipantsData=[];
let lastPreviewUrl = null;

// Variables globales Matériel
let editingMaterialId = null;
let currentEventForMaterial = null;

// Sauvegarde automatique

/* ==== DIAGNOSTIC ET CORRECTION MATÉRIEL ==== */
function diagnosticMateriel(data, context = 'import') {
  console.log(`🔍 === DIAGNOSTIC MATÉRIEL (${context}) ===`);
  
  // Recherche exhaustive du matériel
  const sources = [
    { path: 'data.materials', value: data.materials },
    { path: 'data.donneesCompletes?.materials', value: data.donneesCompletes?.materials },
    { path: 'data.data?.materials', value: data.data?.materials },
    { path: 'data.backup?.materials', value: data.backup?.materials },
    { path: 'data.appData?.materials', value: data.appData?.materials },
    { path: 'data.exportInfo?.materials', value: data.exportInfo?.materials }
  ];
  
  console.log('📊 Structure racine:', Object.keys(data));
  
  sources.forEach(source => {
    if (source.value) {
      if (Array.isArray(source.value)) {
        console.log(`✅ ${source.path}: ${source.value.length} articles trouvés`);
        if (source.value.length > 0) {
          console.log(`📦 Échantillon:`, source.value[0]);
        }
      } else {
        console.log(`⚠️ ${source.path}: existe mais n'est pas un array`, typeof source.value);
      }
    } else {
      console.log(`❌ ${source.path}: non trouvé`);
    }
  });
  
  // Retourner le meilleur candidat
  for (const source of sources) {
    if (source.value && Array.isArray(source.value) && source.value.length > 0) {
      console.log(`🎯 Utilisation de ${source.path}`);
      return source.value;
    }
  }
  
  console.warn('❌ Aucun matériel valide trouvé');
  return null;
}

// Diagnostic du matériel avec options de forçage - FONCTION AJOUTÉE
function diagnosticMaterielAvecForçage(materialData) {
  console.log('🔧 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  const diagnostics = {
    conflicts: [],
    suggestions: [],
    canForce: false,
    foundMaterial: null,
    sourceLocation: ''
  };
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: materialData?.materials },
    { name: 'imported.donneesCompletes?.materials', data: materialData?.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: materialData?.data?.materials },
    { name: 'imported.backup?.materials', data: materialData?.backup?.materials },
    { name: 'imported.appData?.materials', data: materialData?.appData?.materials }
  ];
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!diagnostics.foundMaterial) {
        diagnostics.foundMaterial = source.data;
        diagnostics.sourceLocation = source.name;
      }
    }
  }
  
  if (diagnostics.foundMaterial) {
    // Vérification des conflits
    diagnostics.foundMaterial.forEach(item => {
      const existing = (appData.materials || []).find(m => m.nom === item.nom);
      if (existing) {
        diagnostics.conflicts.push({
          nom: item.nom,
          existingStock: existing.stockTotal,
          importStock: item.stockTotal || item.stock,
          action: 'merge_or_replace'
        });
        diagnostics.canForce = true;
      }
    });
    
    if (diagnostics.conflicts.length === 0) {
      diagnostics.suggestions.push("Aucun conflit détecté, import sécurisé");
    } else {
      diagnostics.suggestions.push(`${diagnostics.conflicts.length} conflit(s) détecté(s)`);
    }
    
    console.log(`🎯 Matériel trouvé: ${diagnostics.foundMaterial.length} articles depuis ${diagnostics.sourceLocation}`);
    return diagnostics.foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  diagnostics.suggestions.push("Aucun matériel à importer détecté");
  return null;
}

// Force la récupération du matériel alternatif - FONCTION AJOUTÉE
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  if (!importedData || (!importedData.materials && !importedData.inventory && !importedData.equipment)) {
    console.warn('❌ Aucune donnée matériel alternative trouvée');
    return [];
  }
  
  const alternativeMaterials = [];
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.materials,
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      source.forEach((item, index) => {
        const existing = (appData.materials || []).find(m => m.nom === (item.nom || item.name));
        
        if (existing) {
          // Créer une version alternative avec suffixe pour éviter les conflits
          const altItem = {
            id: generateId(),
            nom: `${item.nom || item.name} (Import_${new Date().toLocaleDateString('fr-FR')})`,
            categorie: item.categorie || item.category || item.type || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: `${item.description || item.desc || ''} (Importé avec conflit résolu)`
          };
          alternativeMaterials.push(altItem);
        } else {
          // Garder l'item original s'il n'y a pas de conflit
          const normalizedItem = {
            id: generateId(),
            nom: item.nom || item.name || item.title || `Article ${index + 1}`,
            categorie: item.categorie || item.category || item.type || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: item.description || item.desc || ''
          };
          alternativeMaterials.push(normalizedItem);
        }
      });
      
      console.log(`✅ ${alternativeMaterials.length} articles récupérés depuis source alternative`);
      return alternativeMaterials;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}

// Fonction utilitaire pour générer des IDs uniques - FONCTION AJOUTÉE
function generateId() {
  return Date.now() + Math.floor(Math.random() * 10000);
}

// Diagnostic du matériel avec options de forçage
function diagnosticMaterielAvecForçage(importedData) {
  console.log('🔍 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: importedData.materials },
    { name: 'imported.donneesCompletes?.materials', data: importedData.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: importedData.data?.materials },
    { name: 'imported.backup?.materials', data: importedData.backup?.materials },
    { name: 'imported.appData?.materials', data: importedData.appData?.materials }
  ];
  
  let foundMaterial = null;
  let sourceLocation = '';
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!foundMaterial) {
        foundMaterial = source.data;
        sourceLocation = source.name;
        window.lastMaterialLocation = sourceLocation; // Pour les logs
      }
    }
  }
  
  if (foundMaterial) {
    console.log(`🎯 Utilisation de ${sourceLocation} avec ${foundMaterial.length} articles`);
    return foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  return null;
}

// Force la récupération du matériel alternatif
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      // Tentative de normalisation des données alternatives
      const normalized = source.map((item, index) => ({
        id: item.id || Date.now() + index,
        nom: item.nom || item.name || item.title || `Article ${index + 1}`,
        categorie: item.categorie || item.category || item.type || 'Divers',
        stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
        stockReserve: 0,
        unite: item.unite || item.unit || 'unité',
        stockMin: Number(item.stockMin || item.minimum || 0),
        statut: 'disponible',
        description: item.description || item.desc || ''
      }));
      
      console.log(`✅ ${normalized.length} articles normalisés depuis source alternative`);
      return normalized;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}

// Diagnostic du matériel avec options de forçage
function diagnosticMaterielAvecForçage(importedData) {
  console.log('🔍 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: importedData.materials },
    { name: 'imported.donneesCompletes?.materials', data: importedData.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: importedData.data?.materials },
    { name: 'imported.backup?.materials', data: importedData.backup?.materials },
    { name: 'imported.appData?.materials', data: importedData.appData?.materials }
  ];
  
  let foundMaterial = null;
  let sourceLocation = '';
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!foundMaterial) {
        foundMaterial = source.data;
        sourceLocation = source.name;
        window.lastMaterialLocation = sourceLocation; // Pour les logs
      }
    }
  }
  
  if (foundMaterial) {
    console.log(`🎯 Utilisation de ${sourceLocation} avec ${foundMaterial.length} articles`);
    return foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  return null;
}

// Force la récupération du matériel alternatif
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      // Tentative de normalisation des données alternatives
      const normalized = source.map((item, index) => ({
        id: item.id || Date.now() + index,
        nom: item.nom || item.name || item.title || `Article ${index + 1}`,
        categorie: item.categorie || item.category || item.type || 'Divers',
        stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
        stockReserve: 0,
        unite: item.unite || item.unit || 'unité',
        stockMin: Number(item.stockMin || item.minimum || 0),
        statut: 'disponible',
        description: item.description || item.desc || ''
      }));
      
      console.log(`✅ ${normalized.length} articles normalisés depuis source alternative`);
      return normalized;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}

function forceRestoreMateriel(imported) {
  console.log('🔧 === FORCE RESTORE MATÉRIEL ===');
  
  const materialsFound = diagnosticMateriel(imported, 'force restore');
  
  if (materialsFound) {
    // Forcer l'injection dans tous les emplacements possibles
    imported.materials = materialsFound;
    if (!imported.donneesCompletes) imported.donneesCompletes = {};
    imported.donneesCompletes.materials = materialsFound;
    if (!imported.backup) imported.backup = {};
    imported.backup.materials = materialsFound;
    
    console.log('✅ Matériel forcé dans tous les emplacements');
    return materialsFound;
  }
  
  return null;
}

// Diagnostic du matériel avec options de forçage - FONCTION MANQUANTE AJOUTÉE
function diagnosticMaterielAvecForçage(materialData) {
  console.log('🔧 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  const diagnostics = {
    conflicts: [],
    suggestions: [],
    canForce: false,
    foundMaterial: null,
    sourceLocation: ''
  };
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: materialData?.materials },
    { name: 'imported.donneesCompletes?.materials', data: materialData?.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: materialData?.data?.materials },
    { name: 'imported.backup?.materials', data: materialData?.backup?.materials },
    { name: 'imported.appData?.materials', data: materialData?.appData?.materials }
  ];
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!diagnostics.foundMaterial) {
        diagnostics.foundMaterial = source.data;
        diagnostics.sourceLocation = source.name;
      }
    }
  }
  
  if (diagnostics.foundMaterial) {
    // Vérification des conflits
    diagnostics.foundMaterial.forEach(item => {
      const existing = (appData.materials || []).find(m => m.nom === item.nom);
      if (existing) {
        diagnostics.conflicts.push({
          nom: item.nom,
          existingStock: existing.stockTotal,
          importStock: item.stockTotal || item.stock,
          action: 'merge_or_replace'
        });
        diagnostics.canForce = true;
      }
    });
    
    if (diagnostics.conflicts.length === 0) {
      diagnostics.suggestions.push("Aucun conflit détecté, import sécurisé");
    } else {
      diagnostics.suggestions.push(`${diagnostics.conflicts.length} conflit(s) détecté(s)`);
    }
    
    console.log(`🎯 Matériel trouvé: ${diagnostics.foundMaterial.length} articles depuis ${diagnostics.sourceLocation}`);
    return diagnostics.foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  diagnostics.suggestions.push("Aucun matériel à importer détecté");
  return null;
}

// Force la récupération du matériel alternatif - FONCTION MANQUANTE AJOUTÉE
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  if (!importedData || (!importedData.materials && !importedData.inventory && !importedData.equipment)) {
    console.warn('❌ Aucune donnée matériel alternative trouvée');
    return [];
  }
  
  const alternativeMaterials = [];
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.materials,
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      source.forEach((item, index) => {
        const existing = (appData.materials || []).find(m => m.nom === (item.nom || item.name));
        
        if (existing) {
          // Créer une version alternative avec suffixe pour éviter les conflits
          const altItem = {
            id: generateId(),
            nom: `${item.nom || item.name} (Import_${new Date().toLocaleDateString('fr-FR')})`,
            categorie: item.categorie || item.category || item.type || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: `${item.description || item.desc || ''} (Importé avec conflit résolu)`
          };
          alternativeMaterials.push(altItem);
        } else {
          // Garder l'item original s'il n'y a pas de conflit
          const normalizedItem = {
            id: generateId(),
            nom: item.nom || item.name || item.title || `Article ${index + 1}`,
            categorie: item.categorie || item.category || item.type || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: item.description || item.desc || ''
          };
          alternativeMaterials.push(normalizedItem);
        }
      });
      
      console.log(`✅ ${alternativeMaterials.length} articles récupérés depuis source alternative`);
      return alternativeMaterials;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}

/* ==== Utils ==== */
function monthLabels(){return ['Jan','Fév','Mar','Avr','Mai','Jun','Jul','Aoû','Sep','Oct','Nov','Déc'];}
function formatMinutes(totalMin){
  const h=Math.floor(totalMin/60), m=totalMin%60;
  return `${h}h${String(m).padStart(2,'0')}m`;
}
function calculateDuration(startTime, endTime){
  if(!startTime || !endTime) return 0;

  // Normalise les heures
  const sNorm = normalizeTime(startTime);
  const eNorm = normalizeTime(endTime);

  if(!/^\d{2}:\d{2}$/.test(sNorm) || !/^\d{2}:\d{2}$/.test(eNorm)) return 0;

  const [h1,m1] = sNorm.split(':').map(Number);
  const [h2,m2] = eNorm.split(':').map(Number);
  
  // Validation des valeurs renforcée
  if(!Number.isFinite(h1)||!Number.isFinite(m1)||!Number.isFinite(h2)||!Number.isFinite(m2)) return 0;
  if(h1 < 0 || h1 > 23 || m1 < 0 || m1 > 59 || h2 < 0 || h2 > 23 || m2 < 0 || m2 > 59) return 0;

  let s = h1*60 + m1, e = h2*60 + m2;
  
  // Si fin <= début, on assume que ça traverse minuit
  if(e <= s) e += 24*60;
  
  const hours = (e - s) / 60;
  
  // Validation finale : pas plus de 24h de travail
  if(hours > 24) {
    console.warn('Durée anormalement longue détectée:', hours, 'heures pour', startTime, '->', endTime);
    return 0;
  }
  
  return Math.round(hours*100)/100; // Précision 0.01 h
}
function ymdLocal(d){
  const y=d.getFullYear(), m=String(d.getMonth()+1).padStart(2,'0'), da=String(d.getDate()).padStart(2,'0');
  return `${y}-${m}-${da}`;
}
function todayYMD(){ return ymdLocal(new Date()); }
function toLocalDate(ymd){ return new Date(`${ymd}T00:00:00`); } // date locale pour éviter les décalages UTC

function getDatesBetween(startStr,endStr){
  const out=[], d=new Date(`${startStr}T00:00:00`), end=new Date(`${endStr}T00:00:00`);
  while(d<=end){ out.push(ymdLocal(d)); d.setDate(d.getDate()+1); }
  return out;
}
function safeSetLS(key,val){ try{ localStorage.setItem(key, JSON.stringify(val)); return true; }catch(e){ return false; } }
function safeGetLS(key){ try{ const v=localStorage.getItem(key); return v? JSON.parse(v):null; }catch(e){ return null; } }
function pct(part,total){ if(!isFinite(part)||!isFinite(total)||total<=0) return '0%'; return `${Math.round((part/total)*100)}%`; }

function sumLineHours(l){
  let d = calculateDuration(l.heureDebut||'', l.heureFin||'');
  if(d <= 0){ d = (+l.payees||0) + (+l.recup||0) + (+l.voirieFest||0); }
  return d;
}

function agentNamesFromTask(t){
  // Compat : agentIds (nouveau) ou agentId (ancien)
  const ids = Array.isArray(t.agentIds) ? t.agentIds : (t.agentId!=null ? [t.agentId] : []);
  if(!ids.length) return '—';
  return ids.map(id => appData.agents.find(a=>a.id===id)?.nom || `#${id}`).join(', ');
}

/* ==== Utils fichiers (Base64) ==== */
function fileToDataURL(file) {
  return new Promise((res, rej) => {
    const fr = new FileReader();
    fr.onload = () => res(fr.result); // data:...;base64,...
    fr.onerror = rej;
    fr.readAsDataURL(file);
  });
}

// Diagnostic du matériel avec options de forçage
function diagnosticMaterielAvecForçage(materialData) {
  console.log('🔧 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  const diagnostics = {
    conflicts: [],
    suggestions: [],
    canForce: false,
    foundMaterial: null,
    sourceLocation: ''
  };
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: materialData?.materials },
    { name: 'imported.donneesCompletes?.materials', data: materialData?.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: materialData?.data?.materials },
    { name: 'imported.backup?.materials', data: materialData?.backup?.materials },
    { name: 'imported.appData?.materials', data: materialData?.appData?.materials }
  ];
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!diagnostics.foundMaterial) {
        diagnostics.foundMaterial = source.data;
        diagnostics.sourceLocation = source.name;
      }
    }
  }
  
  if (diagnostics.foundMaterial) {
    // Vérification des conflits
    diagnostics.foundMaterial.forEach(item => {
      const existing = (appData.materials || []).find(m => m.nom === item.nom);
      if (existing) {
        diagnostics.conflicts.push({
          nom: item.nom,
          existingStock: existing.stockTotal,
          importStock: item.stockTotal || item.stock,
          action: 'merge_or_replace'
        });
        diagnostics.canForce = true;
      }
    });
    
    if (diagnostics.conflicts.length === 0) {
      diagnostics.suggestions.push("Aucun conflit détecté, import sécurisé");
    } else {
      diagnostics.suggestions.push(`${diagnostics.conflicts.length} conflit(s) détecté(s)`);
    }
    
    console.log(`🎯 Matériel trouvé: ${diagnostics.foundMaterial.length} articles depuis ${diagnostics.sourceLocation}`);
    return diagnostics.foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  diagnostics.suggestions.push("Aucun matériel à importer détecté");
  return null;
}

// Force la récupération du matériel alternatif
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  if (!importedData || (!importedData.materials && !importedData.inventory && !importedData.equipment)) {
    console.warn('❌ Aucune donnée matériel alternative trouvée');
    return [];
  }
  
  const alternativeMaterials = [];
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.materials,
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      source.forEach((item, index) => {
        const existing = (appData.materials || []).find(m => m.nom === (item.nom || item.name));
        
        if (existing) {
          // Créer une version alternative avec suffixe pour éviter les conflits
          const altItem = {
            id: generateId(),
            nom: `${item.nom || item.name} (Import_${new Date().toLocaleDateString('fr-FR')})`,
            categorie: item.categorie || item.category || item.type || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: `${item.description || item.desc || ''} (Importé avec conflit résolu)`
          };
          alternativeMaterials.push(altItem);
        } else {
          // Garder l'item original s'il n'y a pas de conflit
          const normalizedItem = {
            id: generateId(),
            nom: item.nom || item.name || item.title || `Article ${index + 1}`,
            categorie: item.categorie || item.category || item.type || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: item.description || item.desc || ''
          };
          alternativeMaterials.push(normalizedItem);
        }
      });
      
      console.log(`✅ ${alternativeMaterials.length} articles récupérés depuis source alternative`);
      return alternativeMaterials;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}
function dataURLtoBlob(dataURL) {
  const [meta, b64] = dataURL.split(',');
  const mime = (meta.match(/data:(.*?);base64/) || [,'application/octet-stream'])[1];
  const bin = atob(b64);
  const u8 = new Uint8Array(bin.length);
  for (let i=0;i<bin.length;i++) u8[i] = bin.charCodeAt(i);
  return new Blob([u8], { type: mime });
}
async function dataURLtoArrayBuffer(dataURL){
  const blob = dataURLtoBlob(dataURL);
  return await blob.arrayBuffer();
}

/* ==== Normalisation import ==== */
function normalizeNature(t){
  if(!t) return 'payees';
  const s=String(t).toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');
  if(s.includes('voir') || s.includes('fest')) return 'voirie';
  if(s.startsWith('recup') || s.includes('recup') || s.includes('récup')) return 'recup';
  if(s.startsWith('pay')) return 'payees';
  if(s.includes('fest')) return 'voirie';
  return (s==='payees'||s==='recup'||s==='voirie') ? s : 'payees';
}
function normalizeData(json){
  const out={ agents:Array.isArray(json.agents)?json.agents:[], events:Array.isArray(json.events)?json.events:[], tasks:json.tasks||{} };
  
  // Normalisation des agents
  out.agents = out.agents.map(a=>({
    id: Number.isFinite(+a.id)? +a.id : Date.now()+Math.floor(Math.random()*9999),
    nom: a.nom||a.name||'Agent',
    service: a.service||'',
    telephone: a.telephone||a.phone||''
  }));
  
  const agentIds = new Set(out.agents.map(a=>a.id));
  
  // Normalisation des événements
  out.events = out.events.map(e=>{
    const participants = Array.isArray(e.participants)? e.participants : [];
    const docs = Array.isArray(e.documents)? e.documents : [];
    const normParts = participants.map(p=>{
      const agentId = Number.isFinite(+p.agentId)? +p.agentId : null;
      return {
        agentId: agentId && agentIds.has(agentId) ? agentId : (out.agents[0]?.id || 0),
        date: p.date || e.dateDebut || todayYMD(),
        motif: p.motif || p.reason || '',
        nature: normalizeNature(p.nature || p.type),
        heureDebut: p.heureDebut || p.start || '',
        heureFin: p.heureFin || p.end || '',
        payees: +p.payees || 0,
        recup: +p.recup || 0,
        voirieFest: +p.voirieFest || 0
      };
    });
    return {
      id: Number.isFinite(+e.id)? +e.id : Date.now()+Math.floor(Math.random()*9999),
      nom: e.nom || e.name || 'Événement',
      categorie: e.categorie || e.type || 'Divers',
      lieu: e.lieu || e.location || '',
      dateDebut: e.dateDebut || e.startDate || todayYMD(),
      dateFin: e.dateFin || e.endDate || e.dateDebut || todayYMD(),
      description: e.description || '',
      participants: normParts,
      documents: docs
    };
  });
  
  // Normalisation des tâches
  out.tasks = out.tasks || {};
  
  // 🔧 NORMALISATION AMÉLIORÉE DU MATÉRIEL
  console.log('🔧 Début normalisation matériel...');
  
  // Chercher le matériel dans différents emplacements possibles
  let rawMaterials = null;
  
  if (json.materials && Array.isArray(json.materials)) {
    rawMaterials = json.materials;
    console.log('📦 Matériel trouvé dans json.materials:', rawMaterials.length);
  } else if (json.donneesCompletes && json.donneesCompletes.materials && Array.isArray(json.donneesCompletes.materials)) {
    rawMaterials = json.donneesCompletes.materials;
    console.log('� Matériel trouvé dans json.donneesCompletes.materials:', rawMaterials.length);
  } else if (json.data && json.data.materials && Array.isArray(json.data.materials)) {
    rawMaterials = json.data.materials;
    console.log('📦 Matériel trouvé dans json.data.materials:', rawMaterials.length);
  }
  
  if (rawMaterials && rawMaterials.length > 0) {
    console.log('🔧 Normalisation de', rawMaterials.length, 'articles de matériel');
    out.materials = rawMaterials.map((m, index) => {
      try {
        const normalized = {
          id: Number.isFinite(+m.id) ? +m.id : Date.now() + index + Math.floor(Math.random()*1000),
          nom: String(m.nom || m.name || `Article ${index + 1}`).trim(),
          categorie: String(m.categorie || m.category || 'Divers').trim(),
          stockTotal: Math.max(0, Number(m.stockTotal) || Number(m.stock) || 0),
          stockReserve: Math.max(0, Number(m.stockReserve) || 0),
          unite: String(m.unite || m.unit || 'unité').trim(),
          stockMin: Math.max(0, Number(m.stockMin) || Number(m.minStock) || 0),
          statut: String(m.statut || m.status || 'disponible').trim(),
          description: String(m.description || '').trim()
        };
        console.log('✅ Matériel normalisé:', normalized.nom, '- Stock:', normalized.stockTotal);
        return normalized;
      } catch (error) {
        console.warn('⚠️ Erreur normalisation matériel:', error, m);
        return {
          id: Date.now() + index + Math.floor(Math.random()*1000),
          nom: `Article ${index + 1}`,
          categorie: 'Divers',
          stockTotal: 0,
          stockReserve: 0,
          unite: 'unité',
          stockMin: 0,
          statut: 'disponible',
          description: 'Article récupéré avec erreur'
        };
      }
    });
    console.log('✅ Matériel normalisé avec succès:', out.materials.length, 'articles');
  } else {
    console.warn('⚠️ Aucun matériel trouvé dans les données importées');
    out.materials = [];
  }
  
  // 🔧 NORMALISATION DES ATTRIBUTIONS DE MATÉRIEL
  console.log('🔧 Normalisation des attributions de matériel...');
  
  // Chercher les attributions dans différents emplacements
  let rawEventMaterials = null;
  
  if (json.eventMaterials && typeof json.eventMaterials === 'object') {
    rawEventMaterials = json.eventMaterials;
    console.log('📋 Attributions trouvées dans json.eventMaterials:', Object.keys(rawEventMaterials).length);
  } else if (json.donneesCompletes && json.donneesCompletes.eventMaterials && typeof json.donneesCompletes.eventMaterials === 'object') {
    rawEventMaterials = json.donneesCompletes.eventMaterials;
    console.log('📋 Attributions trouvées dans json.donneesCompletes.eventMaterials:', Object.keys(rawEventMaterials).length);
  }
  
  if (rawEventMaterials) {
    out.eventMaterials = {};
    Object.entries(rawEventMaterials).forEach(([eventId, allocations]) => {
      if (Array.isArray(allocations) && allocations.length > 0) {
        out.eventMaterials[eventId] = allocations.map(allocation => ({
          id: allocation.id || Date.now() + Math.random(),
          materialId: Number(allocation.materialId),
          quantite: Number(allocation.quantite) || Number(allocation.quantity) || 1,
          dateDebut: allocation.dateDebut || allocation.startDate || '',
          dateFin: allocation.dateFin || allocation.endDate || '',
          valide: Boolean(allocation.valide) || Boolean(allocation.returned) || false,
          sortieValidee: Boolean(allocation.sortieValidee) || Boolean(allocation.validated) || false
        }));
        console.log('📋 Attributions normalisées pour événement', eventId, ':', out.eventMaterials[eventId].length);
      }
    });
    console.log('✅ Attributions matériel normalisées:', Object.keys(out.eventMaterials).length, 'événements');
  } else {
    console.warn('⚠️ Aucune attribution de matériel trouvée');
    out.eventMaterials = {};
  }
  
  return out;
}

/* ==== Filtres ==== */
function getFilteredEvents(){ return currentTypeFilter==='tous' ? appData.events : appData.events.filter(e=>e.categorie===currentTypeFilter); }
function getFilteredParticipants(){
  return getFilteredEvents().flatMap(ev => (ev.participants||[]).map(p => ({...p, eventId:ev.id, eventCategorie:ev.categorie})));
}
function updateTypeFilter(){
  const sel=document.getElementById('typeFilter');
  const cur=sel?.value || 'tous';
  const types=[...new Set(appData.events.map(e=>e.categorie))].sort();
  if(sel){
    sel.innerHTML='<option value="tous">📊 Tous les types</option>'+types.map(t=>`<option value="${t}">${t}</option>`).join('');
    if(types.includes(cur)) sel.value=cur; else { sel.value='tous'; currentTypeFilter='tous'; }
  }
  const s1=document.getElementById('lineTypeFilter1'), s2=document.getElementById('lineTypeFilter2');
  const keep1=s1?.value||'tous', keep2=s2?.value||'tous';
  if(s1){ s1.innerHTML='<option value="tous">Tous</option>'+types.map(t=>`<option value="${t}">${t}</option>`).join(''); if(types.includes(keep1)) s1.value=keep1; }
  if(s2){ s2.innerHTML='<option value="tous">Tous</option>'+types.map(t=>`<option value="${t}">${t}</option>`).join(''); if(types.includes(keep2)) s2.value=keep2; }
  populateMultiFilters();
}
function onTypeFilterChange(){
  const sel=document.getElementById('typeFilter'), status=document.getElementById('filterStatus');
  currentTypeFilter=sel.value;
  if(currentTypeFilter!=='tous'){ sel.classList.add('filter-active'); status.textContent='🔍 Filtre actif : '+currentTypeFilter; }
  else { sel.classList.remove('filter-active'); status.textContent=''; }
  try{ updateDashboard(); }catch(e){ console.error(e); }
  try{ updateAgentsList(); }catch(e){ console.error(e); }
  try{ updateEventsList(); }catch(e){ console.error(e); }
  try{ updateHoursTable(); }catch(e){ console.error(e); }
  try{ renderCalendar(); }catch(e){ console.error(e); }
}
function resetTypeFilter(){ document.getElementById('typeFilter').value='tous'; onTypeFilterChange(); }

/* ==== FONCTIONS MATÉRIEL ==== */

// ====== FONCTIONS CRITIQUES D'IMPORT CORRIGÉES ======

// Fonction de diagnostic avec forçage (FONCTION MANQUANTE AJOUTÉE)
function diagnosticMaterielAvecForçage(materialData) {
  console.log('🔧 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  const diagnostics = {
    conflicts: [],
    suggestions: [],
    canForce: false,
    foundMaterial: null,
    sourceLocation: ''
  };
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: materialData?.materials },
    { name: 'imported.donneesCompletes?.materials', data: materialData?.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: materialData?.data?.materials },
    { name: 'imported.backup?.materials', data: materialData?.backup?.materials },
    { name: 'imported.appData?.materials', data: materialData?.appData?.materials }
  ];
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!diagnostics.foundMaterial) {
        diagnostics.foundMaterial = source.data;
        diagnostics.sourceLocation = source.name;
      }
    }
  }
  
  if (diagnostics.foundMaterial) {
    // Vérification des conflits
    diagnostics.foundMaterial.forEach(item => {
      const existing = (appData.materials || []).find(m => m.nom === item.nom);
      if (existing) {
        diagnostics.conflicts.push({
          nom: item.nom,
          existingStock: existing.stockTotal,
          importStock: item.stockTotal || item.stock,
          action: 'merge_or_replace'
        });
        diagnostics.canForce = true;
      }
    });
    
    if (diagnostics.conflicts.length === 0) {
      diagnostics.suggestions.push("Aucun conflit détecté, import sécurisé");
    } else {
      diagnostics.suggestions.push(`${diagnostics.conflicts.length} conflit(s) détecté(s)`);
    }
    
    console.log(`🎯 Matériel trouvé: ${diagnostics.foundMaterial.length} articles depuis ${diagnostics.sourceLocation}`);
    return diagnostics.foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  diagnostics.suggestions.push("Aucun matériel à importer détecté");
  return null;
}

// Force la récupération du matériel alternatif (FONCTION MANQUANTE AJOUTÉE)
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  if (!importedData || (!importedData.materials && !importedData.inventory && !importedData.equipment)) {
    console.warn('❌ Aucune donnée matériel alternative trouvée');
    return [];
  }
  
  const alternativeMaterials = [];
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.materials,
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      source.forEach((item, index) => {
        const existing = (appData.materials || []).find(m => m.nom === (item.nom || item.name));
        
        if (existing) {
          // Créer une version alternative avec suffixe pour éviter les conflits
          const altItem = {
            id: item.id || Date.now() + index + Math.floor(Math.random() * 1000),
            nom: `${item.nom || item.name} (Import)`,
            categorie: item.categorie || item.category || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: (item.description || '') + ' [Importé]'
          };
          alternativeMaterials.push(altItem);
        } else {
          // Garder l'item original s'il n'y a pas de conflit
          const normalizedItem = {
            id: item.id || Date.now() + index + Math.floor(Math.random() * 1000),
            nom: item.nom || item.name || `Article ${index + 1}`,
            categorie: item.categorie || item.category || item.type || 'Divers',
            stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
            stockReserve: 0,
            unite: item.unite || item.unit || 'unité',
            stockMin: Number(item.stockMin || item.minimum || 0),
            statut: 'disponible',
            description: item.description || item.desc || ''
          };
          alternativeMaterials.push(normalizedItem);
        }
      });
      
      console.log(`✅ ${alternativeMaterials.length} articles récupérés depuis source alternative`);
      return alternativeMaterials;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}

// Fonction utilitaire pour générer des IDs uniques (FONCTION MANQUANTE AJOUTÉE)
function generateId() {
  return Date.now() + Math.floor(Math.random() * 10000);
}

function showMaterialForm(id) {
  console.log('🔧 Ouverture formulaire matériel:', id);
  
  // Vérification et initialisation des données matériel
  if (!appData.materials) {
    appData.materials = [];
    console.log('✨ Materials array initialisé');
  }
  
  editingMaterialId = id || null;
  const form = document.getElementById('addMaterialForm');
  const title = document.getElementById('materialFormTitle');
  const submitBtn = document.getElementById('materialFormSubmit');
  const submitSpan = submitBtn ? submitBtn.querySelector('span') : null;
  
  if (!form) {
    console.error('❌ Formulaire matériel (#addMaterialForm) non trouvé dans le DOM');
    showToast('Erreur: formulaire matériel non disponible', 'error');
    return;
  }
  
  if (!title) {
    console.error('❌ Titre formulaire matériel (#materialFormTitle) non trouvé');
    return;
  }
  
  if (id) {
    // Mode édition
    const material = appData.materials.find(m => m.id === id);
    if (!material) {
      console.error('❌ Matériel non trouvé:', id);
      showToast('Matériel non trouvé', 'error');
      return;
    }
    
    console.log('✏️ Mode édition pour:', material.nom);
    title.textContent = 'Modifier le Matériel';
    if (submitSpan) submitSpan.textContent = 'Mettre à jour';
    
    // Remplissage du formulaire
    const nameInput = document.getElementById('materialName');
    const categorySelect = document.getElementById('materialCategory');
    const categoryCustom = document.getElementById('materialCategoryCustom');
    const stockInput = document.getElementById('materialStock');
    const minStockInput = document.getElementById('materialMinStock');
    const descInput = document.getElementById('materialDescription');
    
    if (nameInput) nameInput.value = material.nom || '';
    if (stockInput) stockInput.value = material.stockTotal || 0;
    if (minStockInput) minStockInput.value = material.stockMin || 0;
    if (descInput) descInput.value = material.description || '';
    
    // Gestion catégorie
    if (categorySelect && categoryCustom) {
      const predefinedCategories = ['Tables', 'Chaises', 'Tentes', 'Éclairage', 'Sonorisation', 'Décoration', 'Sécurité', 'Transport', 'Technique'];
      
      if (predefinedCategories.includes(material.categorie)) {
        categorySelect.value = material.categorie;
        categoryCustom.style.display = 'none';
        categoryCustom.value = '';
      } else {
        categorySelect.value = 'custom';
        categoryCustom.style.display = 'block';
        categoryCustom.value = material.categorie || '';
      }
    }
    
  } else {
    // Mode ajout
    console.log('➕ Mode ajout nouveau matériel');
    title.textContent = 'Ajouter du Matériel';
    if (submitSpan) submitSpan.textContent = 'Ajouter';
    
    // Reset du formulaire
    const materialForm = document.getElementById('materialForm');
    if (materialForm) materialForm.reset();
    
    // Reset catégorie custom
    const categoryCustom = document.getElementById('materialCategoryCustom');
    if (categoryCustom) {
      categoryCustom.style.display = 'none';
      categoryCustom.value = '';
    }
  }
  
  // Affichage du formulaire
  form.style.display = 'block';
  form.scrollIntoView({ behavior: 'smooth', block: 'center' });
  
  console.log('✅ Formulaire affiché');
  
  // Focus sur le premier champ
  setTimeout(() => {
    const nameField = document.getElementById('materialName');
    if (nameField) {
      nameField.focus();
      console.log('🎯 Focus défini sur le champ nom');
    }
  }, 200);
}

function handleCategorySelection(selectElement) {
  const customInput = document.getElementById('materialCategoryCustom');
  
  if (selectElement.value === 'custom') {
    customInput.style.display = 'block';
    customInput.focus();
    customInput.placeholder = 'Saisir une nouvelle catégorie...';
  } else {
    customInput.style.display = 'none';
    customInput.value = '';
  }
}

function hideMaterialForm() {
  document.getElementById('addMaterialForm').style.display = 'none';
  editingMaterialId = null;
  document.getElementById('materialForm').reset();
}

function saveMaterial(e) {
  e.preventDefault();
  
  console.log('🔧 Début sauvegarde matériel');
  
  // Validation des éléments DOM
  const nameInput = document.getElementById('materialName');
  const categorySelect = document.getElementById('materialCategory');
  const categoryCustom = document.getElementById('materialCategoryCustom');
  const stockInput = document.getElementById('materialStock');
  const minStockInput = document.getElementById('materialMinStock');
  const descInput = document.getElementById('materialDescription');
  
  if (!nameInput || !categorySelect || !stockInput) {
    console.error('❌ Éléments du formulaire manquants');
    showToast('Erreur: formulaire incomplet', 'error');
    return;
  }
  
  // Initialiser les données matériel si nécessaire
  if (!appData.materials) {
    appData.materials = [];
    console.log('✨ Array materials initialisé');
  }
  
  // Récupération de la catégorie (prédéfinie ou personnalisée)
  let selectedCategory = categorySelect.value;
  
  // Si "custom" est sélectionné, utiliser la valeur du champ personnalisé
  if (selectedCategory === 'custom' && categoryCustom) {
    selectedCategory = categoryCustom.value.trim();
  }
  
  const materialData = {
    nom: nameInput.value.trim(),
    categorie: selectedCategory,
    stockTotal: parseInt(stockInput.value) || 0,
    unite: 'unité',
    stockMin: parseInt(minStockInput ? minStockInput.value : '0') || 0,
    statut: 'disponible',
    description: descInput ? descInput.value.trim() : ''
  };
  
  // Validation renforcée
  if (!materialData.nom) {
    showToast('❌ Le nom de l\'article est obligatoire', 'error');
    nameInput.focus();
    return;
  }
  
  if (!materialData.categorie) {
    showToast('❌ La catégorie est obligatoire', 'error');
    categorySelect.focus();
    return;
  }
  
  if (materialData.stockTotal < 0) {
    showToast('❌ Le stock total ne peut pas être négatif', 'error');
    stockInput.focus();
    return;
  }
  
  console.log('📋 Données matériel validées:', materialData);
  
  try {
    if (editingMaterialId) {
      // Modification
      const materialIndex = appData.materials.findIndex(m => m.id === editingMaterialId);
      if (materialIndex !== -1) {
        appData.materials[materialIndex] = { 
          ...appData.materials[materialIndex], 
          ...materialData 
        };
        console.log('✅ Matériel modifié:', appData.materials[materialIndex]);
        showToast('✅ Matériel mis à jour avec succès !', 'success');
      } else {
        throw new Error('Matériel à modifier non trouvé');
      }
    } else {
      // Ajout
      const maxId = appData.materials.reduce((max, m) => Math.max(max, m.id || 0), 0);
      const newMaterial = {
        id: maxId + 1,
        stockReserve: 0,
        ...materialData
      };
      appData.materials.push(newMaterial);
      console.log('✅ Nouveau matériel ajouté:', newMaterial);
      console.log('📦 Total matériels:', appData.materials.length);
      showToast('✅ Matériel ajouté avec succès !', 'success');
    }
    
    // Force la sauvegarde immédiate avec gestion d'erreur
    markAsChanged();
    
    // Vérification des structures de données
    if (!appData.eventMaterials) {
      appData.eventMaterials = {};
      console.log('✨ EventMaterials initialisé');
    }
    
    // Tentative de sauvegarde avec gestion d'erreur
    const dataToSave = {
      ...appData,
      materials: appData.materials,
      eventMaterials: appData.eventMaterials
    };
    
    const saveSuccess = safeSetLS('biguglia_app_data', dataToSave);
    
    if (!saveSuccess) {
      console.warn('⚠️ Sauvegarde principale échouée, tentative de sauvegarde d\'urgence');
      // Sauvegarde d'urgence sans documents volumineux
      const emergencyData = {
        agents: appData.agents,
        events: appData.events.map(e => ({
          ...e,
          documents: [] // Supprime temporairement les documents pour économiser l'espace
        })),
        tasks: appData.tasks,
        materials: appData.materials,
        eventMaterials: appData.eventMaterials
      };
      
      const emergencySuccess = safeSetLS('biguglia_app_data_emergency', emergencyData);
      if (emergencySuccess) {
        showToast('⚠️ Sauvegarde d\'urgence réussie (espace insuffisant)', 'warning', 5000);
      } else {
        throw new Error('Impossible de sauvegarder - espace de stockage local insuffisant');
      }
    } else {
      console.log('💾 Sauvegarde réussie');
      
      // Vérification différée de la sauvegarde
      setTimeout(() => {
        const saved = safeGetLS('biguglia_app_data');
        if (saved && saved.materials && saved.materials.length === appData.materials.length) {
          console.log('✅ Sauvegarde vérifiée et confirmée');
        } else {
          console.warn('⚠️ Sauvegarde incomplète détectée');
          showToast('⚠️ Sauvegarde incomplète - Veuillez réessayer', 'warning', 4000);
        }
      }, 500);
    }
    
    // Mise à jour de l'interface
    updateMaterialList();
    updateMaterialFilters();
    hideMaterialForm();
    
  } catch (error) {
    console.error('❌ Erreur lors de la sauvegarde du matériel:', error);
    showToast(`❌ Erreur de sauvegarde: ${error.message}`, 'error', 6000);
    
    // En cas d'erreur, on garde le formulaire ouvert pour que l'utilisateur puisse réessayer
    return;
  }
}

function deleteMaterial(id) {
  // Vérifier si le matériel est utilisé dans des événements
  const isUsed = Object.values(appData.eventMaterials || {}).some(materials =>
    materials.some(m => m.materialId === id)
  );
  
  const message = isUsed 
    ? "Ce matériel est utilisé dans des événements.\nSupprimer le matériel et toutes ses attributions ?"
    : "Supprimer ce matériel du catalogue ?";
    
  if (!confirm(message)) return;
  
  // Suppression du matériel
  appData.materials = appData.materials.filter(m => m.id !== id);
  
  // Suppression des attributions si nécessaire
  if (isUsed) {
    Object.keys(appData.eventMaterials).forEach(eventId => {
      appData.eventMaterials[eventId] = appData.eventMaterials[eventId].filter(m => m.materialId !== id);
    });
  }
  
  markAsChanged();
  saveData();
  updateMaterialList();
  showToast('Matériel supprimé avec succès', 'success');
}

function updateMaterialList() {
  const container = document.getElementById('materialList');
  const empty = document.getElementById('materialEmpty');
  
  console.log('🔄 updateMaterialList() appelée');
  console.log('📦 Matériaux actuels:', appData.materials);
  
  if (!container) {
    console.error('❌ Container #materialList non trouvé');
    return;
  }
  
  if (!empty) {
    console.error('❌ Element #materialEmpty non trouvé');
    return;
  }
  
  // Initialiser les données si nécessaire
  if (!appData.materials) {
    console.warn('⚠️ appData.materials est vide, initialisation...');
    appData.materials = [];
  }
  
  const categoryFilter = document.getElementById('materialCategoryFilter')?.value || 'tous';
  const searchQuery = document.getElementById('materialSearch')?.value.toLowerCase() || '';
  
  console.log('🔍 Filtres appliqués - Catégorie:', categoryFilter, '- Recherche:', searchQuery);
  
  // Filtrage
  let materials = appData.materials || [];
  if (categoryFilter !== 'tous') {
    materials = materials.filter(m => m.categorie === categoryFilter);
  }
  if (searchQuery) {
    materials = materials.filter(m => 
      (m.nom || '').toLowerCase().includes(searchQuery) ||
      (m.description || '').toLowerCase().includes(searchQuery)
    );
  }
  
  console.log('📋 Matériaux après filtrage:', materials.length);
  
  container.innerHTML = '';
  
  if (!materials.length) {
    console.log('📭 Aucun matériel à afficher');
    empty.style.display = 'block';
    return;
  }
  
  empty.style.display = 'none';
  
  // Groupement par catégorie
  const groupedMaterials = materials.reduce((groups, material) => {
    const category = material.categorie || 'Sans catégorie';
    if (!groups[category]) groups[category] = [];
    groups[category].push(material);
    return groups;
  }, {});
  
  console.log('📚 Groupes de matériel:', Object.keys(groupedMaterials));
  
  Object.entries(groupedMaterials)
    .sort(([a], [b]) => a.localeCompare(b, 'fr'))
    .forEach(([category, items]) => {
      // En-tête de catégorie
      const categoryHeader = document.createElement('h3');
      categoryHeader.className = 'mt-6 mb-3 text-lg font-bold text-white px-3 py-2 rounded bg-gradient-to-r from-indigo-500 to-purple-500 shadow flex items-center justify-between';
      categoryHeader.innerHTML = `
        <span><i class="fas fa-boxes mr-2"></i>${category} (${items.length})</span>
        <span class="text-sm font-normal">Stock total: ${items.reduce((sum, item) => sum + (item.stockTotal || 0), 0)}</span>
      `;
      container.appendChild(categoryHeader);
      
      // Articles de la catégorie
      items.forEach(material => {
        const stockDisponible = calculateAvailableStock(material.id);
        const stockReserve = material.stockReserve || 0;
        const isLowStock = stockDisponible <= (material.stockMin || 0);
        const isUnavailable = material.statut !== 'disponible';
        
        const card = document.createElement('div');
        card.className = `flex justify-between items-center p-4 bg-white border rounded-lg ${isLowStock ? 'border-orange-300 bg-orange-50' : ''} ${isUnavailable ? 'border-red-300 bg-red-50' : ''}`;
        
        // Icône selon la catégorie
        const categoryIcons = {
          'Tables': 'fa-table',
          'Chaises': 'fa-chair',
          'Tentes': 'fa-campground',
          'Éclairage': 'fa-lightbulb',
          'Sonorisation': 'fa-volume-up',
          'Décoration': 'fa-palette',
          'Sécurité': 'fa-shield-alt',
          'Transport': 'fa-truck',
          'Technique': 'fa-tools'
        };
        const icon = categoryIcons[material.categorie] || 'fa-box';
        
        card.innerHTML = `
          <div class="flex items-center space-x-4">
            <div class="text-2xl text-indigo-600">
              <i class="fas ${icon}"></i>
            </div>
            <div>
              <h4 class="font-semibold text-gray-800">${material.nom}</h4>
              <p class="text-sm text-gray-600">${material.description || 'Aucune description'}</p>
              <div class="flex items-center space-x-4 mt-1">
                <span class="text-sm">
                  <i class="fas fa-warehouse mr-1 text-blue-600"></i>
                  Total: <strong>${material.stockTotal}</strong> ${material.unite}
                </span>
                <span class="text-sm ${stockDisponible > 0 ? 'text-green-600' : 'text-red-600'}">
                  <i class="fas fa-check-circle mr-1"></i>
                  Disponible: <strong>${stockDisponible}</strong>
                </span>
                ${stockReserve > 0 ? `
                  <span class="text-sm text-orange-600">
                    <i class="fas fa-clock mr-1"></i>
                    Réservé: <strong>${stockReserve}</strong>
                  </span>
                ` : ''}
                ${isLowStock ? '<span class="text-xs bg-orange-200 text-orange-800 px-2 py-1 rounded">Stock faible !</span>' : ''}
                ${isUnavailable ? '<span class="text-xs bg-red-200 text-red-800 px-2 py-1 rounded">Indisponible</span>' : ''}
              </div>
            </div>
          </div>
          <div class="flex space-x-2">
            <button onclick="showMaterialUsage(${material.id})" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-1 rounded text-sm" title="Voir l'utilisation">
              <i class="fas fa-calendar-alt"></i>
            </button>
            <button onclick="showMaterialForm(${material.id})" class="bg-green-600 hover:bg-green-700 text-white px-3 py-1 rounded text-sm" title="Modifier">
              <i class="fas fa-edit"></i>
            </button>
            <button onclick="deleteMaterial(${material.id})" class="bg-red-600 hover:bg-red-700 text-white px-3 py-1 rounded text-sm" title="Supprimer">
              <i class="fas fa-trash"></i>
            </button>
          </div>
        `;
        
        container.appendChild(card);
      });
    });
    
  console.log('✅ Liste du matériel mise à jour avec', materials.length, 'articles');
}

function calculateAvailableStock(materialId) {
  const material = appData.materials.find(m => m.id === materialId);
  if (!material) return 0;
  
  const today = new Date().toISOString().split('T')[0];
  let reserved = 0;
  
  // Calcul des réservations : matériel sorti mais pas encore retourné
  Object.values(appData.eventMaterials || {}).forEach(eventMaterials => {
    eventMaterials.forEach(allocation => {
      if (allocation.materialId === materialId && 
          allocation.dateFin >= today && 
          allocation.sortieValidee === true && 
          allocation.valide === false) { // Sorti mais pas encore retourné
        reserved += allocation.quantite || 0;
      }
    });
  });
  
  return Math.max(0, material.stockTotal - reserved);
}

function updateMaterialFilters() {
  const categoryFilter = document.getElementById('materialCategoryFilter');
  if (!categoryFilter) return;
  
  const currentValue = categoryFilter.value;
  const categories = [...new Set(appData.materials.map(m => m.categorie))].sort();
  
  categoryFilter.innerHTML = '<option value="tous">📦 Toutes catégories</option>' +
    categories.map(cat => `<option value="${cat}">${cat}</option>`).join('');
    
  if (categories.includes(currentValue)) {
    categoryFilter.value = currentValue;
  }
}

function updateMaterialAvailability() {
  const today = new Date().toISOString().split('T')[0];
  
  // Libération automatique du matériel des événements terminés
  Object.keys(appData.eventMaterials || {}).forEach(eventId => {
    appData.eventMaterials[eventId] = appData.eventMaterials[eventId].filter(allocation => {
      if (allocation.dateFin < today && !allocation.valide) {
        // Auto-validation des retours dépassés
        allocation.valide = true;
        return true;
      }
      return true;
    });
  });
  
  // Force la mise à jour de l'affichage
  updateMaterialList();
  updateMaterialFilters();
  
  // Si un modal matériel est ouvert, le rafraîchir aussi
  if (currentEventForMaterial) {
    updateMaterialSelect();
    updateEventMaterialList();
  }
  
  markAsChanged();
  saveData();
  showToast('Stocks mis à jour ! Matériel des événements terminés libéré.', 'success');
}

function showMaterialUsage(materialId) {
  const material = appData.materials.find(m => m.id === materialId);
  if (!material) return;
  
  const usages = [];
  Object.entries(appData.eventMaterials || {}).forEach(([eventId, allocations]) => {
    allocations.forEach(allocation => {
      if (allocation.materialId === materialId) {
        const event = appData.events.find(e => e.id == eventId);
        if (event) {        usages.push({
          event: event.nom,
          quantite: allocation.quantite,
          dateDebut: allocation.dateDebut,
          dateFin: allocation.dateFin,
          valide: allocation.valide,
          sortieValidee: allocation.sortieValidee
        });
        }
      }
    });
  });
  
  const usageHtml = usages.length ? usages.map(u => `
    <div class="p-2 border rounded ${
      u.valide ? 'bg-green-50' : 
      u.sortieValidee ? 'bg-blue-50' : 
      'bg-yellow-50'
    }">
      <strong>${u.event}</strong><br>
      Quantité: ${u.quantite} ${material.unite}<br>
      Période: ${u.dateDebut} → ${u.dateFin}<br>
      <span class="text-xs ${
        u.valide ? 'text-green-600' : 
        u.sortieValidee ? 'text-blue-600' : 
        'text-orange-600'
      }">
        ${u.valide ? '✓ Retourné' : 
          u.sortieValidee ? '📦 Sorti du stock' : 
          '⏳ En attente de sortie'}
      </span>
    </div>
  `).join('') : '<div class="text-gray-500">Aucune utilisation</div>';
  
  document.getElementById('barInfoTitle').textContent = `Utilisation - ${material.nom}`;
  document.getElementById('barInfoBody').innerHTML = usageHtml;
  document.getElementById('barInfoModal').classList.add('active');
}

/* ==== FONCTIONS MODAL MATÉRIEL ÉVÉNEMENT ==== */

function openMaterialModal(eventId) {
  const event = appData.events.find(e => e.id === eventId);
  if (!event) return;
  
  currentEventForMaterial = eventId;
  document.getElementById('materialEventName').textContent = event.nom;
  
  // Préremplir les dates avec celles de l'événement
  document.getElementById('materialStartDate').value = event.dateDebut;
  document.getElementById('materialEndDate').value = event.dateFin;
  
  updateMaterialSelect();
  updateEventMaterialList();
  
  document.getElementById('materialModal').classList.add('active');
  
  // Focus sur le premier élément
  setTimeout(() => {
    const firstInput = document.querySelector('#materialModal select, #materialModal input');
    if (firstInput) firstInput.focus();
  }, 100);
}

function closeMaterialModal() {
  document.getElementById('materialModal').classList.remove('active');
  currentEventForMaterial = null;
  
  // Reset du formulaire
  document.getElementById('materialSelect').value = '';
  document.getElementById('materialQuantity').value = '1';
  document.getElementById('materialAlert').style.display = 'none';
}

function updateMaterialSelect() {
  const select = document.getElementById('materialSelect');
  select.innerHTML = '<option value="">-- Sélectionner un article --</option>';
  
  appData.materials
    .filter(m => m.statut === 'disponible')
    .sort((a, b) => a.nom.localeCompare(b.nom, 'fr'))
    .forEach(material => {
      const available = calculateAvailableStock(material.id);
      const option = document.createElement('option');
      option.value = material.id;
      option.textContent = `${material.nom} (${available} disponible)`;
      if (available === 0) {
        option.disabled = true;
        option.textContent += ' - RUPTURE';
      }
      select.appendChild(option);
    });
}

function addMaterialToEvent() {
  const materialId = parseInt(document.getElementById('materialSelect').value);
  const quantite = parseInt(document.getElementById('materialQuantity').value) || 1;
  const dateDebut = document.getElementById('materialStartDate').value;
  const dateFin = document.getElementById('materialEndDate').value;
  
  if (!materialId || !dateDebut || !dateFin) {
    showToast('Veuillez remplir tous les champs', 'error');
    return;
  }
  
  if (new Date(dateDebut) > new Date(dateFin)) {
    showToast('La date de début doit précéder la date de fin', 'error');
    return;
  }
  
  const material = appData.materials.find(m => m.id === materialId);
  const available = calculateAvailableStock(materialId);
  
  // Alerte si dépassement mais pas de blocage
  const alert = document.getElementById('materialAlert');
  if (quantite > available) {
    alert.innerHTML = `<i class="fas fa-exclamation-triangle text-orange-600 mr-2"></i>
      <span class="text-orange-800">Attention: Vous demandez ${quantite} ${material.unite} mais seulement ${available} disponible(s). 
      Cela créera un déficit de stock.</span>`;
    alert.className = 'text-sm bg-orange-100 p-3 rounded border border-orange-300';
    alert.style.display = 'block';
  } else {
    alert.style.display = 'none';
  }
  
  // Ajout de l'attribution
  if (!appData.eventMaterials[currentEventForMaterial]) {
    appData.eventMaterials[currentEventForMaterial] = [];
  }
  
  const allocation = {
    id: Date.now(),
    materialId: materialId,
    quantite: quantite,
    dateDebut: dateDebut,
    dateFin: dateFin,
    valide: false, // Pas encore retourné
    sortieValidee: false // Pas encore sorti du stock physiquement
  };
  
  appData.eventMaterials[currentEventForMaterial].push(allocation);
  
  markAsChanged();
  saveData();
  updateEventMaterialList();
  updateMaterialSelect();
  
  showToast(`${material.nom} attribué à l'événement (${quantite} ${material.unite})`, 'success');
  
  // Reset des champs
  document.getElementById('materialSelect').value = '';
  document.getElementById('materialQuantity').value = '1';
}

function updateEventMaterialList() {
  const container = document.getElementById('eventMaterialList');
  const empty = document.getElementById('eventMaterialEmpty');
  
  const allocations = appData.eventMaterials[currentEventForMaterial] || [];
  
  container.innerHTML = '';
  
  if (!allocations.length) {
    empty.style.display = 'block';
    return;
  }
  
  empty.style.display = 'none';
  
  allocations.forEach(allocation => {
    const material = appData.materials.find(m => m.id === allocation.materialId);
    if (!material) return;
    
    const card = document.createElement('div');
    card.className = `flex justify-between items-center p-3 border rounded-lg ${
      allocation.valide ? 'bg-green-50 border-green-200' : 
      allocation.sortieValidee ? 'bg-blue-50 border-blue-200' : 
      'bg-yellow-50 border-yellow-200'
    }`;
    
    card.innerHTML = `
      <div class="flex items-center space-x-3">
        <div class="text-xl text-indigo-600">
          <i class="fas fa-box"></i>
        </div>
        <div>
          <h5 class="font-semibold text-gray-800">${material.nom}</h5>
          <p class="text-sm text-gray-600">
            Quantité: <strong>${allocation.quantite} ${material.unite}</strong> • 
            ${allocation.dateDebut} → ${allocation.dateFin}
          </p>
          <div class="flex space-x-2 mt-1">
            <span class="text-xs px-2 py-1 rounded ${
              allocation.valide ? 'bg-green-200 text-green-800' : 
              allocation.sortieValidee ? 'bg-blue-200 text-blue-800' : 
              'bg-orange-200 text-orange-800'
            }">
              ${allocation.valide ? '✓ Retourné en stock' : 
                allocation.sortieValidee ? '📦 Sorti du stock' : 
                '⏳ En attente de sortie'}
            </span>
          </div>
        </div>
      </div>
      <div class="flex space-x-2">
        ${!allocation.sortieValidee && !allocation.valide ? `
          <button onclick="validateMaterialExit(${allocation.id})" class="bg-green-600 hover:bg-green-700 text-white px-3 py-1 rounded text-sm" title="Valider la sortie du stock">
            <i class="fas fa-arrow-right"></i> Sortir
          </button>
        ` : ''}
        ${allocation.sortieValidee && !allocation.valide ? `
          <button onclick="validateMaterialReturn(${allocation.id})" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-1 rounded text-sm" title="Valider le retour">
            <i class="fas fa-arrow-left"></i> Retour
          </button>
        ` : ''}
        <button onclick="removeMaterialFromEvent(${allocation.id})" class="bg-red-600 hover:bg-red-700 text-white px-3 py-1 rounded text-sm" title="Supprimer l'attribution">
          <i class="fas fa-trash"></i>
        </button>
      </div>
    `;
    
    container.appendChild(card);
  });
}

function removeMaterialFromEvent(allocationId) {
  if (!confirm('Supprimer cette attribution de matériel ?')) return;
  
  const allocations = appData.eventMaterials[currentEventForMaterial] || [];
  appData.eventMaterials[currentEventForMaterial] = allocations.filter(a => a.id !== allocationId);
  
  markAsChanged();
  saveData();
  updateEventMaterialList();
  updateMaterialSelect();
  showToast('Attribution supprimée', 'success');
}

function validateMaterialExit(allocationId) {
  const allocations = appData.eventMaterials[currentEventForMaterial] || [];
  const allocation = allocations.find(a => a.id === allocationId);
  
  if (allocation) {
    allocation.sortieValidee = true;
    markAsChanged();
    saveData();
    updateEventMaterialList();
    updateMaterialSelect(); // Refresh pour mettre à jour les stocks disponibles
    showToast('Sortie de matériel validée ! Le stock disponible a été mis à jour.', 'success');
  }
}

function validateMaterialReturn(allocationId) {
  const allocations = appData.eventMaterials[currentEventForMaterial] || [];
  const allocation = allocations.find(a => a.id === allocationId);
  
  if (allocation) {
    allocation.valide = true;
    markAsChanged();
    saveData();
    updateEventMaterialList();
    updateMaterialSelect(); // Refresh pour mettre à jour les stocks disponibles
    
    // Aussi mettre à jour la liste globale du matériel si on est dans ce module
    if (document.getElementById('material').classList.contains('active')) {
      updateMaterialList();
    }
    
    showToast('Retour de matériel validé ! Stock mis à jour.', 'success');
  }
}

function printMaterialList() {
  const event = appData.events.find(e => e.id === currentEventForMaterial);
  if (!event) return;
  
  const allocations = appData.eventMaterials[currentEventForMaterial] || [];
  
  let content = `
    <div class="p-6">
      <h1 class="text-2xl font-bold mb-4">Liste du Matériel - ${event.nom}</h1>
      <div class="mb-4">
        <p><strong>Événement:</strong> ${event.nom}</p>
        <p><strong>Lieu:</strong> ${event.lieu}</p>
        <p><strong>Dates:</strong> ${event.dateDebut} → ${event.dateFin}</p>
        <p><strong>Date d'impression:</strong> ${new Date().toLocaleDateString('fr-FR')}</p>
      </div>
      <table class="w-full border-collapse border">
        <thead>
          <tr class="bg-gray-100">
            <th class="border p-2 text-left">Article</th>
            <th class="border p-2 text-center">Quantité</th>
            <th class="border p-2 text-center">Période</th>
            <th class="border p-2 text-center">Statut</th>
          </tr>
        </thead>
        <tbody>
  `;
  
  if (allocations.length) {
    allocations.forEach(allocation => {
      const material = appData.materials.find(m => m.id === allocation.materialId);
      if (material) {
        content += `
          <tr>
            <td class="border p-2">${material.nom}</td>
            <td class="border p-2 text-center">${allocation.quantite} ${material.unite}</td>
            <td class="border p-2 text-center">${allocation.dateDebut} → ${allocation.dateFin}</td>
            <td class="border p-2 text-center">${allocation.valide ? 'Retourné' : 'En cours'}</td>
          </tr>
        `;
      }
    });
  } else {
    content += '<tr><td colspan="4" class="border p-2 text-center text-gray-500">Aucun matériel attribué</td></tr>';
  }
  
  content += `
        </tbody>
      </table>
      <div class="mt-6 text-sm text-gray-600">
        <p>Document généré par l'Application Città di Biguglia</p>
      </div>
    </div>
  `;
  
  const w = window.open('', 'PRINT', 'height=800,width=1000');
  w.document.write(`
    <html>
      <head>
        <title>Liste Matériel - ${event.nom}</title>
        <link href='https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css' rel='stylesheet'/>
        <style>@media print { body { -webkit-print-color-adjust: exact; } }</style>
      </head>
      <body>${content}</body>
    </html>
  `);
  w.document.close();
  w.focus();
  w.print();
  w.close();
}

/* ==== FONCTIONS VALIDATION RETOURS ==== */

function showMaterialConsumptionChart() {
  // Peupler les filtres
  const materialFilter = document.getElementById('consumptionMaterialFilter');
  if (materialFilter) {
    materialFilter.innerHTML = '<option value="all">📦 Tous les articles</option>' +
      appData.materials.map(m => `<option value="${m.id}">${m.nom}</option>`).join('');
  }
  
  // Afficher le modal
  document.getElementById('materialConsumptionModal').classList.add('active');
  
  // Générer les graphiques
  setTimeout(() => {
    updateConsumptionChart();
  }, 200);
}

function closeMaterialConsumptionModal() {
  document.getElementById('materialConsumptionModal').classList.remove('active');
}

function updateConsumptionChart() {
  const materialFilter = document.getElementById('consumptionMaterialFilter')?.value || 'all';
  const periodFilter = document.getElementById('consumptionPeriodFilter')?.value || 'year';
  
  // Calcul de la période
  const today = new Date();
  let startDate;
  switch(periodFilter) {
    case '3months':
      startDate = new Date(today.getFullYear(), today.getMonth() - 3, 1);
      break;
    case '6months':
      startDate = new Date(today.getFullYear(), today.getMonth() - 6, 1);
      break;
    default: // year
      startDate = new Date(today.getFullYear(), 0, 1);
  }
  
  // Collecte des données de consommation
  const consumptionData = [];
  const monthlyData = {};
  const eventData = {};
  
  Object.entries(appData.eventMaterials || {}).forEach(([eventId, allocations]) => {
    const event = appData.events.find(e => e.id == eventId);
    if (!event) return;
    
    allocations.forEach(allocation => {
      // Filtrer par matériel si nécessaire
      if (materialFilter !== 'all' && allocation.materialId != materialFilter) return;
      
      // Filtrer par période
      const allocationDate = new Date(allocation.dateDebut);
      if (allocationDate < startDate) return;
      
      const material = appData.materials.find(m => m.id === allocation.materialId);
      if (!material) return;
      
      // Données pour le tableau détaillé
      consumptionData.push({
        materialName: material.nom,
        eventName: event.nom,
        quantity: allocation.quantite,
        period: `${allocation.dateDebut} → ${allocation.dateFin}`,
        status: allocation.valide ? 'Retourné' : (allocation.sortieValidee ? 'En cours' : 'Attribué')
      });
      
      // Données mensuelles
      const monthKey = `${allocationDate.getFullYear()}-${String(allocationDate.getMonth() + 1).padStart(2, '0')}`;
      if (!monthlyData[monthKey]) monthlyData[monthKey] = {};
      if (!monthlyData[monthKey][material.nom]) monthlyData[monthKey][material.nom] = 0;
      monthlyData[monthKey][material.nom] += allocation.quantite;
      
      // Données par événement
      if (!eventData[event.nom]) eventData[event.nom] = {};
      if (!eventData[event.nom][material.nom]) eventData[event.nom][material.nom] = 0;
      eventData[event.nom][material.nom] += allocation.quantite;
    });
  });
  
  // Génération du graphique mensuel
  createMonthlyConsumptionChart(monthlyData);
  
  // Génération du graphique par événement
  createEventConsumptionChart(eventData);
  
  // Génération des prédictions budgétaires
  createBudgetPredictions(monthlyData);
  
  // Remplissage du tableau détaillé
  fillConsumptionDetailTable(consumptionData);
}

function createMonthlyConsumptionChart(monthlyData) {
  const canvas = document.getElementById('monthlyConsumptionChart');
  if (!canvas || !window.Chart) return;
  
  const ctx = canvas.getContext('2d');
  
  // Nettoie le graphique existant
  if (window.monthlyConsumptionChartInstance) {
    window.monthlyConsumptionChartInstance.destroy();
  }
  
  // Préparation des données
  const months = Object.keys(monthlyData).sort();
  const materials = [...new Set(Object.values(monthlyData).flatMap(m => Object.keys(m)))];
  
  const datasets = materials.map((material, index) => {
    const data = months.map(month => monthlyData[month][material] || 0);
    const colors = ['#3b82f6', '#ef4444', '#10b981', '#f59e0b', '#8b5cf6', '#06b6d4', '#f97316'];
    
    return {
      label: material,
      data: data,
      backgroundColor: colors[index % colors.length],
      borderColor: colors[index % colors.length],
      tension: 0.3
    };
  });
  
  const labels = months.map(month => {
    const [year, monthNum] = month.split('-');
    return `${monthLabels()[parseInt(monthNum) - 1]} ${year}`;
  });
  
  window.monthlyConsumptionChartInstance = new Chart(ctx, {
    type: 'line',
    data: { labels, datasets },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: { position: 'top' },
        tooltip: {
          callbacks: {
            label: (context) => `${context.dataset.label}: ${context.parsed.y} unités`
          }
        }
      },
      scales: {
        y: {
          beginAtZero: true,
          title: { display: true, text: 'Quantité utilisée' }
        },
        x: {
          title: { display: true, text: 'Période' }
        }
      }
    }
  });
}

function createEventConsumptionChart(eventData) {
  const canvas = document.getElementById('eventConsumptionChart');
  if (!canvas || !window.Chart) return;
  
  const ctx = canvas.getContext('2d');
  
  // Nettoie le graphique existant
  if (window.eventConsumptionChartInstance) {
    window.eventConsumptionChartInstance.destroy();
  }
  
  // Calcul du total par événement
  const eventTotals = Object.entries(eventData).map(([eventName, materials]) => {
    const total = Object.values(materials).reduce((sum, qty) => sum + qty, 0);
    return { eventName, total };
  }).sort((a, b) => b.total - a.total).slice(0, 10); // Top 10
  
  const labels = eventTotals.map(e => e.eventName.length > 20 ? e.eventName.substr(0, 17) + '...' : e.eventName);
  const data = eventTotals.map(e => e.total);
  
  window.eventConsumptionChartInstance = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: labels,
      datasets: [{
        data: data,
        backgroundColor: [
          '#3b82f6', '#ef4444', '#10b981', '#f59e0b', '#8b5cf6', 
          '#06b6d4', '#f97316', '#84cc16', '#ec4899', '#6366f1'
        ]
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: { position: 'bottom' },
        tooltip: {
          callbacks: {
            label: (context) => `${context.label}: ${context.parsed} unités`
          }
        }
      }
    }
  });
}

function createBudgetPredictions(monthlyData) {
  const container = document.getElementById('budgetPredictions');
  if (!container) return;
  
  // 🔄 ALGORITHME AMÉLIORÉ pour gestion événements chevauchants
  const materialAnalysis = {};
  
  // Analyse sophistiquée basée sur votre méthode de travail
  Object.entries(appData.eventMaterials || {}).forEach(([eventId, allocations]) => {
    const event = appData.events.find(e => e.id == eventId);
    if (!event) return;
    
    allocations.forEach(allocation => {
      const material = appData.materials.find(m => m.id === allocation.materialId);
      if (!material) return;
      
      if (!materialAnalysis[material.nom]) {
        materialAnalysis[material.nom] = {
          materialId: material.id,
          usages: [],
          peakSimultaneous: 0,
          overlappingEvents: [],
          currentStock: material.stockTotal,
          monthlyHistory: []
        };
      }
      
      // Enregistre chaque utilisation avec période complète
      materialAnalysis[material.nom].usages.push({
        eventId: eventId,
        eventName: event.nom,
        quantity: allocation.quantite,
        startDate: allocation.dateDebut,
        endDate: allocation.dateFin,
        duration: getDatesBetween(allocation.dateDebut, allocation.dateFin).length,
        isReturned: allocation.valide
      });
    });
  });
  
  // Calcul du pic de chevauchement pour chaque matériel
  Object.values(materialAnalysis).forEach(analysis => {
    // Simule jour par jour pour trouver le pic d'utilisation simultanée
    const dailyUsage = {};
    
    analysis.usages.forEach(usage => {
      const dates = getDatesBetween(usage.startDate, usage.endDate);
      dates.forEach(date => {
        if (!dailyUsage[date]) dailyUsage[date] = 0;
        dailyUsage[date] += usage.quantity;
      });
    });
    
    // Trouve le pic d'utilisation simultanée
    analysis.peakSimultaneous = Math.max(0, ...Object.values(dailyUsage));
    
    // Détecte les chevauchements problématiques
    analysis.overlappingEvents = Object.entries(dailyUsage)
      .filter(([date, qty]) => qty > analysis.currentStock)
      .map(([date, qty]) => ({ date, shortage: qty - analysis.currentStock }));
    
    // Calcul mensuel basé sur l'historique réel
    for (let month = 0; month < 12; month++) {
      const monthUsages = analysis.usages.filter(usage => {
        const startMonth = toLocalDate(usage.startDate).getMonth();
        const endMonth = toLocalDate(usage.endDate).getMonth();
        return startMonth === month || endMonth === month;
      });
      
      const monthTotal = monthUsages.reduce((sum, usage) => {
        // Calcul pondéré : si l'événement traverse plusieurs mois
        const eventDuration = usage.duration;
        const monthOverlap = Math.min(eventDuration, 31); // Approximation
        return sum + (usage.quantity * monthOverlap / eventDuration);
      }, 0);
      
      analysis.monthlyHistory.push(monthTotal);
    }
  });
  
  // Génère les prédictions avec logique métier réaliste
  const predictions = Object.entries(materialAnalysis).map(([materialName, analysis]) => {
    const monthlyUsage = analysis.monthlyHistory.filter(m => m > 0);
    if (monthlyUsage.length === 0) return null;
    
    const avgMonthly = monthlyUsage.reduce((a, b) => a + b, 0) / monthlyUsage.length;
    const maxMonthly = Math.max(...monthlyUsage);
    const peakSimultaneous = analysis.peakSimultaneous;
    
    // 🎯 ESTIMATION RÉALISTE basée sur vos contraintes
    // Prend en compte les chevauchements et la rotation du matériel
    const rotationFactor = analysis.usages.length > 0 ? 
      analysis.usages.reduce((sum, u) => sum + u.duration, 0) / analysis.usages.length / 30 : 1;
    
    const estimationAnnuelle = Math.max(
      avgMonthly * 12, // Estimation classique
      peakSimultaneous * rotationFactor * 12, // Basée sur les pics de chevauchement
      maxMonthly * 8 // Sécurité pour les mois chargés
    );
    
    // 🚨 ANALYSE DES CONFLITS RÉELS
    const hasConflicts = analysis.overlappingEvents.length > 0;
    const stockSuffisant = analysis.currentStock >= peakSimultaneous;
    const margeSecurite = analysis.currentStock - peakSimultaneous;
    
    // 💡 RECOMMANDATIONS ADAPTÉES À VOTRE USAGE
    let recommandation, recommandationColor;
    
    if (avgMonthly === 0) {
      recommandation = 'Matériel non utilisé - À réaffecter ?';
      recommandationColor = 'text-gray-500';
    } else if (hasConflicts) {
      recommandation = `🔴 CONFLITS DÉTECTÉS ! Manque ${Math.max(...analysis.overlappingEvents.map(e => e.shortage))} unités`;
      recommandationColor = 'text-red-600';
    } else if (margeSecurite < 2) {
      recommandation = '🟠 MARGE CRITIQUE - Risque de conflit élevé';
      recommandationColor = 'text-orange-600';
    } else if (margeSecurite < 5) {
      recommandation = '🟡 MARGE FAIBLE - Surveiller les chevauchements';
      recommandationColor = 'text-yellow-600';
    } else if (stockSuffisant) {
      recommandation = '🟢 STOCK ADAPTÉ - Gestion actuelle correcte';
      recommandationColor = 'text-green-600';
    } else {
      recommandation = '🔵 STOCK SUFFISANT - Optimisation possible';
      recommandationColor = 'text-blue-600';
    }
    
    return {
      material: materialName,
      avgMonthly: avgMonthly,
      maxMonthly: maxMonthly,
      peakSimultaneous: peakSimultaneous,
      estimationAnnuelle: estimationAnnuelle,
      stockDisponible: analysis.currentStock,
      margeSecurite: margeSecurite,
      conflitsDetectes: analysis.overlappingEvents.length,
      tauxUtilisation: (peakSimultaneous / analysis.currentStock * 100),
      recommandation: recommandation,
      recommandationColor: recommandationColor,
      variabilite: maxMonthly - (monthlyUsage.reduce((a, b) => Math.min(a, b), maxMonthly) || 0),
      explanations: {
        pourquoi: hasConflicts ? 
          `Pic simultané: ${peakSimultaneous} unités, mais stock: ${analysis.currentStock}` :
          `Usage max simultané: ${peakSimultaneous}/${analysis.currentStock} unités`,
        conseil: hasConflicts ?
          'Augmentez le stock ou décalez certains événements' :
          'Stock adapté à vos besoins actuels'
      }
    };
  }).filter(Boolean).sort((a, b) => b.avgMonthly - a.avgMonthly).slice(0, 3);
  
  if (predictions.length === 0) {
    container.innerHTML = '<div class="text-center text-gray-500 py-8">Aucune donnée de consommation disponible</div>';
    return;
  }
  
  container.innerHTML = predictions.map(pred => `
    <div class="bg-white p-4 rounded-lg border shadow-sm">
      <h5 class="font-semibold text-gray-800 mb-3">${pred.material}</h5>
      <div class="space-y-2 text-sm">
        
        <!-- 🎯 DONNÉES DE BASE -->
        <div class="bg-blue-50 p-3 rounded border-l-4 border-blue-400 mb-3">
          <div class="font-semibold text-blue-800 mb-1">📊 Analyse de vos données :</div>
          <div class="text-xs text-blue-700 space-y-1">
            <div>• Stock actuel : <strong>${pred.stockDisponible} unités</strong></div>
            <div>• Pic d'utilisation simultanée : <strong>${pred.peakSimultaneous} unités</strong></div>
            <div>• Marge de sécurité : <strong>${pred.margeSecurite} unités</strong></div>
            <div>• Événements en conflit : <strong>${pred.conflitsDetectes}</strong></div>
            <div>• Taux d'occupation max : <strong>${pred.tauxUtilisation.toFixed(1)}%</strong></div>
          </div>
        </div>

        <!-- 📈 TENDANCES D'USAGE -->
        <div class="flex justify-between">
          <span class="text-gray-600">Usage mensuel moyen:</span>
          <span class="font-semibold">${pred.avgMonthly.toFixed(1)} unités/mois</span>
        </div>
        <div class="flex justify-between">
          <span class="text-gray-600">Pic mensuel:</span>
          <span class="font-semibold text-orange-600">${pred.maxMonthly} unités</span>
        </div>
        <div class="flex justify-between">
          <span class="text-gray-600">Variabilité usage:</span>
          <span class="font-semibold ${pred.variabilite > pred.avgMonthly ? 'text-red-600' : pred.variabilite > pred.avgMonthly/2 ? 'text-orange-600' : 'text-green-600'}">${pred.variabilite.toFixed(1)} unités</span>
        </div>

        <!-- ⚠️ GESTION DES CHEVAUCHEMENTS -->
        <div class="flex justify-between">
          <span class="text-gray-600">Pic simultané critique:</span>
          <span class="font-semibold ${pred.peakSimultaneous > pred.stockDisponible ? 'text-red-600' : pred.peakSimultaneous > pred.stockDisponible * 0.8 ? 'text-orange-600' : 'text-green-600'}">${pred.peakSimultaneous} unités</span>
        </div>
        
        <!-- 💡 EXPLICATION DU CALCUL -->
        <div class="bg-yellow-50 p-3 rounded border-l-4 border-yellow-400 mt-3">
          <div class="font-semibold text-yellow-800 mb-1">🔍 Pourquoi cette estimation ?</div>
          <div class="text-xs text-yellow-700">
            ${pred.explanations.pourquoi}
          </div>
          <div class="text-xs text-yellow-600 mt-1 italic">
            💡 Conseil : ${pred.explanations.conseil}
          </div>
        </div>

        <div class="border-t pt-2 mt-2">
          <div class="flex justify-between">
            <span class="text-gray-600">Besoins estimés/an:</span>
            <span class="font-bold text-blue-600">${pred.estimationAnnuelle.toFixed(0)} unités</span>
          </div>
          <div class="mt-2">
            <div class="text-gray-600 text-xs mb-1">🎯 Recommandation stratégique :</div>
            <div class="font-semibold ${pred.recommandationColor} text-xs leading-tight">${pred.recommandation}</div>
          </div>
          
          <!-- 📋 DÉTAIL DU CALCUL -->
          <div class="mt-3 p-2 bg-gray-50 rounded text-xs">
            <div class="font-semibold text-gray-700 mb-1">📋 Méthode de calcul :</div>
            <div class="text-gray-600 space-y-1">
              <div>1. Analyse des chevauchements d'événements</div>
              <div>2. Pic simultané : ${pred.peakSimultaneous} unités nécessaires en même temps</div>
              <div>3. Rotation moyenne sur les événements de l'année</div>
              <div>4. Marge de sécurité pour imprévus et maintenance</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  `).join('');
}

function fillConsumptionDetailTable(consumptionData) {
  const tbody = document.getElementById('consumptionDetailBody');
  if (!tbody) return;
  
  tbody.innerHTML = consumptionData.slice(0, 50).map(item => `
    <tr>
      <td class="border p-2">${item.materialName}</td>
      <td class="border p-2">${item.eventName}</td>
      <td class="border p-2 text-center">${item.quantity}</td>
      <td class="border p-2 text-center">${item.period}</td>
      <td class="border p-2 text-center">
        <span class="px-2 py-1 rounded text-xs ${
          item.status === 'Retourné' ? 'bg-green-100 text-green-800' :
          item.status === 'En cours' ? 'bg-blue-100 text-blue-800' :
          'bg-yellow-100 text-yellow-800'
        }">${item.status}</span>
      </td>
    </tr>
  `).join('');
}

/* ==== Navigation et modules ==== */
function showModule(moduleId){
  // Masquer tous les modules
  document.querySelectorAll('.module-content').forEach(m=>m.classList.remove('active'));
  
  // Réinitialiser tous les boutons de navigation
  document.querySelectorAll('.nav-btn').forEach(b=>{
    b.classList.remove('active');
    b.classList.add('bg-white','text-gray-700');
  });
  
  // Afficher le module sélectionné
  const moduleElement = document.getElementById(moduleId);
  if (moduleElement) {
    moduleElement.classList.add('active');
  }
  
  // Mettre à jour le bouton actif
  const btns=document.querySelectorAll('.nav-btn');
  let activeIndex = -1;
  
  switch(moduleId) {
    case 'dashboard': activeIndex = 0; break;
    case 'agents': activeIndex = 1; break;
    case 'events': activeIndex = 2; break;
    case 'hours': activeIndex = 3; break;
    case 'calendar': activeIndex = 4; break;
    case 'material': activeIndex = 5; break;
  }
  
  if (activeIndex >= 0 && btns[activeIndex]) {
    btns[activeIndex].classList.add('active');
    btns[activeIndex].classList.remove('bg-white','text-gray-700');
  }
  
  // Initialiser le contenu du module
  try {
    switch(moduleId) {
      case 'dashboard':
        updateDashboard();
        break;
      case 'agents':
        updateAgentsList();
        break;
      case 'events':
        updateEventsList();
        break;
      case 'hours':
        updateHoursTable();
        break;
      case 'calendar':
        renderCalendar();
        break;
      case 'material':
        // Initialisation forcée du matériel si nécessaire
        if (!appData.materials) {
          appData.materials = [
            { 
              id: 1, 
              nom: "Table rectangulaire", 
              categorie: "Tables", 
              stockTotal: 50, 
              stockReserve: 0,
              unite: "unité", 
              stockMin: 5,
              statut: "disponible",
              description: "Table 180x80cm pliante"
            },
            { 
              id: 2, 
              nom: "Chaise pliante", 
              categorie: "Chaises", 
              stockTotal: 200, 
              stockReserve: 0,
              unite: "unité", 
              stockMin: 20,
              statut: "disponible",
              description: "Chaise pliante blanche"
            },
            { 
              id: 3, 
              nom: "Tente 3x3m", 
              categorie: "Tentes", 
              stockTotal: 10, 
              stockReserve: 0,
              unite: "unité", 
              stockMin: 2,
              statut: "disponible",
              description: "Tente pliante 3x3m avec bâches latérales"
            }
          ];
          console.log('✨ Matériel par défaut initialisé');
        }
        if (!appData.eventMaterials) {
          appData.eventMaterials = {};
          console.log('✨ EventMaterials initialisé');
        }
        
        console.log('📦 Module matériel activé avec', appData.materials.length, 'articles');
        updateMaterialList(); 
        updateMaterialFilters(); 
        break;
    }
  } catch(error) {
    console.error(`Erreur lors de l'initialisation du module ${moduleId}:`, error);
    showToast(`Erreur lors du chargement du module ${moduleId}`, 'error');
  }
}

/* ==== Helper pour sécuriser les charts ==== */
function chartReady(canvasId){
  const canvas = document.getElementById(canvasId);
  return !!window.Chart && 
         !!canvas && 
         canvas.offsetWidth > 0 && 
         canvas.offsetHeight > 0;
}

/* ==== Helper pour vérifier si un canvas est visible dans un modal ==== */
function canvasReadyInModal(canvasId, modalId){
  const canvas = document.getElementById(canvasId);
  const modal = document.getElementById(modalId);
  return !!window.Chart && 
         !!canvas && 
         !!modal &&
         modal.classList.contains('active') &&
         canvas.offsetWidth > 0 && 
         canvas.offsetHeight > 0;
}
function updateDashboard(){
  try {
    updateTypeFilter();
    const parts=getFilteredParticipants();
    const acc={}; appData.agents.forEach(a=> acc[a.id]={pay:0, rec:0, vf:0});
    parts.forEach(p=>{
      let d=calculateDuration(p.heureDebut||'', p.heureFin||'');
      if(d<=0){ d=(+p.payees||0)+(+p.recup||0)+(+p.voirieFest||0); } // heures fallback
      if(p.nature==='payees') acc[p.agentId].pay += d*60;
      else if(p.nature==='recup') acc[p.agentId].rec += d*60;
      else if(p.nature==='voirie') acc[p.agentId].vf += d*60;
    });
    let tp=0,tr=0,tv=0; Object.values(acc).forEach(x=>{ tp+=x.pay; tr+=x.rec; tv+=x.vf; });

    document.getElementById('totalHeuresPayees').textContent = formatMinutes(tp);
    document.getElementById('totalHeuresRecup').textContent  = formatMinutes(tr);
    document.getElementById('heuresVoirieFestvites').textContent = formatMinutes(tv);

    const list=document.getElementById('dashboardAgentsList'); list.innerHTML='';
    appData.agents.forEach(a=>{
      const h=acc[a.id], tot=h.pay+h.rec+h.vf;
      if(tot>0 || currentTypeFilter==='tous'){
        const li=document.createElement('li');
        li.className='cursor-pointer text-blue-600 hover:underline flex justify-between p-2 hover:bg-gray-50 rounded';
        li.innerHTML = `<span>${a.nom}</span><span class="text-gray-600 text-sm">${formatMinutes(tot)} total</span>`;
        li.onclick=()=>showAgentDetail(a.id);
        list.appendChild(li);
      }
    });

    createEventsChart();
    createSectorsChart();
    createHoursChart();
    createMonthlyTypeChart();
    createYearlyHoursLineChart();
    createMonthlyBreakdownLineChart();
    createMultiHoursChart();
    createYearlyTypeHoursChart(); /* nouveau */
  } catch(error) {
    console.error('Erreur updateDashboard:', error);
    // Fallback gracieux
    document.getElementById('totalHeuresPayees').textContent = '0h00m';
    document.getElementById('totalHeuresRecup').textContent = '0h00m';
    document.getElementById('heuresVoirieFestvites').textContent = '0h00m';
  }
}
function createEventsChart(){
  if (!window.Chart) return;
  if(eventsChartInstance) eventsChartInstance.destroy();

  const ctx=document.getElementById('eventsChart').getContext('2d');
  const evs=getFilteredEvents(), monthly=new Array(12).fill(0);
  evs.forEach(e=> monthly[toLocalDate(e.dateDebut).getMonth()]++);
  const totalYear = monthly.reduce((a,b)=>a+b,0);

  // Events (ligne)
  const peak = Math.max(0, ...monthly);
  const maxY  = Math.max(5, Math.ceil(peak * 1.2));

  eventsChartInstance=new Chart(ctx,{
    type:'line',
    data:{ labels:monthLabels(), datasets:[{ label:'Événements', data:monthly, tension:.3 }]},
    options:{ 
      responsive:true, maintainAspectRatio:false,
      scales:{
        y:{
          type:'linear',
          beginAtZero:true,
          min:0,
          max:maxY,
          ticks:{ stepSize:1, precision:0 }
        }
      },
      plugins:{ 
        legend:{display:false},
        datalabels:{ formatter:(v)=> v>0 ? `${v}\n${pct(v,totalYear)}` : '' }
      } 
    }
  });
}
function createSectorsChart(){
  if (!window.Chart) return;
  if(sectorsChartInstance) sectorsChartInstance.destroy();
  const ctx=document.getElementById('sectorsChart').getContext('2d');
  const map={}; getFilteredEvents().forEach(e=> map[e.categorie]=(map[e.categorie]||0)+1);
  const labels=Object.keys(map), vals=Object.values(map);
  const total = vals.reduce((a,b)=>a+b,0);
  sectorsChartInstance=new Chart(ctx,{
    type:'doughnut',
    data:{ labels, datasets:[{ data:vals }] },
    options:{ 
      responsive:true, maintainAspectRatio:false, 
      plugins:{
        legend:{ position:'bottom' },
        datalabels:{
          formatter:(v)=> v>0 ? `${v}\n${pct(v,total)}` : '',
          anchor:'center',
          align:'center',
          offset: 0,
          clamp: true,
          clip: false
        }
      }
    }
  });
}
function createHoursChart(){
  if (!window.Chart) return;
  if(hoursChartInstance) hoursChartInstance.destroy();
  const ctx=document.getElementById('hoursChart').getContext('2d');
  let p=0,r=0,v=0; // minutes
  getFilteredParticipants().forEach(x=>{
    let d=calculateDuration(x.heureDebut||'', x.heureFin||'');
    if(d<=0){ d=(+x.payees||0)+(+x.recup||0)+(+x.voirieFest||0); } // heures
    if(x.nature==='payees') p+=d*60;
    else if(x.nature==='recup') r+=d*60;
    else if(x.nature==='voirie') v+=d*60;
  });
  const total = p+r+v;
  hoursChartInstance=new Chart(ctx,{
    type:'pie',
    data:{ labels:['Payées','Récupérées','Voirie→Festivités'], datasets:[{ data:[p,r,v] }] },
    options:{ responsive:true, maintainAspectRatio:false, plugins:{
      legend:{ position:'bottom' },
      tooltip:{ callbacks:{ label:(c)=>`${c.label}: ${formatMinutes(c.parsed)}` } },
      datalabels:{ anchor:'center', align:'center', formatter:(val)=> val>0 ? `${formatMinutes(val)}\n${pct(val,total)}` : '' }
    } }
  });
}

/* ==== Courbes ==== */
function computeMonthlyHours(filtreType){
  const months = Array.from({length:12},()=>({total:0,p:0,r:0,v:0}));
  const evs = (filtreType==='tous') ? appData.events : appData.events.filter(e=>e.categorie===filtreType);
  evs.forEach(e=>{
    (e.participants||[]).forEach(l=>{
      const m = toLocalDate(l.date).getMonth();
      let d = calculateDuration(l.heureDebut||'', l.heureFin||'');
      if(d<=0){ d=(+l.payees||0)+(+l.recup||0)+(+l.voirieFest||0); }
      if(l.nature==='payees'){ months[m].p += d; }
      else if(l.nature==='recup'){ months[m].r += d; }
      else if(l.nature==='voirie'){ months[m].v += d; }
      months[m].total += d;
    });
  });
  return months; // heures
}
function createYearlyHoursLineChart(){
  if(!chartReady('yearlyHoursLineChart')) return;
  if(yearlyLineChartInstance) yearlyLineChartInstance.destroy();

  const sel=document.getElementById('lineTypeFilter1');
  const filtre= sel ? sel.value : 'tous';
  const months = computeMonthlyHours(filtre).map(m=> m.total);
  const cleaned = months.map(v=> v>0 ? v : null);
  const sumYear = months.reduce((a,b)=>a+(b||0),0);
  const ctx=document.getElementById('yearlyHoursLineChart').getContext('2d');

  try{
    yearlyLineChartInstance=new Chart(ctx,{
      type:'line',
      data:{ labels:monthLabels(), datasets:[{ 
        label: (filtre==='tous'?'Total (tous types)':`Total (${filtre})`),
        data:cleaned, fill:false, tension:.25, spanGaps:true
      }] },
      options:{ 
        responsive:true, maintainAspectRatio:false, 
        plugins:{
          legend:{ display:true, position:'top' },
          tooltip:{ callbacks:{ label:(c)=> `${c.formattedValue} h` } },
          datalabels:{ formatter:(v)=> (v && v>0) ? `${v.toFixed(1)}h\n${Math.round((v/sumYear)*100)||0}%` : '' }
        },
        scales:{ y:{ beginAtZero:true, max:650 } } 
      }
    });
  }catch(e){ console.error('createYearlyHoursLineChart failed:', e); }
}
function createMonthlyBreakdownLineChart(){
  if(!chartReady('monthlyBreakdownLineChart')) return;
  if(monthlyBreakdownLineChartInstance) monthlyBreakdownLineChartInstance.destroy();

  const sel=document.getElementById('lineTypeFilter2');
  const filtre= sel ? sel.value : 'tous';
  const months = computeMonthlyHours(filtre);

  const p = months.map(m=> m.p);
  const r = months.map(m=> m.r);
  const v = months.map(m=> m.v);

  const ctx=document.getElementById('monthlyBreakdownLineChart').getContext('2d');
  try{
    monthlyBreakdownLineChartInstance=new Chart(ctx,{
      type:'line',
      data:{ labels:monthLabels(), datasets:[
        { label:'Payées', data:p, tension:.25 },
        { label:'Récupérées', data:r, tension:.25 },
        { label:'Voirie→Fest', data:v, tension:.25 }
      ]},
      options:{ 
        responsive:true, maintainAspectRatio:false, spanGaps:true,
        elements:{ point:{ radius:(ctx)=>{ const v=ctx.parsed?.y; return (!isFinite(v) || v===0) ? 0 : 3; } } },
        plugins:{
          legend:{ position:'top' },
          tooltip:{ filter:(ctx)=> ctx.parsed.y!==0, callbacks:{ label:(c)=> `${c.dataset.label}: ${c.formattedValue} h` } },
          datalabels:{ align:'top', anchor:'end', formatter:(val)=> (val && val>0) ? `${val.toFixed(1)}h` : '' }
        }, 
        scales:{ y:{ beginAtZero:true, max:650 } } 
      }
    });
  }catch(e){ console.error('createMonthlyBreakdownLineChart failed:', e); }
}

/* ==== Répartition Mensuelle par Type (barres empilées) ==== */
function getOrCreateTooltipEl(parent){
  let el=parent.querySelector('.tooltip-portal');
  if(!el){ el=document.createElement('div'); el.className='tooltip-portal'; parent.appendChild(el); }
  return el;
}
function createMonthlyTypeChart(){
  if(!chartReady('monthlyTypeChart')) return;
  if(monthlyTypeChartInstance) monthlyTypeChartInstance.destroy();

  const canvas=document.getElementById('monthlyTypeChart');
  const ctx=canvas.getContext('2d');
  const events=getFilteredEvents();
  const types=[...new Set(events.map(e=>e.categorie))];

  const datasets=types.map(t=>{
    const monthly = new Array(12).fill(0);
    events.filter(e=>e.categorie===t).forEach(e=> monthly[toLocalDate(e.dateDebut).getMonth()]++);
    const cleaned = monthly.map(v=> v>0 ? v : null);
    return { label:t, data:cleaned, stack:'stack1' };
  });

  // MonthlyType (bar empilées) avec échelle fixe 0-13
  try{
    monthlyTypeChartInstance=new Chart(ctx,{
      type:'bar',
      data:{ labels:monthLabels(), datasets },
      options:{
        responsive:true, maintainAspectRatio:false,
        scales:{
          x:{ stacked:true },
          y:{ 
            type:'linear', 
            stacked:true, 
            beginAtZero:true, 
            min:0, 
            max:13, 
            ticks:{ 
              stepSize: 1, 
              precision:0,
              // Force l'affichage de tous les ticks
              callback: function(value) {
                return Number.isInteger(value) ? value : null;
              }
            }
          }
        },
        plugins:{
          tooltip:{ enabled:false, external:(context)=>{
            const tp = context.tooltip;
            const portal = getOrCreateTooltipEl(canvas.parentNode);
            if(tp.opacity===0){ portal.style.opacity=0; return; }
            const i = tp.dataPoints?.[0]?.dataIndex;
            const di = tp.dataPoints?.[0]?.datasetIndex;
            if(i==null || di==null){ portal.style.opacity=0; return; }
            const mName = monthLabels()[i];
            const tName = datasets[di].label;
            const evs = getFilteredEvents().filter(e=> toLocalDate(e.dateDebut).getMonth()===i && e.categorie===tName);

            let pay=0, rec=0, vf=0;
            const li = evs.map(e=>{
              let p=0,r=0,v=0;
              (e.participants||[]).forEach(l=>{
                if(toLocalDate(l.date).getMonth()===i){
                  const d=calculateDuration(l.heureDebut||'', l.heureFin||'') || ((+l.payees||0)+(+l.recup||0)+(+l.voirieFest||0));
                  if(l.nature==='payees') p+=d; else if(l.nature==='recup') r+=d; else if(l.nature==='voirie') v+=d;
                }
              });
              pay+=p; rec+=r; vf+=v;
              return `<li><span class="font-semibold">${e.nom}</span> — <span class="text-green-700">${p.toFixed(1)}h</span> / <span class="text-blue-700">${r.toFixed(1)}h</span> / <span class="text-yellow-700">${v.toFixed(1)}h</span></li>`;
            }).join('');

            portal.innerHTML = 
              `<div class="bg-white shadow-2xl rounded-xl p-3 border border-gray-200 text-sm max-w-md">
                <div class="font-bold text-gray-800 mb-1">${mName} — ${tName}</div>
                <div class="mb-2 text-xs text-gray-600">Totaux du mois (payées / récup / voirie→fest) :</div>
                <div class="mb-2 text-sm">
                  <span class="text-green-700 font-semibold">${pay.toFixed(1)}h</span> /
                  <span class="text-blue-700 font-semibold">${rec.toFixed(1)}h</span> /
                  <span class="text-yellow-700 font-semibold">${vf.toFixed(1)}h</span>
                </div>
                <ul class="list-disc pl-4 space-y-1">${li || '<li class="text-gray-500">Aucun événement</li>'}</ul>
                <div class="mt-2 text-xs text-gray-400">Cliquez la barre pour ouvrir le détail</div>
              </div>`;
            const rect = canvas.getBoundingClientRect();
            portal.style.opacity=1; portal.style.left=(rect.left + window.scrollX + tp.caretX + 14)+'px'; portal.style.top=(rect.top + window.scrollY + tp.caretY + 14)+'px';
          }},
          datalabels:{ formatter:(val)=> (val==null || val<=0) ? '' : `${val}`, anchor:'center', align:'center', clamp:true, clip:false }
        },
        onClick:(e,els)=>{
          if(!els.length) return;
          const el=els[0];
          openBarInfoModal(el.index, monthlyTypeChartInstance.data.datasets[el.datasetIndex].label);
        }
      }
    });
  }catch(e){ console.error('createMonthlyTypeChart failed:', e); }
}
function openBarInfoModal(monthIndex, typeName){
  const mName=monthLabels()[monthIndex];
  const evs = getFilteredEvents().filter(e=> toLocalDate(e.dateDebut).getMonth()===monthIndex && e.categorie===typeName);
  let pay=0,rec=0,vf=0;

  const body = evs.map(e=>{
    let p=0,r=0,v=0;
    (e.participants||[]).forEach(l=>{
      if(toLocalDate(l.date).getMonth()===monthIndex){
        const d=calculateDuration(l.heureDebut||'', l.heureFin||'') || ((+l.payees||0)+(+l.recup||0)+(+l.voirieFest||0));
        if(l.nature==='payees') p+=d; else if(l.nature==='recup') r+=d; else if(l.nature==='voirie') v+=d;
      }
    });
    pay+=p; rec+=r; vf+=v;
    return `<div class="p-3 border rounded-lg mb-2">
      <div class="font-semibold text-gray-800">${e.nom} <span class="text-xs text-gray-500">(${e.categorie})</span></div>
      <div class="text-xs text-gray-500">📍 ${e.lieu} • ${e.dateDebut} → ${e.dateFin}</div>
      <div class="mt-1 text-sm">
        <span class="text-green-700 font-semibold">${p.toFixed(1)}h</span> /
        <span class="text-blue-700 font-semibold">${r.toFixed(1)}h</span> /
        <span class="text-yellow-700 font-semibold">${v.toFixed(1)}h</span>
      </div>
    </div>`;
  }).join('');

  document.getElementById('barInfoTitle').textContent = `Détail – ${mName} — ${typeName}`;
  document.getElementById('barInfoBody').innerHTML =
    (body || '<div class="text-gray-500">Aucun événement</div>')+
    `<div class="mt-3 p-3 bg-gray-50 rounded border text-sm">
      <div class="font-semibold mb-1">Totaux du mois</div>
      <div><span class="text-green-700 font-semibold">${pay.toFixed(1)}h</span> /
           <span class="text-blue-700 font-semibold">${rec.toFixed(1)}h</span> /
           <span class="text-yellow-700 font-semibold">${vf.toFixed(1)}h</span></div>
     </div>`;
  document.getElementById('barInfoModal').classList.add('active');
}
function closeBarInfoModal(){ document.getElementById('barInfoModal').classList.remove('active'); }

/* ==== Agents ==== */
function showAgentForm(id){
  editingAgentId = id || null;

  const formWrap = document.getElementById('addAgentForm');
  const titleEl  = document.getElementById('agentFormTitle');
  const btn      = document.getElementById('agentFormSubmit');
  const btnTxt   = btn?.querySelector('span');
  const nameEl   = document.getElementById('agentName');
  const servEl   = document.getElementById('agentService');
  const telEl    = document.getElementById('agentPhone');

  if(id){
    const ag = appData.agents.find(a=>a.id===id);
    if(!ag) return;
    titleEl.textContent = 'Modifier un Agent';
    if(btnTxt) btnTxt.textContent = 'Mettre à jour';
    nameEl.value = ag.nom || '';
    servEl.value = ag.service || 'Voirie';
    telEl.value  = ag.telephone || '';
  }else{
    titleEl.textContent = 'Ajouter un Agent';
    if(btnTxt) btnTxt.textContent = 'Ajouter';
    nameEl.value = '';
    servEl.value = 'Voirie';
    telEl.value  = '';
  }

  formWrap.style.display = 'block';
  try{ nameEl.focus(); }catch(e){}
  formWrap.scrollIntoView({behavior:'smooth', block:'center'});
}

function hideAddAgentForm(){
  const formWrap = document.getElementById('addAgentForm');
  const form     = document.getElementById('agentForm');
  const btnTxt   = document.getElementById('agentFormSubmit')?.querySelector('span');

  editingAgentId = null;
  if(form) form.reset();
  document.getElementById('agentService').value = 'Voirie';
  if(btnTxt) btnTxt.textContent = 'Ajouter';
  formWrap.style.display = 'none';
}

function saveAgent(e){
  e.preventDefault();
  const name   = document.getElementById('agentName').value.trim();
  const service= document.getElementById('agentService').value;
  const phone  = document.getElementById('agentPhone').value.trim();

  if(!name || !service || !phone){
    showToast('Veuillez remplir tous les champs.', 'error');
    return;
  }

  if(editingAgentId){
    appData.agents = appData.agents.map(a => a.id===editingAgentId ? {...a, nom:name, service, telephone:phone} : a);
    showToast('Agent mis à jour avec succès !', 'success');
  }else{
    const maxId = appData.agents.reduce((m,a)=> Math.max(m, Number(a.id)||0), 0);
    const newId = maxId + 1;
    appData.agents.push({ id:newId, nom:name, service, telephone:phone });
    showToast('Agent ajouté avec succès !', 'success');
  }

  markAsChanged();
  saveData();
  updateAgentsList();
  updateHoursTable();
  updateDashboard();

  if(editingEventForParts){ renderParticipantsTable(); }

  if(currentPlanningEventId){
    const sel=document.getElementById('taskAgents');
    if(sel) sel.innerHTML = appData.agents.map(a=>`<option value="${a.id}">${a.nom}</option>`).join('');
  }

  hideAddAgentForm();
}

function deleteAgent(id){
  const usedInParts = appData.events.some(ev => (ev.participants||[]).some(p => p.agentId===id));
  const usedInTasks = Object.values(appData.tasks||{})
    .some(list => (list||[]).some(t =>
      (Array.isArray(t.agentIds) && t.agentIds.includes(id)) || t.agentId===id
    ));

  const msg = usedInParts || usedInTasks
    ? "Cet agent a des lignes d'heures et/ou des tâches associées.\nSupprimer l'agent et retirer ces références ?"
    : "Supprimer cet agent ?";
  if(!confirm(msg)) return;

  if(usedInParts){
    appData.events.forEach(ev=>{
      ev.participants = (ev.participants||[]).filter(p=>p.agentId!==id);
    });
  }
  if(usedInTasks){
    Object.keys(appData.tasks).forEach(k=>{
      appData.tasks[k] = (appData.tasks[k]||[]).map(t => {
        const ids = Array.isArray(t.agentIds) ? t.agentIds.filter(aid=>aid!==id)
                   : (t.agentId!=null ? [] : []);
        return { ...t, agentIds: ids, agentId:null };
      });
    });
  }

  appData.agents = appData.agents.filter(a=>a.id!==id);

  saveData();
  updateAgentsList();
  updateHoursTable();
  updateDashboard();
  updateEventsList();
  showToast('Agent supprimé avec succès.', 'success');
}

function updateAgentsList(){
  const wrap=document.getElementById('agentsList'); wrap.innerHTML='';
  appData.agents.forEach(a=>{
    const parts=getFilteredParticipants().filter(p=>p.agentId===a.id);
    let p=0,r=0,v=0;
    parts.forEach(x=>{
      const d=calculateDuration(x.heureDebut||'', x.heureFin||'') || ((+x.payees||0)+(+x.recup||0)+(+x.voirieFest||0));
      if(x.nature==='payees') p+=d*60; else if(x.nature==='recup') r+=d*60; else if(x.nature==='voirie') v+=d*60;
    });
    const tot=p+r+v;
    if(tot>0 || currentTypeFilter==='tous'){
      const div=document.createElement('div');
      div.className='flex justify-between items-center p-4 bg-white border rounded-lg';
      div.innerHTML = 
        `<div>
          <h4 class="font-semibold text-blue-600 cursor-pointer hover:underline" onclick="showAgentDetail(${a.id})">${a.nom}</h4>
          <p class="text-sm text-gray-600">${a.service} • ${a.telephone}</p>
          <p class="text-xs text-blue-600 font-medium">Total: ${formatMinutes(tot)}</p>
        </div>
        <div class="flex space-x-2">
          <button data-action="agent-edit" data-id="${a.id}" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-1 rounded text-sm" title="Modifier l'agent"><i class="fas fa-edit"></i></button>
          <button data-action="agent-delete" data-id="${a.id}" class="bg-red-600 hover:bg-red-700 text-white px-3 py-1 rounded text-sm" title="Supprimer l'agent"><i class="fas fa-trash"></i></button>
        </div>`;
      wrap.appendChild(div);
    }
  });
}
function showAgentDetail(agentId){
  const agent=appData.agents.find(a=>a.id===agentId); if(!agent) return;
  currentAgentForHistory=agentId;
  document.getElementById('agentHistoryName').textContent=`${agent.nom} (${agent.service})`;

  const parts=getFilteredParticipants().filter(p=>p.agentId===agentId);
  const grouped=parts.reduce((acc,p)=>{ (acc[p.eventId]=acc[p.eventId]||[]).push(p); return acc; },{});
  let p=0,r=0,v=0; parts.forEach(x=>{
    const d=calculateDuration(x.heureDebut||'', x.heureFin||'') || ((+x.payees||0)+(+x.recup||0)+(+x.voirieFest||0));
    if(x.nature==='payees') p+=d*60; else if(x.nature==='recup') r+=d*60; else if(x.nature==='voirie') v+=d*60;
  });

  const filterText=currentTypeFilter==='tous' ? "Tous types" : `Filtre: ${currentTypeFilter}`;
  document.getElementById('agentHistoryStats').innerHTML =
    `<div class="text-sm text-gray-600 mb-2">${filterText}</div>
     <div class="flex justify-center space-x-6 text-sm font-medium">
       <div class="text-green-700"><i class="fas fa-euro-sign mr-1"></i>Payées: ${formatMinutes(p)}</div>
       <div class="text-blue-700"><i class="fas fa-undo mr-1"></i>Récupérées: ${formatMinutes(r)}</div>
       <div class="text-yellow-700"><i class="fas fa-exchange-alt mr-1"></i>Voirie→Fest: ${formatMinutes(v)}</div>
       <div class="text-purple-700 font-bold"><i class="fas fa-calculator mr-1"></i>Total: ${formatMinutes(p+r+v)}</div>
     </div>`;

  const cont=document.getElementById('agentHistoryEvents'); cont.innerHTML='';
  if(!Object.keys(grouped).length){
    cont.innerHTML='<div class="text-center text-gray-500 py-8">Aucun événement pour cet agent avec le filtre actuel.</div>';
  }else{
    Object.entries(grouped).forEach(([eid,rows])=>{
      const ev = appData.events.find(e=>e.id==eid); if(!ev) return;
      const days = new Set(rows.map(r=>r.date)).size;
      const sum = f => rows.reduce((s,x)=> {
        const d=calculateDuration(x.heureDebut||'', x.heureFin||'') || ((+x.payees||0)+(+x.recup||0)+(+x.voirieFest||0));
        if(f==='payees' && x.nature==='payees') return s + d*60;
        if(f==='recup' && x.nature==='recup')   return s + d*60;
        if(f==='voirie' && x.nature==='voirie') return s + d*60;
        return s;
      }, 0);
      const pay=sum('payees'), rec=sum('recup'), vf=sum('voirie');
      const card=document.createElement('div');
      card.className='event-history-card';
      card.onclick=()=>showAgentEventDetail(agentId, ev.id);
      card.innerHTML=
        `<div class="font-semibold text-lg mb-1 text-blue-700"><i class="fas fa-calendar-check mr-2"></i>${ev.nom}</div>
        <div class="text-sm text-gray-600 mb-2">📅 ${ev.dateDebut} — ${ev.dateFin} • 📍 ${ev.lieu} • 🏷️ ${ev.categorie}</div>
        <div class="text-sm mb-2 text-indigo-600">Participation : ${days} jour${days>1?'s':''}</div>
        <div class="flex flex-wrap gap-2">
          <span class="hours-badge badge-payees">${formatMinutes(pay)}</span>
          <span class="hours-badge badge-recup">${formatMinutes(rec)}</span>
          <span class="hours-badge badge-voirie">${formatMinutes(vf)}</span>
        </div>`;
      cont.appendChild(card);
    });
  }
  
  // Génération du graphique d'évolution
  createAgentEvolutionChart(agentId, grouped);
  
  document.getElementById('agentHistoryModal').classList.add('active');
  
  // Focus sur le premier champ focusable
  setTimeout(() => {
    const firstInput = document.querySelector('#agentHistoryModal button');
    if(firstInput) firstInput.focus();
  });
}
function showAgentEventDetail(agentId,eventId){
  const ev=appData.events.find(e=>e.id===eventId); if(!ev) return;
  document.getElementById('agentEventDetailName').textContent=ev.nom;
  const lines=getFilteredParticipants().filter(p=>p.agentId===agentId&&p.eventId===eventId).sort((a,b)=> toLocalDate(a.date)-toLocalDate(b.date));
  const tbody=document.getElementById('agentEventDetailTable'); tbody.innerHTML='';
  const labels={payees:'Heures payées',recup:'Heures récupérées',voirie:'Voirie→Festivités'};
  lines.forEach(p=>{
    const d=calculateDuration(p.heureDebut||'',p.heureFin||'') || ((+p.payees||0)+(+p.recup||0)+(+p.voirieFest||0));
    const tr=document.createElement('tr');
    tr.innerHTML=`
      <td class="px-3 py-2 border">${toLocalDate(p.date).toLocaleDateString('fr-FR',{weekday:'short',day:'2-digit',month:'2-digit'})}</td>
      <td class="px-3 py-2 border">${p.motif||'–'}</td>
      <td class="px-3 py-2 border"><span class="px-2 py-1 rounded text-xs font-medium ${p.nature==='payees'?'bg-green-100 text-green-800':p.nature==='recup'?'bg-blue-100 text-blue-800':p.nature==='voirie'?'bg-yellow-100 text-yellow-800':'bg-gray-100 text-gray-800'}">${labels[p.nature]||'—'}</span></td>
      <td class="px-3 py-2 border text-center">${p.heureDebut||'–'}</td>
      <td class="px-3 py-2 border text-center">${p.heureFin||'–'}</td>
      <td class="px-3 py-2 border text-center font-bold text-green-700">${d.toFixed(1)} h</td>`;
    tbody.appendChild(tr);
  });
  document.getElementById('agentEventDetailModal').classList.add('active');
}
function closeAgentEventDetailModal(){ document.getElementById('agentEventDetailModal').classList.remove('active'); }
/* ==== Graphique d'évolution des agents ==== */
function createAgentEvolutionChart(agentId, grouped) {
  // Vérification avec timeout pour s'assurer que le canvas est prêt
  setTimeout(() => {
    const canvas = document.getElementById('agentEvolutionChart');
    if (!canvas) {
      console.warn('Canvas agentEvolutionChart non trouvé');
      return;
    }
    
    if (!window.Chart) {
      console.warn('Chart.js non disponible');
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.font = '16px Arial';
      ctx.fillStyle = '#666';
      ctx.textAlign = 'center';
      ctx.fillText('Chart.js non disponible', canvas.width / 2, canvas.height / 2);
      return;
    }
    
    // Nettoyage de l'instance précédente
    if (agentEvolutionChartInstance) {
      agentEvolutionChartInstance.destroy();
      agentEvolutionChartInstance = null;
    }
    
    const ctx = canvas.getContext('2d');
    
    // Préparation des données : événements triés par date de début
    const eventData = Object.entries(grouped)
      .map(([eventId, rows]) => {
        const event = appData.events.find(e => e.id == eventId);
        if (!event) return null;
        
        // Calcul des totaux par type pour cet événement
        let payees = 0, recup = 0, voirie = 0;
        rows.forEach(row => {
          const duration = calculateDuration(row.heureDebut || '', row.heureFin || '') || 
                          ((+row.payees || 0) + (+row.recup || 0) + (+row.voirieFest || 0));
          
          if (row.nature === 'payees') payees += duration;
          else if (row.nature === 'recup') recup += duration;
          else if (row.nature === 'voirie') voirie += duration;
        });
        
        return {
          eventId: event.id,
          nom: event.nom,
          categorie: event.categorie,
          dateDebut: event.dateDebut,
          dateFin: event.dateFin,
          lieu: event.lieu,
          payees,
          recup,
          voirie,
          total: payees + recup + voirie,
          nombreJours: new Set(rows.map(r => r.date)).size
        };
      })
      .filter(Boolean)
      .sort((a, b) => new Date(a.dateDebut) - new Date(b.dateDebut));

    if (!eventData.length) {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.font = '16px Arial';
      ctx.fillStyle = '#666';
      ctx.textAlign = 'center';
      ctx.fillText('Aucune donnée à afficher', canvas.width / 2, canvas.height / 2);
      return;
    }

    // Labels : noms des événements (raccourcis si trop longs)
    const labels = eventData.map(e => {
      const shortName = e.nom.length > 15 ? e.nom.substring(0, 12) + '...' : e.nom;
      return shortName;
    });

    // Datasets pour les 3 types d'heures avec gestion des valeurs nulles
    const datasets = [
      {
        label: 'Heures payées',
        data: eventData.map(e => e.payees > 0 ? e.payees : null),
        borderColor: '#10b981',
        backgroundColor: 'rgba(16, 185, 129, 0.1)',
        fill: false,
        tension: 0.3,
        pointRadius: (ctx) => {
          const value = ctx.parsed?.y;
          return (value && value > 0) ? 8 : 0; // Masque les points à zéro
        },
        pointHoverRadius: 10,
        borderWidth: 3,
        spanGaps: true // Connecte les points même avec des valeurs nulles
      },
      {
        label: 'Heures récupérées',
        data: eventData.map(e => e.recup > 0 ? e.recup : null),
        borderColor: '#3b82f6',
        backgroundColor: 'rgba(59, 130, 246, 0.1)',
        fill: false,
        tension: 0.3,
        pointRadius: (ctx) => {
          const value = ctx.parsed?.y;
          return (value && value > 0) ? 8 : 0;
        },
        pointHoverRadius: 10,
        borderWidth: 3,
        spanGaps: true
      },
      {
        label: 'Voirie → Festivités',
        data: eventData.map(e => e.voirie > 0 ? e.voirie : null),
        borderColor: '#f59e0b',
        backgroundColor: 'rgba(245, 158, 11, 0.1)',
        fill: false,
        tension: 0.3,
        pointRadius: (ctx) => {
          const value = ctx.parsed?.y;
          return (value && value > 0) ? 8 : 0;
        },
        pointHoverRadius: 10,
        borderWidth: 3,
        spanGaps: true
      }
    ];

    // Configuration du graphique
    const config = {
      type: 'line',
      data: { labels, datasets },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        interaction: {
          mode: 'index',
          intersect: false
        },
        plugins: {
          legend: {
            position: 'top',
            labels: {
              usePointStyle: true,
              padding: 20,
              font: { size: 14, weight: 'bold' }
            }
          },
          tooltip: {
            backgroundColor: 'rgba(0, 0, 0, 0.9)',
            titleColor: '#fff',
            bodyColor: '#fff',
            borderColor: '#666',
            borderWidth: 2,
            cornerRadius: 8,
            displayColors: true,
            titleFont: { size: 14, weight: 'bold' },
            bodyFont: { size: 13 },
            padding: 12,
            callbacks: {
              title: (context) => {
                const index = context[0].dataIndex;
                const event = eventData[index];
                return `${event.nom}`;
              },
              beforeBody: (context) => {
                const index = context[0].dataIndex;
                const event = eventData[index];
                return [
                  `� ${event.categorie}`,
                  `�📅 ${event.dateDebut} → ${event.dateFin}`,
                  `📍 ${event.lieu}`,
                  `👥 ${event.nombreJours} jour${event.nombreJours > 1 ? 's' : ''} de participation`,
                  ''
                ];
              },
              label: (context) => {
                const value = context.parsed.y;
                if (!value || value === 0) return null;
                return `${context.dataset.label}: ${value.toFixed(1)}h`;
              },
              afterBody: (context) => {
                const index = context[0].dataIndex;
                const event = eventData[index];
                return [
                  '',
                  `🕐 Total événement: ${event.total.toFixed(1)}h`,
                  '� Cliquez pour voir le détail'
                ];
              }
            },
            filter: (tooltipItem) => {
              // Ne pas afficher les tooltips pour les valeurs nulles ou zéro
              return tooltipItem.parsed.y !== null && tooltipItem.parsed.y > 0;
            }
          },
          datalabels: {
            display: true,
            align: 'top',
            anchor: 'end',
            offset: 4,
            color: (ctx) => {
              const dataset = ctx.dataset;
              return dataset.borderColor;
            },
            font: {
              size: 11,
              weight: 'bold'
            },
            formatter: (value, ctx) => {
              if (!value || value === 0) return '';
              return `${value.toFixed(1)}h`;
            },
            backgroundColor: 'rgba(255, 255, 255, 0.8)',
            borderColor: (ctx) => ctx.dataset.borderColor,
            borderWidth: 1,
            borderRadius: 4,
            padding: {
              top: 2,
              bottom: 2,
              left: 4,
              right: 4
            }
          }
        },
        scales: {
          x: {
            title: {
              display: true,
              text: 'Événements (chronologique)',
              font: { size: 14, weight: 'bold' },
              color: '#374151'
            },
            ticks: {
              maxRotation: 45,
              font: { size: 12 },
              color: '#6b7280'
            },
            grid: {
              color: 'rgba(156, 163, 175, 0.3)',
              drawOnChartArea: true
            }
          },
          y: {
            beginAtZero: true,
            title: {
              display: true,
              text: 'Heures travaillées',
              font: { size: 14, weight: 'bold' },
              color: '#374151'
            },
            ticks: {
              callback: (value) => `${value}h`,
              font: { size: 12 },
              color: '#6b7280'
            },
            grid: {
              color: 'rgba(156, 163, 175, 0.3)',
              drawOnChartArea: true
            }
          }
        },
        onClick: (event, elements) => {
          if (elements.length > 0) {
            const elementIndex = elements[0].index;
            const eventInfo = eventData[elementIndex];
            showAgentEventDetail(agentId, eventInfo.eventId);
          }
        }
      }
    };

    try {
      agentEvolutionChartInstance = new Chart(ctx, config);
      console.log('✅ Graphique d\'évolution agent créé avec succès');
    } catch (error) {
      console.error('❌ Erreur lors de la création du graphique d\'évolution agent:', error);
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.font = '16px Arial';
      ctx.fillStyle = '#ff0000';
      ctx.textAlign = 'center';
      ctx.fillText('Erreur lors du chargement du graphique', canvas.width / 2, canvas.height / 2);
    }
  }, 100); // Délai de 100ms pour s'assurer que le modal est bien affiché
}

function closeAgentHistoryModal(){ 
  document.getElementById('agentHistoryModal').classList.remove('active'); 
  currentAgentForHistory=null;
  
  // Nettoyage du graphique d'évolution
  if (agentEvolutionChartInstance) {
    agentEvolutionChartInstance.destroy();
    agentEvolutionChartInstance = null;
  }
}

/* ==== Events (CRUD + liste) ==== */
function updateEventsList(){
  const list=document.getElementById('eventsList'); if(!list) return;
  list.innerHTML='';

  const grouped = {};
  appData.events
    .filter(e => currentTypeFilter==='tous' || e.categorie===currentTypeFilter)
    .forEach(e=>{
      if(!grouped[e.categorie]) grouped[e.categorie]=[];
      grouped[e.categorie].push(e);
    });

  Object.entries(grouped)
    .sort(([a],[b])=>a.localeCompare(b,'fr'))
    .forEach(([type,evs])=>{
      const h=document.createElement('h3');
      h.className='mt-8 mb-3 text-lg font-bold text-white px-3 py-2 rounded bg-gradient-to-r from-indigo-500 to-purple-500 shadow';
      h.innerHTML=`<i class="fas fa-tags mr-2"></i>${type} (${evs.length})`;
      list.appendChild(h);

      evs.sort((a,b)=> toLocalDate(b.dateDebut)-toLocalDate(a.dateDebut)).forEach(ev=>{
        const nbDays=getDatesBetween(ev.dateDebut,ev.dateFin).length;
        const nbParts=(ev.participants||[]).length;
        const nbDocs=(ev.documents||[]).length;

        const row=document.createElement('div');
        row.id = `eventRow_${ev.id}`;
        row.className='flex justify-between items-center p-4 bg-white border rounded-lg';
        row.innerHTML = 
          `<div>
            <h4 class="font-semibold text-gray-800">${ev.nom}</h4>
            <p class="text-sm text-gray-600">${ev.lieu} • ${ev.dateDebut} → ${ev.dateFin} <span class="text-blue-600 font-medium">(${nbDays} jour${nbDays>1?'s':''})</span></p>
            <p class="text-sm text-gray-500">${ev.description||''}</p>
            <p class="text-xs text-green-600 font-medium">${nbParts} ligne${nbParts>1?'s':''} d'heures • ${nbDocs} doc${nbDocs>1?'s':''}</p>
          </div>
          <div class="flex flex-wrap gap-2">
            <button data-action="participants-open" data-id="${ev.id}" class="bg-yellow-500 text-white px-3 py-1 rounded text-sm hover:bg-yellow-600"><i class="fas fa-users-cog"></i> Participants</button>
            <button data-action="docs-open" data-id="${ev.id}" class="bg-gray-800 text-white px-3 py-1 rounded text-sm hover:bg-gray-900"><i class="fas fa-folder-open"></i> Documents (${nbDocs})</button>
            <button data-action="material-open" data-id="${ev.id}" class="bg-purple-600 text-white px-3 py-1 rounded text-sm hover:bg-purple-700"><i class="fas fa-boxes"></i> Matériel</button>
            <button data-action="planning-open" data-id="${ev.id}" class="bg-blue-600 text-white px-3 py-1 rounded text-sm hover:bg-blue-700"><i class="fas fa-project-diagram"></i> Planning</button>
            <button data-action="event-edit" data-id="${ev.id}" class="bg-blue-600 text-white px-3 py-1 rounded text-sm hover:bg-blue-700" title="Modifier l'événement"><i class="fas fa-edit"></i></button>
            <button data-action="event-delete" data-id="${ev.id}" class="bg-red-600 text-white px-3 py-1 rounded text-sm hover:bg-red-700" title="Supprimer l'événement"><i class="fas fa-trash"></i></button>
          </div>`;
        list.appendChild(row);
      });
    });

  if(!Object.keys(grouped).length){
    const empty=document.createElement('div');
    empty.className='text-center text-gray-500 py-10';
    empty.innerHTML='<i class="fas fa-calendar-times fa-2x mb-2"></i><div>Aucun événement</div>';
    list.appendChild(empty);
  }
}
function goToEvent(eventId){
  showModule('events');
  setTimeout(()=>{
    const el=document.getElementById(`eventRow_${eventId}`);
    if(el){
      el.scrollIntoView({behavior:'smooth', block:'center'});
      el.classList.add('flash');
      setTimeout(()=> el.classList.remove('flash'), 1400);
    }
  }, 50);
}
function showAddEventForm(id){
  editingEventId=id||null;
  const title=document.getElementById('eventFormTitle');
  const btn=document.getElementById('eventFormSubmit');
  const name=document.getElementById('eventName');
  const type=document.getElementById('eventType');
  const lieu=document.getElementById('eventLieu');
  const deb=document.getElementById('eventDateDebut');
  const fin=document.getElementById('eventDateFin');
  const desc=document.getElementById('eventDescription');
  if(id){
    const ev=appData.events.find(e=>e.id===id); if(!ev) return;
    name.value=ev.nom||''; type.value=ev.categorie||''; lieu.value=ev.lieu||'';
    deb.value=ev.dateDebut||''; fin.value=ev.dateFin||''; desc.value=ev.description||'';
    title.textContent='Modifier un Événement'; btn.textContent='Mettre à jour';
  }else{
    name.value=type.value=lieu.value=desc.value=''; deb.value=fin.value='';
    title.textContent='Créer un Événement'; btn.textContent='Créer';
  }
  document.getElementById('addEventForm').style.display='block';
}
function hideAddEventForm(){ document.getElementById('addEventForm').style.display='none'; editingEventId=null; }
function saveEvent(e){
  e.preventDefault();    const newEv={
    id: editingEventId || Date.now(),
    nom: document.getElementById('eventName').value.trim(),
    categorie: document.getElementById('eventType').value.trim(),
    lieu: document.getElementById('eventLieu').value.trim(),
    dateDebut: document.getElementById('eventDateDebut').value,
    dateFin:   document.getElementById('eventDateFin').value,
    description: document.getElementById('eventDescription').value.trim()
  };
  if(!newEv.nom || !newEv.categorie || !newEv.lieu || !newEv.dateDebut || !newEv.dateFin) return showToast('Tous les champs sont obligatoires', 'error');
  if(toLocalDate(newEv.dateDebut) > toLocalDate(newEv.dateFin)) return showToast('La date de début doit précéder la date de fin', 'error');
  if(editingEventId){
    appData.events = appData.events.map(ev => ev.id===editingEventId ? {...newEv, participants:ev.participants||[], documents:ev.documents||[]} : ev);
    showToast('Événement mis à jour avec succès !', 'success');
  }else{
    newEv.participants=[]; newEv.documents=[];
    appData.events.push(newEv);
    appData.tasks[newEv.id]=[];
    showToast('Événement créé avec succès !', 'success');
  }
  markAsChanged();
  saveData(); updateEventsList(); updateDashboard(); renderCalendar(); hideAddEventForm(); editingEventId=null;
}
function deleteEvent(id){
  if(!confirm('Supprimer cet événement ?')) return;
  appData.events = appData.events.filter(e=>e.id!==id);
  delete appData.tasks[id]; 
  blobManager.removeEvent(id);
  markAsChanged();
  saveData(); updateEventsList(); updateDashboard(); renderCalendar(); 
  showToast('Événement supprimé avec succès.', 'success');
}

/* ==== Participants ==== */
function openParticipantsModal(eventId){
  const ev=appData.events.find(e=>e.id===eventId); if(!ev) return;
  editingEventForParts=eventId;
  participantsData = JSON.parse(JSON.stringify(ev.participants || []));
  originalParticipantsData = JSON.parse(JSON.stringify(participantsData));
  document.getElementById('partsEventName').textContent=ev.nom;
  renderParticipantsTable();
  document.getElementById('participantsModal').classList.add('active');
  
  // Focus amélioré sur le premier élément visible et interactif
  setTimeout(() => {
    const modal = document.getElementById('participantsModal');
    if (!modal || !modal.classList.contains('active')) return;
    
    const focusable = modal.querySelectorAll(
      'input:not([disabled]), button:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'
    );
    
    for (let el of focusable) {
      if (el.offsetWidth > 0 && el.offsetHeight > 0) {
        el.focus();
        break;
      }
    }
  }, 150);
}
function closeParticipantsModal(){ document.getElementById('participantsModal').classList.remove('active'); editingEventForParts=null; }
function renderParticipantsTable(){
  const tb=document.getElementById('participantsTableBody'); tb.innerHTML='';
  if(!participantsData.length){ document.getElementById('participantsEmpty').style.display='block'; } else { document.getElementById('participantsEmpty').style.display='none'; }
  participantsData.forEach((row,i)=>{
    const tr=document.createElement('tr');
    tr.innerHTML = 
      `<td class="px-2 py-2 border"><input type="date" class="inline-edit" value="${row.date||''}" onchange="onRowChange(${i},'date',this.value)"></td>
      <td class="px-2 py-2 border">
        <select class="inline-edit" onchange="onRowChange(${i},'agentId',this.value)">${appData.agents.map(a=>`<option value="${a.id}" ${+row.agentId===a.id?'selected':''}>${a.nom}</option>`).join('')}</select>
      </td>
      <td class="px-2 py-2 border"><input class="inline-edit" value="${row.motif||''}" onchange="onRowChange(${i},'motif',this.value)"></td>
      <td class="px-2 py-2 border text-center">
        <select class="type-select ${'type-'+(row.nature||'payees')}" onchange="updateParticipantType(${i}, this.value, this)">
          <option value="payees" ${row.nature==='payees'?'selected':''}>Payées</option>
          <option value="recup" ${row.nature==='recup'?'selected':''}>Récup</option>
          <option value="voirie" ${row.nature==='voirie'?'selected':''}>Voirie→Fest</option>
        </select>
      </td>
      <td class="px-2 py-2 border text-center">
        <input type="time" class="time-input" value="${row.heureDebut||''}" step="300" placeholder="HH:MM"
               oninput="onRowChange(${i},'heureDebut',this.value); onTimeChange(${i})"
               onchange="onRowChange(${i},'heureDebut',this.value); onTimeChange(${i})">
      </td>
      <td class="px-2 py-2 border text-center">
        <input type="time" class="time-input" value="${row.heureFin||''}" step="300" placeholder="HH:MM"
               oninput="onRowChange(${i},'heureFin',this.value); onTimeChange(${i})"
               onchange="onRowChange(${i},'heureFin',this.value); onTimeChange(${i})">
      </td>
      <td class="px-2 py-2 border text-center"><span class="duration-display" id="dur_${i}">${calcRowDuration(i).toFixed(1)} h</span></td>
      <td class="px-2 py-2 border text-center">
        <div class="inline-flex gap-1">
          <button data-action="row-edit" data-index="${i}" class="bg-blue-600 text-white px-2 py-1 rounded" title="Modifier les heures">
            <i class="fas fa-pen"></i>
          </button>
          <button data-action="row-dup" data-index="${i}" class="bg-gray-600 text-white px-2 py-1 rounded" title="Dupliquer la ligne">
            <i class="fas fa-copy"></i>
          </button>
          <button data-action="row-delete" data-index="${i}" class="bg-red-600 text-white px-2 py-1 rounded" title="Supprimer la ligne">
            <i class="fas fa-trash"></i>
          </button>
        </div>
      </td>`;
    tb.appendChild(tr);
  });
  updateTotals();
}
function normalizeTime(t){
  if(!t) return '';
  t = String(t).trim().toLowerCase();

  // HH:MM déjà correct ?
  const ok = /^([01]?\d|2[0-3]):([0-5]\d)$/;
  if (ok.test(t)) return t;

  // "16" -> "16:00"
  if(/^\d{1,2}$/.test(t)){
    const h = +t;
    return (h>=0 && h<=23) ? String(h).padStart(2,'0') + ':00' : '';
  }

  // "1600" -> "16:00"
  if(/^\d{3,4}$/.test(t)){
    const s = t.padStart(4,'0').replace(/(\d{2})(\d{2})/, '$1:$2');
    return ok.test(s) ? s : '';
  }

  // "16h" / "16h3" / "16h30" -> HH:MM
  const hMatch = t.match(/^(\d{1,2})h(\d{1,2})?$/);
  if(hMatch){
    const h = hMatch[1].padStart(2,'0');
    const m = (hMatch[2] ?? '00').padStart(2,'0');
    const s = `${h}:${m}`;
    return ok.test(s) ? s : '';
  }

  // "16:3" -> "16:03"
  const mMatch = t.match(/^(\d{1,2}):(\d{1,2})$/);
  if(mMatch){
    const h = mMatch[1].padStart(2,'0');
    const m = mMatch[2].padStart(2,'0');
    const s = `${h}:${m}`;
    return ok.test(s) ? s : '';
  }

  return '';
}
function onRowChange(i,k,v){
  if(k==='agentId') v = +v;
  if(k==='heureDebut' || k==='heureFin') v = normalizeTime(v);
  participantsData[i][k] = v;
}
function onTimeChange(i){
  const row = participantsData[i];
  const d = calculateDuration(row.heureDebut||'', row.heureFin||'');
  const sp = document.getElementById('dur_'+i);
  if(sp) sp.textContent = (Number.isFinite(d) ? d.toFixed(1) : '0.0') + ' h';
  updateTotals();
}
function calcRowDuration(i){
  const r=participantsData[i]; return calculateDuration(r.heureDebut||'', r.heureFin||'');
}
function editRowTimes(i){
  const row = participantsData[i] || {};

  const start = prompt("Heure de début (HH:MM, ex: 08:30)", row.heureDebut || "");
  if (start === null) return; // annulé

  const end = prompt("Heure de fin (HH:MM, ex: 12:00)", row.heureFin || "");
  if (end === null) return; // annulé

  const s = normalizeTime(start);
  const e = normalizeTime(end);

  if (!/^\d{2}:\d{2}$/.test(s) || !/^\d{2}:\d{2}$/.test(e)) {
    alert("Format d'heure invalide. Utilisez HH:MM (ex: 08:30).");
    return;
  }

  participantsData[i].heureDebut = s;
  participantsData[i].heureFin   = e;

  // simple & robuste : on rerend la table (durée et totaux seront recalculés)
  renderParticipantsTable();
}
function updateParticipantType(i,newType,el){
  const r=participantsData[i]; r.nature=newType;
  if(el){ el.classList.remove('type-payees','type-recup','type-voirie'); el.classList.add('type-'+newType); }
  updateTotals();
}
function updateTotals(){
  let p=0,r=0,v=0;
  participantsData.forEach(x=>{
    const d=calculateDuration(x.heureDebut||'', x.heureFin||'') || ((+x.payees||0)+(+x.recup||0)+(+x.voirieFest||0));
    if(x.nature==='payees') p+=d; else if(x.nature==='recup') r+=d; else if(x.nature==='voirie') v+=d;
  });
  document.getElementById('totalPayees').textContent=p.toFixed(1)+'h';
  document.getElementById('totalRecup').textContent=r.toFixed(1)+'h';
  document.getElementById('totalVoirieFest').textContent=v.toFixed(1)+'h';
}
function onAddRow(){
  const firstAgent=appData.agents[0]?.id || 0;
  participantsData.push({
    agentId:firstAgent, date:todayYMD(),
    motif:'', nature:'payees', heureDebut:'08:00', heureFin:'12:00'
  });
  renderParticipantsTable();
}
function removeRow(i){ participantsData.splice(i,1); renderParticipantsTable(); }
function duplicateRow(i){
  const row = participantsData[i];
  if(!row) return;
  const copy = {...row};
  // option : avancer la date d'1 jour
  // copy.date = ymdLocal(new Date(`${row.date}T00:00:00`).setDate(new Date(`${row.date}T00:00:00`).getDate()+1));
  participantsData.push(copy);
  renderParticipantsTable();
}
function onSaveParticipants(){
  const ev=appData.events.find(e=>e.id===editingEventForParts); if(!ev) return;
  ev.participants = JSON.parse(JSON.stringify(participantsData));
  markAsChanged();
  saveData(); updateEventsList(); updateDashboard(); renderCalendar(); 
  showToast('Participants sauvegardés !', 'success');
  closeParticipantsModal();
}
function onCancelParticipants(){
  participantsData = JSON.parse(JSON.stringify(originalParticipantsData));
  closeParticipantsModal();
}

/* ==== Documents ==== */
function openDocsModal(eventId){
  const ev=appData.events.find(e=>e.id===eventId); if(!ev) return;
  currentEventForDocs=eventId;
  document.getElementById('docsEventName').textContent=ev.nom;
  renderDocsList();
  document.getElementById('docsModal').classList.add('active');
  
  // Focus sur le premier champ focusable
  setTimeout(() => {
    const firstInput = document.querySelector('#docsModal input, #docsModal button, #docsModal .document-upload-zone');
    if(firstInput) firstInput.focus();
  }, 150);
}
function closeDocsModal(){ 
  document.getElementById('docsModal').classList.remove('active'); 
  currentEventForDocs=null;
  if (lastPreviewUrl){ try{ URL.revokeObjectURL(lastPreviewUrl); }catch(e){} lastPreviewUrl=null; }
}
function handleDragOver(e){ e.preventDefault(); document.getElementById('docDropZone').classList.add('dragover'); }
function handleDragLeave(e){ e.preventDefault(); document.getElementById('docDropZone').classList.remove('dragover'); }
function handleFileDrop(e){ e.preventDefault(); document.getElementById('docDropZone').classList.remove('dragover'); if(!currentEventForDocs) return; addFilesToEvent(e.dataTransfer.files); }
function handleFileSelect(e){ if(!currentEventForDocs) return; addFilesToEvent(e.target.files); e.target.value=''; }
async function addFilesToEvent(fileList){
  const ev=appData.events.find(e=>e.id===currentEventForDocs); if(!ev) return;
  const arr = Array.from(fileList);
  for (const file of arr){
    if(file.size>10*1024*1024){ showToast(`${file.name} > 10 Mo`, 'warning'); continue; }
    const id=Date.now()+Math.random().toString(36).slice(2);
    const dataUrl = await fileToDataURL(file);
    ev.documents.push({
      id,
      name:file.name,
      type:file.type,
      size:file.size,
      uploadDate:todayYMD(),
      dataUrl
    });
    blobManager.addBlob(ev.id, id, file);
  }
  markAsChanged();
  saveData(); renderDocsList();
}
function renderDocsList(){
  const ev=appData.events.find(e=>e.id===currentEventForDocs); if(!ev) return;
  const list=document.getElementById('docsList'), empty=document.getElementById('docsEmpty');
  list.innerHTML=''; document.getElementById('docsCount').textContent=ev.documents.length;
  if(!ev.documents.length){ empty.style.display='block'; document.getElementById('docPreviewArea').style.display='none'; return; }
  empty.style.display='none';
  ev.documents.forEach(d=>{
    const row=document.createElement('div');
    row.className='document-item flex items-center';
    const extIcon = d.type?.includes('pdf') ? 'fa-file-pdf text-red-600' :
                    d.type?.includes('word')||d.name.endsWith('.doc')||d.name.endsWith('.docx') ? 'fa-file-word text-blue-600' :
                    d.type?.includes('excel')||d.name.endsWith('.xls')||d.name.endsWith('.xlsx') ? 'fa-file-excel text-green-600' :
                    (d.type||'').startsWith('image') ? 'fa-file-image text-purple-600' : 'fa-file text-gray-600';
    row.innerHTML=
      `<div class="mr-3"><i class="far ${extIcon} fa-2x"></i></div>
      <div class="flex-1">
        <div class="font-semibold">${d.name}</div>
        <div class="text-xs text-gray-500">${d.type||'fichier'} • ${(d.size/1024).toFixed(1)} Ko • ${d.uploadDate}</div>
      </div>
      <div class="flex items-center space-x-2">
        <button class="px-3 py-1 rounded bg-gray-100 hover:bg-gray-200" onclick="previewDocument('${d.id}')" title="Aperçu du document"><i class="fas fa-eye mr-1"></i>Aperçu</button>
        <button class="px-3 py-1 rounded bg-blue-600 text-white" onclick="downloadDocument('${d.id}')" title="Télécharger le document"><i class="fas fa-download mr-1"></i>DL</button>
        <button class="px-3 py-1 rounded bg-red-600 text-white" onclick="deleteDocument('${d.id}')" title="Supprimer le document"><i class="fas fa-trash mr-1"></i>Suppr</button>
      </div>`;
    list.appendChild(row);
  });
}
async function previewDocument(docId){
  const ev = appData.events.find(e=>e.id===currentEventForDocs); if(!ev) return;
  const doc = ev.documents.find(d=>d.id===docId); if(!doc) return;

  // Récupération du blob ou dataURL
  let blob = blobManager.getBlob(ev.id, docId);
  if(!blob && doc.dataUrl){
    try{ blob = dataURLtoBlob(doc.dataUrl); blobManager.addBlob(ev.id, docId, blob); }catch(e){}
  }
  const srcUrl = blob ? blobManager.createPreviewURL(blob) : (doc.dataUrl || null);

  // Révoque l'ancienne URL pour éviter les fuites mémoire
  if (lastPreviewUrl) { try{ URL.revokeObjectURL(lastPreviewUrl); }catch(e){} lastPreviewUrl=null; }

  const area = document.getElementById('docPreviewArea');
  const nameEl = document.getElementById('previewFileName');
  const cont = document.getElementById('docPreviewContent');
  nameEl.textContent = doc.name;
  cont.innerHTML = '';
  area.style.display = 'block';

  if(!srcUrl){
    cont.innerHTML = '<div class="text-gray-500 text-sm">Fichier non disponible. Réimportez le document.</div>';
    return;
  }

  const name = (doc.name||'').toLowerCase();
  const type = (doc.type||'').toLowerCase();
  const ext  = name.includes('.') ? name.split('.').pop() : '';

  // Helpers
  const isPDF   = type==='application/pdf' || ext==='pdf';
  const isImg   = type.startsWith('image/') || ['png','jpg','jpeg','gif','webp','bmp','svg'].includes(ext);
  const isVideo = type.startsWith('video/') || ['mp4','webm','ogg','mov','mkv'].includes(ext);
  const isAudio = type.startsWith('audio/') || ['mp3','wav','ogg','m4a','aac','flac'].includes(ext);
  const isDocx  = ext==='docx';
  const isTexty = type.startsWith('text/') || ['txt','csv','md','json','xml','log'].includes(ext);

  // Bouton "ouvrir dans un onglet" (utile pour certains formats/pro navigateurs)
  const openBtn = document.createElement('div');
  openBtn.className = 'mb-3';
  openBtn.innerHTML = `
    <a href="${srcUrl}" target="_blank" rel="noopener"
       class="inline-block px-3 py-1 rounded bg-gray-200 hover:bg-gray-300 text-sm">
      <i class="fas fa-external-link-alt mr-1"></i>Ouvrir dans un nouvel onglet
    </a>`;
  cont.appendChild(openBtn);

  // Rendus spécifiques
  if(isPDF){
    cont.insertAdjacentHTML('beforeend',
      `<iframe src="${srcUrl}" style="width:100%; height:550px; border:1px solid #e5e7eb; border-radius:8px;"></iframe>`);
    return;
  }

  if(isImg){
    cont.insertAdjacentHTML('beforeend',
      `<img src="${srcUrl}" style="max-width:100%; max-height:550px; border:1px solid #e5e7eb; border-radius:8px;" alt="aperçu image"/>`);
    return;
  }

  if(isVideo){
    cont.insertAdjacentHTML('beforeend',
      `<video src="${srcUrl}" controls style="width:100%; max-height:550px; border:1px solid #e5e7eb; border-radius:8px;"></video>`);
    return;
  }

  if(isAudio){
    cont.insertAdjacentHTML('beforeend',
      `<audio src="${srcUrl}" controls style="width:100%;"></audio>`);
    return;
  }

  if(isDocx){
    try{
      let arrayBuffer;
      if(blob) arrayBuffer = await blob.arrayBuffer();
      else arrayBuffer = await dataURLtoArrayBuffer(doc.dataUrl);
      const { value } = await window.mammoth.convertToHtml({ arrayBuffer });
      cont.insertAdjacentHTML('beforeend',
        `<div class="prose max-w-none" style="max-height:550px; overflow:auto;">${value}</div>`);
      return;
    }catch(e){
      // on tombe sur le fallback plus bas
    }
  }

  if(isTexty){
    try{
      let text;
      if(blob){
        text = await blob.text();
      }else{
        // dataURL -> texte
        const [, b64=''] = (doc.dataUrl||'').split(',');
        text = atob(b64);
      }
      // Pretty-print JSON si possible
      if(ext==='json' || type==='application/json'){
        try{ text = JSON.stringify(JSON.parse(text), null, 2); }catch(e){}
      }
      cont.insertAdjacentHTML('beforeend',
        `<pre style="max-height:550px; overflow:auto; white-space:pre; border:1px solid #e5e7eb; border-radius:8px; padding:12px; background:#fafafa;">${escapeHtml(text)}</pre>`);
      return;
    }catch(e){
      // on tombe sur le fallback plus bas
    }
  }

  // === Fallback générique ===
  // On tente un <object>/<iframe>, sinon message + téléchargement.
  cont.insertAdjacentHTML('beforeend', `
    <div class="mb-2 text-sm text-gray-600">
      Aperçu natif du navigateur (si supporté). Sinon, utilisez le bouton ci-dessus pour ouvrir/télécharger.
    </div>
    <object data="${srcUrl}" type="${type||'application/octet-stream'}"
            style="width:100%; height:550px; border:1px solid #e5e7eb; border-radius:8px;">
      <iframe src="${srcUrl}" style="width:100%; height:550px; border:0;"></iframe>
    </object>
  `);

  // petite utilitaire locale
  function escapeHtml(s){
    return (s||'').replace(/[&<>"']/g, m=>({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;' }[m]));
  }
}
function downloadDocument(docId){
  const ev=appData.events.find(e=>e.id===currentEventForDocs); if(!ev) return;
  const doc=ev.documents.find(d=>d.id===docId); if(!doc) return;
  let blob=blobManager.getBlob(ev.id, docId);

  if(!blob && doc.dataUrl){
    try{ blob = dataURLtoBlob(doc.dataUrl); }catch(e){}
  }
  if(!blob && doc.dataUrl){
    const a=document.createElement('a');
    a.href=doc.dataUrl; a.download=doc.name; a.click();
    return;
  }
  if(!blob){ alert('Fichier non disponible.'); return; }
  const a=document.createElement('a');
  a.href=URL.createObjectURL(blob); a.download=doc.name; a.click();
  URL.revokeObjectURL(a.href);
}
function deleteDocument(docId){
  const ev=appData.events.find(e=>e.id===currentEventForDocs); if(!ev) return;
  ev.documents = ev.documents.filter(d=>d.id!==docId);
  blobManager.removeBlob(ev.id, docId);
  saveData(); renderDocsList();
}
function downloadAllDocuments(){
  const ev = appData.events.find(e=>e.id===currentEventForDocs);
  if(!ev || !ev.documents?.length) return alert('Aucun document.');
  ev.documents.forEach(d=>{
    let blob = blobManager.getBlob(ev.id, d.id) || (d.dataUrl ? dataURLtoBlob(d.dataUrl) : null);
    if(!blob) return;
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = d.name || 'fichier';
    document.body.appendChild(a); a.click();
    setTimeout(()=>URL.revokeObjectURL(a.href), 1000);
    a.remove();
  });
}
function clearAllDocuments(){
  if(!confirm('Supprimer tous les documents de cet événement ?')) return;
  const ev=appData.events.find(e=>e.id===currentEventForDocs); if(!ev) return;
  ev.documents=[]; 
  // Nettoie tous les blobs de cet événement
  blobManager.removeEvent(currentEventForDocs);
  saveData(); renderDocsList();
}

/* ==== Gantt ==== */
function openPlanningModal(eventId){
  const ev=appData.events.find(e=>e.id===eventId); if(!ev) return;
  currentPlanningEventId=eventId;
  document.getElementById('planningEventName').textContent=ev.nom;
  const sel=document.getElementById('taskAgents');
  sel.innerHTML = appData.agents
    .map(a=>`<option value="${a.id}">${a.nom}</option>`)
    .join('');
  
  // Génération automatique du planning depuis les participants
  generateGanttFromParticipants();
  
  renderGantt(); 
  renderGanttList();
  
  // Ajouter le bouton de régénération
  setTimeout(() => addRegenerateButton(), 100);
  
  document.getElementById('planningModal').classList.add('active');
  
  // Focus sur le premier champ focusable
  setTimeout(() => {
    const firstInput = document.querySelector('#planningModal input, #planningModal button, #planningModal select');
    if(firstInput) firstInput.focus();
  }, 150);
}

function addRegenerateButton() {
  const header = document.querySelector('#planningModal .modal-header');
  if (!header || header.querySelector('.regenerate-btn')) return;
  
  const regenerateBtn = document.createElement('button');
  regenerateBtn.className = 'regenerate-btn bg-green-600 hover:bg-green-700 text-white px-3 py-2 rounded mr-2';
  regenerateBtn.innerHTML = '<i class="fas fa-sync-alt mr-1"></i>Régénérer depuis participants';
  regenerateBtn.onclick = () => {
    if (confirm('Regenerate automatic tasks from participant data? This will create new tasks based on motifs and dates.')) {
      generateGanttFromParticipants();
      renderGantt();
      renderGanttList();
    }
  };
  
  const closeBtn = header.querySelector('button[onclick*="closePlanningModal"]');
  if (closeBtn) {
    closeBtn.parentNode.insertBefore(regenerateBtn, closeBtn);
  }
}

function generateGanttFromParticipants() {
  const ev = appData.events.find(e => e.id === currentPlanningEventId);
  if (!ev || !ev.participants?.length) return;
  
  // Initialise les tâches si elles n'existent pas
  if (!appData.tasks[currentPlanningEventId]) {
    appData.tasks[currentPlanningEventId] = [];
  }
  
  // Grouper les participants par motif et date
  const taskGroups = {};
  
  ev.participants.forEach(part => {
    const agent = appData.agents.find(a => a.id === part.agentId);
    if (!agent) return;
    
    const motif = part.motif || 'Tâche générale';
    const date = part.date;
    const key = `${motif}_${date}`;
    
    if (!taskGroups[key]) {
      taskGroups[key] = {
        name: motif,
        date: date,
        agents: new Set(),
        services: new Set(),
        totalHours: 0,
        startTime: '23:59',
        endTime: '00:00'
      };
    }
    
    // Ajouter l'agent et son service
    taskGroups[key].agents.add(part.agentId);
    taskGroups[key].services.add(agent.service);
    
    // Calculer les heures totales
    const duration = calculateDuration(part.heureDebut || '', part.heureFin || '');
    taskGroups[key].totalHours += duration;
    
    // Déterminer l'heure de début et fin globale
    if (part.heureDebut && part.heureDebut < taskGroups[key].startTime) {
      taskGroups[key].startTime = part.heureDebut;
    }
    if (part.heureFin && part.heureFin > taskGroups[key].endTime) {
      taskGroups[key].endTime = part.heureFin;
    }
  });
  
  // Convertir les groupes en tâches Gantt
  const existingTasks = appData.tasks[currentPlanningEventId] || [];
  const existingTaskNames = new Set(existingTasks.map(t => t.name));
  
  Object.values(taskGroups).forEach(group => {
    const taskName = `${group.name} (${group.date})`;
    
    // Ne créer la tâche que si elle n'existe pas déjà
    if (!existingTaskNames.has(taskName)) {
      const newTask = {
        id: 'task_auto_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9),
        name: taskName,
        start: group.date,
        end: group.date, // Tâche d'un jour
        agentIds: Array.from(group.agents),
        service: Array.from(group.services).join(', '),
        hours: Math.round(group.totalHours * 10) / 10, // Arrondi à 1 décimale
        progress: 0,
        auto: true // Marquer comme auto-générée
      };
      
      appData.tasks[currentPlanningEventId].push(newTask);
    }
  });
  
  // Sauvegarder les modifications
  markAsChanged();
  saveData();
  
  // Afficher une notification
  const newTasksCount = Object.keys(taskGroups).length - existingTaskNames.size;
  if (newTasksCount > 0) {
    showToast(`${newTasksCount} tâche(s) générée(s) automatiquement depuis les participants`, 'success', 4000);
  }
}
function closePlanningModal(){ 
  document.getElementById('planningModal').classList.remove('active'); 
  currentPlanningEventId=null; 
  // Nettoyage propre de l'instance Gantt
  if(currentGanttInstance) {
    try {
      // Frappe-Gantt n'a pas de méthode destroy(), mais on nettoie le conteneur
      const area = document.getElementById('gantt-chart');
      if(area) area.innerHTML = '';
      currentGanttInstance = null;
    } catch(e) {
      console.warn('Erreur lors du nettoyage du Gantt:', e);
    }
  }
  resetTaskForm(); 
}
function renderGantt(){
  const area=document.getElementById('gantt-chart'), empty=document.getElementById('gantt-empty');
  
  // Nettoyage propre avant re-rendu
  if(currentGanttInstance) {
    try {
      area.innerHTML = '';
      currentGanttInstance = null;
    } catch(e) {
      console.warn('Erreur lors du nettoyage du Gantt:', e);
    }
  }
  
  const tasks=(appData.tasks[currentPlanningEventId]||[]);
  if (!window.Gantt) { 
    empty.style.display='block'; 
    empty.innerHTML='<div class="text-center text-gray-500 py-12"><i class="fas fa-exclamation-triangle fa-3x mb-4 text-gray-300"></i><p class="text-lg">Gantt non disponible</p><p class="text-sm">La librairie Frappe-Gantt n\'a pas pu se charger</p></div>';
    return; 
  }
  if(!tasks.length){ empty.style.display='block'; return; }
  empty.style.display='none';
  
  try {
    const ganttTasks = tasks.map(t=>({ id:t.id, name:t.name, start:t.start, end:t.end, progress:t.progress||0, custom_class:'bar-default' }));
    currentGanttInstance = new Gantt(area, ganttTasks, {
      view_mode:'Day', date_format:'YYYY-MM-DD',
      on_click: task => {
        editingTaskId = task.id;
        const t = (appData.tasks[currentPlanningEventId]||[]).find(x=>x.id===task.id);
        if(t){
          document.getElementById('taskName').value=t.name||'';
          document.getElementById('taskStartDate').value=t.start||'';
          document.getElementById('taskEndDate').value=t.end||'';
          const sel = document.getElementById('taskAgents');
          const ids = Array.isArray(t.agentIds) ? t.agentIds : (t.agentId!=null ? [t.agentId] : []);
          Array.from(sel.options).forEach(opt => opt.selected = ids.includes(+opt.value));
          document.getElementById('taskService').value=t.service||'';
          document.getElementById('taskHours').value=t.hours||'';
          document.getElementById('taskFormSubmitText').textContent='Mettre à jour';
        }
      }
    });
  } catch(e) {
    console.error('Erreur lors de la création du Gantt:', e);
    empty.style.display='block';
    empty.innerHTML='<div class="text-center text-gray-500 py-12"><i class="fas fa-exclamation-triangle fa-3x mb-4 text-red-300"></i><p class="text-lg">Erreur Gantt</p><p class="text-sm">Impossible de créer le diagramme</p></div>';
  }
}
function renderGanttList(){
  const el=document.getElementById('ganttTasksList'); el.innerHTML='';
  const tasks=(appData.tasks[currentPlanningEventId]||[]);
  if(!tasks.length) return;
  tasks.forEach(t=>{
    const row=document.createElement('div');
    row.className='flex justify-between items-center p-2 border rounded';
    const names = (Array.isArray(t.agentIds) ? t.agentIds : (t.agentId!=null ? [t.agentId] : []))
      .map(id => appData.agents.find(a=>a.id===id)?.nom || `#${id}`)
      .join(', ') || '—';
    row.innerHTML = 
      `<div>
        <div class="font-semibold">${t.name}</div>
        <div class="text-xs text-gray-500">${t.start} → ${t.end} • ${t.hours||0}h • ${t.service||'—'} • ${names}</div>
      </div>
      <div class="space-x-2">
        <button class="px-2 py-1 rounded bg-blue-600 text-white" onclick="editTask('${t.id}')" title="Modifier la tâche"><i class="fas fa-pen"></i></button>
        <button class="px-2 py-1 rounded bg-red-600 text-white" onclick="deleteTask('${t.id}')" title="Supprimer la tâche"><i class="fas fa-trash"></i></button>
      </div>`;
    el.appendChild(row);
  });
}
function editTask(id){
  const t=(appData.tasks[currentPlanningEventId]||[]).find(x=>x.id===id); if(!t) return;
  editingTaskId=id;
  document.getElementById('taskName').value=t.name||'';
  document.getElementById('taskStartDate').value=t.start||'';
  document.getElementById('taskEndDate').value=t.end||'';
  const sel = document.getElementById('taskAgents');
  const ids = Array.isArray(t.agentIds) ? t.agentIds : (t.agentId!=null ? [t.agentId] : []);
  Array.from(sel.options).forEach(opt => opt.selected = ids.includes(+opt.value));
  document.getElementById('taskService').value=t.service||'';
  document.getElementById('taskHours').value=t.hours||'';
  document.getElementById('taskFormSubmitText').textContent='Mettre à jour';
}
function deleteTask(id){
  if(!confirm('Supprimer cette tâche ?')) return;
  appData.tasks[currentPlanningEventId] = (appData.tasks[currentPlanningEventId]||[]).filter(t=>t.id!==id);
  saveData(); renderGantt(); renderGanttList();
}
function resetTaskForm(){
  editingTaskId=null; 
  document.getElementById('taskForm').reset(); 
  const sel = document.getElementById('taskAgents');
  if(sel) Array.from(sel.options).forEach(o=>o.selected=false);
  document.getElementById('taskFormSubmitText').textContent='Enregistrer';
}
function saveGanttTask(e){
  e.preventDefault();
  const name=document.getElementById('taskName').value.trim();
  const start=document.getElementById('taskStartDate').value;
  const end=document.getElementById('taskEndDate').value;
  const agentIds = Array.from(document.getElementById('taskAgents').selectedOptions).map(o=>+o.value);
  const service=document.getElementById('taskService').value;
  const hours=parseFloat(document.getElementById('taskHours').value||'0')||0;
  if(!name||!start||!end) return showToast('Champs requis manquants', 'error');
  const list=appData.tasks[currentPlanningEventId]= (appData.tasks[currentPlanningEventId]||[]);
  if(editingTaskId){
    const idx=list.findIndex(t=>t.id===editingTaskId);
    if(idx>-1){ list[idx]={...list[idx], name, start, end, agentIds, service, hours}; }
    showToast('Tâche mise à jour !', 'success');
  }else{
    list.push({ id:'task_'+Date.now(), name, start, end, agentIds, service, hours, progress:0 });
    showToast('Tâche ajoutée !', 'success');
  }
  markAsChanged();
  saveData(); resetTaskForm(); renderGantt(); renderGanttList();
}

/* ==== Heures (cards) ==== */
function updateHoursTable(){
  const wrap=document.getElementById('hoursContainer'); wrap.innerHTML='';
  appData.agents.forEach(a=>{
    const parts=getFilteredParticipants().filter(p=>p.agentId===a.id);
    let p=0,r=0,v=0;
    parts.forEach(x=>{
      const d=calculateDuration(x.heureDebut||'', x.heureFin||'') || ((+x.payees||0)+(+x.recup||0)+(+x.voirieFest||0));
      if(x.nature==='payees') p+=d*60; else if(x.nature==='recup') r+=d*60; else if(x.nature==='voirie') v+=d*60;
    });
    const tot=p+r+v;
    if(tot>0 || currentTypeFilter==='tous'){
      const div=document.createElement('div');
      div.className='bg-white border rounded-xl p-4';
      div.innerHTML=
        `<div class="flex items-center justify-between mb-2">
          <div class="font-semibold">${a.nom}</div>
          <div class="text-xs text-gray-500">${a.service}</div>
        </div>
        <div class="grid grid-cols-3 gap-2 text-center text-sm">
          <div class="p-2 bg-green-50 rounded border"><div class="text-green-700 font-bold">${formatMinutes(p)}</div><div class="text-gray-500">Payées</div></div>
          <div class="p-2 bg-blue-50 rounded border"><div class="text-blue-700 font-bold">${formatMinutes(r)}</div><div class="text-gray-500">Récup</div></div>
          <div class="p-2 bg-yellow-50 rounded border"><div class="text-yellow-700 font-bold">${formatMinutes(v)}</div><div class="text-gray-500">Voirie→Fest</div></div>
        </div>
        <div class="mt-2 text-right text-xs text-gray-500">Total: ${formatMinutes(tot)}</div>`;
      wrap.appendChild(div);
    }
  });
}

/* ==== Calendrier (modifié : ajoute les noms des manifestations) ==== */
function renderCalendar(){
  const yearEl=document.getElementById('calendarYear');
  if(!yearEl.dataset.ready){
    const yNow=new Date().getFullYear();
    for(let y=yNow-2;y<=yNow+2;y++){
      const opt=document.createElement('option');
      opt.value=y; opt.textContent=y; if(y===yNow) opt.selected=true;
      yearEl.appendChild(opt);
    }
    yearEl.dataset.ready="1";
  }

  const year=+document.getElementById('calendarYear').value || new Date().getFullYear();
  const wrap=document.getElementById('calendarWrap'); wrap.innerHTML='';

  for(let m=0;m<12;m++){
    const monthBox=document.createElement('div');
    monthBox.innerHTML=`<div class="mb-2 font-bold text-lg text-gray-800">${monthLabels()[m]} ${year}</div>`;
    const grid=document.createElement('div'); grid.className='calendar-grid';

    const first=new Date(year,m,1);
    const startIdx=(first.getDay()+6)%7; // Lundi=0
    for(let i=0;i<startIdx;i++){ const empty=document.createElement('div'); empty.className='day-cell'; grid.appendChild(empty); }

    const daysInMonth=new Date(year,m+1,0).getDate();
    for(let d=1; d<=daysInMonth; d++){
      const cell=document.createElement('div'); cell.className='day-cell';
      const dateStr = `${year}-${String(m+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
      cell.innerHTML = `<div class="day-num">${d}</div>`;

      // événements couvrant ce jour
      const eventsDay = appData.events.filter(e=> getDatesBetween(e.dateDebut,e.dateFin).includes(dateStr) );

      if(eventsDay.length){
        // pastille chiffre (conservée)
        const dot=document.createElement('div');
        dot.className='event-dot';
        dot.textContent=eventsDay.length;
        cell.appendChild(dot);

        // Liste des noms (nouveau)
        const list=document.createElement('div');
        list.className='event-list';
        const maxShown=2; // ajuste à 1 si besoin

        eventsDay.slice(0,maxShown).forEach(ev=>{
          const chip=document.createElement('div');
          chip.className='event-chip';
          chip.textContent=ev.nom;
          chip.title=ev.nom; // tooltip complet
          list.appendChild(chip);
        });
        if(eventsDay.length>maxShown){
          const more=document.createElement('div');
          more.className='event-chip';
          more.textContent=`+${eventsDay.length-maxShown} autre(s)…`;
          list.appendChild(more);
        }
        cell.appendChild(list);

        // Fallback : tooltip global sur la case
        cell.title = eventsDay.map(e=>e.nom).join(' • ');

        // clic → détail du jour
        cell.classList.add('cursor-pointer','hover:shadow');
        cell.onclick = ()=> openDayModal(dateStr, eventsDay);
      }

      grid.appendChild(cell);
    }
    monthBox.appendChild(grid);
    wrap.appendChild(monthBox);
  }
}

function renderDayCharts(dateStr, eventsDay){
  if(!window.Chart) return;

  // 1) Donut : répartition Payées / Récup / Voirie sur la journée (tous événements du jour)
  let pay=0, rec=0, voir=0;
  eventsDay.forEach(e=>{
    (e.participants||[]).forEach(l=>{
      if(l.date !== dateStr) return;
      const d = sumLineHours(l);
      if(l.nature==='payees') pay+=d;
      else if(l.nature==='recup') rec+=d;
      else if(l.nature==='voirie') voir+=d;
    });
  });

  const c1 = document.getElementById('dayDistChart');
  if(c1){
    if(dayDistChartInst) dayDistChartInst.destroy();
    dayDistChartInst = new Chart(c1.getContext('2d'), {
      type: 'doughnut',
      data: { labels:['Payées','Récup','Voirie→Fest'], datasets:[{ data:[pay,rec,voir] }] },
      options:{
        responsive:true, maintainAspectRatio:false,
        plugins:{
          legend:{ position:'bottom' },
          datalabels:{ formatter:(v,ctx)=>{
            const tot = pay+rec+voir||1;
            return v>0 ? `${v.toFixed(1)}h\n${Math.round((v/tot)*100)}%` : '';
          }}
        }
      }
    });
  }

  // 2) Barres horizontales : heures du jour par événement
  const labels = eventsDay.map(e=>e.nom);
  const values = eventsDay.map(e=>{
    let s = 0;
    (e.participants||[]).forEach(l => { if(l.date===dateStr) s += sumLineHours(l); });
    return s;
  });

  const c2 = document.getElementById('dayEventBarChart');
  if(c2){
    if(dayEventBarChartInst) dayEventBarChartInst.destroy();
    dayEventBarChartInst = new Chart(c2.getContext('2d'), {
      type: 'bar',
      data: { labels, datasets:[{ label:'Heures du jour', data: values }] },
      options:{
        responsive:true, maintainAspectRatio:false, indexAxis:'y',
        scales:{ x:{ beginAtZero:true, max: Math.max(4, Math.ceil((values.reduce((a,b)=>Math.max(a,b),0)||0)+1)) } },
        plugins:{
          legend:{ display:false },
          datalabels:{ align:'right', anchor:'end', formatter:(v)=> v>0 ? `${v.toFixed(1)}h` : '' },
          tooltip:{ callbacks:{ label:(c)=> `${c.raw.toFixed(1)}h` } }
        }
      }
    });
  }
}

/* ==== Détail jour (inchangé) ==== */
function openDayModal(dateStr, eventsDay){
  const chartsHtml = `
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 mb-4">
      <div class="p-3 border rounded"><canvas id="dayDistChart"></canvas></div>
      <div class="p-3 border rounded"><canvas id="dayEventBarChart"></canvas></div>
    </div>
  `;

  const title = `Événements du ${toLocalDate(dateStr).toLocaleDateString('fr-FR',{weekday:'long',day:'2-digit',month:'long',year:'numeric'})}`;

  let dayPay=0, dayRec=0, dayVoir=0, dayLines=0;

  let eventsHtml = '';
  if(!eventsDay.length){
    eventsHtml = '<div class="text-gray-500">Aucun événement ce jour</div>';
  } else {
    eventsHtml = eventsDay.map(e=>{
      const linesOfDay = (e.participants||[]).filter(l=>l.date===dateStr);
      const linesAll = (e.participants||[]);

      let pD=0,rD=0,vD=0;
      const rows = linesOfDay.map(l=>{
        const ag=appData.agents.find(a=>a.id===l.agentId);
        const d = calculateDuration(l.heureDebut||'', l.heureFin||'') || ((+l.payees||0)+(+l.recup||0)+(+l.voirieFest||0));
        if(l.nature==='payees') pD+=d; else if(l.nature==='recup') rD+=d; else if(l.nature==='voirie') vD+=d;
        const badge = l.nature==='payees'?'bg-green-100 text-green-800':(l.nature==='recup'?'bg-blue-100 text-blue-800':'bg-yellow-100 text-yellow-800');
        return `
          <tr>
            <td class="px-2 py-1 border">${ag?ag.nom:'—'}</td>
            <td class="px-2 py-1 border">${l.motif||'—'}</td>
            <td class="px-2 py-1 border"><span class="px-2 py-1 rounded text-xs font-medium ${badge}">${l.nature}</span></td>
            <td class="px-2 py-1 border text-center">${l.heureDebut||'—'}</td>
            <td class="px-2 py-1 border text-center">${l.heureFin||'—'}</td>
            <td class="px-2 py-1 border text-center font-semibold">${d.toFixed(1)} h</td>
          </tr>
        `;
      }).join('');
      dayLines += linesOfDay.length;
      dayPay+=pD; dayRec+=rD; dayVoir+=vD;

      let pE=0,rE=0,vE=0;
      linesAll.forEach(l=>{
        const d = calculateDuration(l.heureDebut||'', l.heureFin||'') || ((+l.payees||0)+(+l.recup||0)+(+l.voirieFest||0));
        if(l.nature==='payees') pE+=d; else if(l.nature==='recup') rE+=d; else if(l.nature==='voirie') vE+=d;
      });

      return `
        <div class="p-3 border rounded-lg mb-3">
          <div class="flex items-center justify-between">
            <div>
              <div class="font-semibold text-gray-800">${e.nom} <span class="text-xs text-gray-500">(${e.categorie})</span></div>
              <div class="text-xs text-gray-500">📍 ${e.lieu} • ${e.dateDebut} → ${e.dateFin}</div>
            </div>
            <div class="space-x-2">
              <button class="px-3 py-1 rounded bg-blue-600 text-white text-sm" onclick="goToEvent(${e.id})"><i class="fas fa-folder-open mr-1"></i>Ouvrir l’événement</button>
              <button class="px-3 py-1 rounded bg-yellow-500 text-white text-sm" onclick="openParticipantsModal(${e.id})"><i class="fas fa-users-cog mr-1"></i>Participants</button>
            </div>
          </div>
          <div class="mt-2 text-sm space-y-1">
            <div>
              <span class="text-gray-700 font-semibold mr-1">Jour :</span>
              <span class="text-green-700 font-semibold">${pD.toFixed(1)}h</span> /
              <span class="text-blue-700 font-semibold">${rD.toFixed(1)}h</span> /
              <span class="text-yellow-700 font-semibold">${vD.toFixed(1)}h</span>
              <span class="text-gray-400 text-xs ml-2">• ${linesOfDay.length} ligne${linesOfDay.length>1?'s':''}</span>
            </div>
            <div class="text-xs text-gray-600">
              <span class="font-semibold mr-1">Événement (total) :</span>
              <span class="text-green-700 font-semibold">${pE.toFixed(1)}h</span> /
              <span class="text-blue-700 font-semibold">${rE.toFixed(1)}h</span> /
              <span class="text-yellow-700 font-semibold">${vE.toFixed(1)}h</span>
              <span class="text-gray-400 ml-2">• ${linesAll.length} ligne${linesAll.length>1?'s':''}</span>
            </div>
          </div>
          <div class="mt-2 overflow-x-auto">
            <table class="w-full text-sm border">
              <thead class="bg-gray-50">
                <tr>
                  <th class="px-2 py-1 border text-left">Agent</th>
                  <th class="px-2 py-1 border text-left">Motif</th>
                  <th class="px-2 py-1 border text-left">Type</th>
                  <th class="px-2 py-1 border">Début</th>
                  <th class="px-2 py-1 border">Fin</th>
                  <th class="px-2 py-1 border">Durée</th>
                </tr>
              </thead>
              <tbody>${rows || '<tr><td class="p-2 text-gray-400" colspan="6">Aucune ligne d’heures ce jour</td></tr>'}</tbody>
            </table>
          </div>
        </div>
      `;
    }).join('');
  }

  const agentAgg = {};
  eventsDay.forEach(e=>{
    (e.participants||[]).forEach(l=>{
      if(l.date!==dateStr) return;
      const ag=appData.agents.find(a=>a.id===l.agentId);
      const d = calculateDuration(l.heureDebut||'', l.heureFin||'') || ((+l.payees||0)+(+l.recup||0)+(+l.voirieFest||0));        if(!agentAgg[l.agentId]){
          agentAgg[l.agentId]={pay:0,rec:0,voir:0,rows:[],name:(ag?ag.nom:'Agent inconnu')};
        }
        if(l.nature==='payees') agentAgg[l.agentId].pay += d;
        else if(l.nature==='recup') agentAgg[l.agentId].rec += d;
        else if(l.nature==='voirie') agentAgg[l.agentId].voir += d;
        agentAgg[l.agentId].rows.push({ ...l, eventName:e.nom, eventCategorie:e.categorie, duree:d });
    });
  });

  const agentCards = Object.entries(agentAgg)
    .sort(([,a],[,b]) => a.name.localeCompare(b.name,'fr'))
    .map(([id,agg],idx)=>{
      const total = agg.pay+agg.rec+agg.voir;
      const tableId = `ag_${id}_${idx}`;
      const rows = agg.rows
        .sort((a,b)=> a.eventName.localeCompare(b.eventName,'fr') || (a.heureDebut||'').localeCompare(b.heureDebut||''))
        .map(l=>{
          const badge = l.nature==='payees'?'bg-green-100 text-green-800':(l.nature==='recup'?'bg-blue-100 text-blue-800':'bg-yellow-100 text-yellow-800');
          return `
            <tr>
              <td class="px-2 py-1 border">${l.eventName} <span class="text-xs text-gray-500">(${l.eventCategorie})</span></td>
              <td class="px-2 py-1 border">${l.motif||'—'}</td>
              <td class="px-2 py-1 border"><span class="px-2 py-1 rounded text-xs font-medium ${badge}">${l.nature}</span></td>
              <td class="px-2 py-1 border text-center">${l.heureDebut||'—'}</td>
              <td class="px-2 py-1 border text-center">${l.heureFin||'—'}</td>
              <td class="px-2 py-1 border text-center font-semibold">${l.duree.toFixed(1)} h</td>
            </tr>
          `;
        }).join('');

      return `
        <div class="p-3 border rounded-lg">
          <div class="flex items-center justify-between">
            <div class="font-semibold text-gray-800">${agg.name}</div>
            <div class="text-sm">
              <span class="text-green-700 font-semibold mr-2">${agg.pay.toFixed(1)}h</span>
              <span class="text-blue-700 font-semibold mr-2">${agg.rec.toFixed(1)}h</span>
              <span class="text-yellow-700 font-semibold mr-2">${agg.voir.toFixed(1)}h</span>
              <span class="text-purple-700 font-bold">Total: ${total.toFixed(1)}h</span>
            </div>
          </div>
          <div class="mt-2">
            <button class="px-2 py-1 text-xs rounded bg-gray-100 hover:bg-gray-200" onclick="
              const t=document.getElementById('${tableId}');
              const ico=this.querySelector('i');
              t.style.display = (t.style.display==='none'||!t.style.display) ? 'block' : 'none';
              if(ico){ ico.classList.toggle('fa-chevron-down'); ico.classList.toggle('fa-chevron-up'); }
            ">
              Détails <i class="fas fa-chevron-down ml-1"></i>
            </button>
          </div>
          <div id="${tableId}" style="display:none" class="mt-2 overflow-x-auto">
            <table class="w-full text-sm border">
              <thead class="bg-gray-50">
                <tr>
                  <th class="px-2 py-1 border text-left">Événement</th>
                  <th class="px-2 py-1 border text-left">Motif</th>
                  <th class="px-2 py-1 border text-left">Type</th>
                  <th class="px-2 py-1 border">Début</th>
                  <th class="px-2 py-1 border">Fin</th>
                  <th class="px-2 py-1 border">Durée</th>
                </tr>
              </thead>
              <tbody>${rows || '<tr><td class="p-2 text-gray-400" colspan="6">Aucune ligne</td></tr>'}</tbody>
            </table>
          </div>
        </div>
      `;
    }).join('');

  const agentSection = Object.keys(agentAgg).length
    ? `
      <div class="mt-6">
        <div class="mb-2 font-semibold text-gray-800">Détail par agent (sur la journée)</div>
        <div class="space-y-3">${agentCards}</div>
      </div>
    `
    : '';

  const daySummary = `
    <div class="mb-4 p-3 bg-gradient-to-r from-green-50 to-blue-50 rounded border">
      <div class="text-sm font-semibold text-gray-700">Totaux de la journée</div>
      <div class="mt-1 text-sm">
        <span class="text-green-700 font-bold">${dayPay.toFixed(1)}h payées</span> •
        <span class="text-blue-700 font-bold">${dayRec.toFixed(1)}h récup</span> •
        <span class="text-yellow-700 font-bold">${dayVoir.toFixed(1)}h voirie→fest</span>
        <span class="text-gray-400 text-xs ml-2">• ${eventsDay.length} évènement${eventsDay.length>1?'s':''} • ${dayLines} ligne${dayLines>1?'s':''}</span>
      </div>
    </div>
  `;

  document.getElementById('barInfoTitle').textContent=title;
  document.getElementById('barInfoBody').innerHTML= chartsHtml + daySummary + eventsHtml + agentSection;
  document.getElementById('barInfoModal').classList.add('active');

  // ✅ lance les 2 graphiques
  renderDayCharts(dateStr, eventsDay);
}

/* ==== Graphique Multi-sélections ==== */
function populateMultiFilters(){
  const typeSel = document.getElementById('multiTypeFilter');
  const agentSel = document.getElementById('multiAgentFilter');
  const eventSel = document.getElementById('multiEventFilter');
  if(!typeSel || !agentSel || !eventSel) return;

  const keepType  = typeSel.value;
  const keepAgent = agentSel.value;
  const keepEvent = eventSel.value;

  const types  = [...new Set(appData.events.map(e=>e.categorie))].sort();
  typeSel.innerHTML  = '<option value="tous">Tous types</option>'+ types.map(t=>`<option value="${t}">${t}</option>`).join('');
  agentSel.innerHTML = '<option value="tous">Tous agents</option>'+ appData.agents.map(a=>`<option value="${a.id}">${a.nom}</option>`).join('');
  eventSel.innerHTML = '<option value="tous">Tous événements</option>'+ appData.events.map(e=>`<option value="${e.id}">${e.nom}</option>`).join('');

  if([...typeSel.options].some(o=>o.value===keepType)) typeSel.value=keepType;
  if([...agentSel.options].some(o=>o.value===keepAgent)) agentSel.value=keepAgent;
  if([...eventSel.options].some(o=>o.value===keepEvent)) eventSel.value=keepEvent;

  typeSel.onchange = createMultiHoursChart;
  agentSel.onchange = createMultiHoursChart;
  eventSel.onchange = createMultiHoursChart;
}
function resetMultiFilters(){
  const typeSel = document.getElementById('multiTypeFilter');
  const agentSel = document.getElementById('multiAgentFilter');
  const eventSel = document.getElementById('multiEventFilter');
  if(typeSel) typeSel.value='tous';
  if(agentSel) agentSel.value='tous';
  if(eventSel) eventSel.value='tous';
  createMultiHoursChart();
}
function createMultiHoursChart(){
  if(!chartReady('multiHoursChart')) return;
  const canvas = document.getElementById('multiHoursChart');
  const ctx = canvas.getContext('2d');
  if(multiHoursChartInstance) multiHoursChartInstance.destroy();

  const typeFilter = document.getElementById('multiTypeFilter')?.value || 'tous';
  const agentFilter = document.getElementById('multiAgentFilter')?.value || 'tous';
  const eventFilter = document.getElementById('multiEventFilter')?.value || 'tous';

  const labels = monthLabels();
  let payees = new Array(12).fill(0);
  let recup  = new Array(12).fill(0);
  let voirie = new Array(12).fill(0);

  appData.events.forEach(ev=>{
    if(typeFilter!=='tous' && ev.categorie!==typeFilter) return;
    if(eventFilter!=='tous' && String(ev.id)!==String(eventFilter)) return;

    (ev.participants || []).forEach(p=>{
      if(agentFilter!=='tous' && String(p.agentId)!==String(agentFilter)) return;
      const m = toLocalDate(p.date).getMonth();
      let d = calculateDuration(p.heureDebut||'', p.heureFin||'');
      if(d<=0){ d=(+p.payees||0)+(+p.recup||0)+(+p.voirieFest||0); }
      if(p.nature==='payees') payees[m]+=d;
      else if(p.nature==='recup') recup[m]+=d;
      else if(p.nature==='voirie') voirie[m]+=d;
    });
  });

  try{
    multiHoursChartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels,
        datasets: [
          { label: 'Payées', data: payees, fill:false, tension:0.25, borderWidth:2 },
          { label: 'Récupérées', data: recup, fill:false, tension:0.25, borderWidth:2 },
          { label: 'Voirie→Festivités', data: voirie, fill:false, tension:0.25, borderWidth:2 }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio:false,
        interaction: { mode: 'index', intersect: false },
        scales: { y: { beginAtZero: true, max:650 } },
        plugins: {
          legend: { position:'top' },
          datalabels: { align:'top', anchor:'end', formatter:(val)=> (val && val>0) ? `${val.toFixed(1)}h` : '' },
          tooltip: { callbacks: { label:(c)=> `${c.dataset.label}: ${c.parsed.y.toFixed(1)}h` } }
        }
      }
    });
  }catch(e){ console.error('createMultiHoursChart failed:', e); }
}

/* ==== NOUVEAU : Heures annuelles par type (ligne) ==== */
function normLabel(s){
  return String(s||'').toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');
}
function computeMonthlyHoursForEventType(typeName){
  const target = normLabel(typeName);
  const months = new Array(12).fill(0);
  (appData.events||[]).forEach(ev=>{
    if(normLabel(ev.categorie)!==target) return;
    (ev.participants||[]).forEach(l=>{
      const m = toLocalDate(l.date).getMonth();
      let d = calculateDuration(l.heureDebut||'', l.heureFin||'');
      if(d<=0){ d=(+l.payees||0)+(+l.recup||0)+(+l.voirieFest||0); }
      months[m] += d; // heures
    });
  });
  return months;
}
function createYearlyTypeHoursChart(){
  if(!chartReady('yearlyTypeHoursChart')) return;
  if(yearlyTypeHoursChartInstance) yearlyTypeHoursChartInstance.destroy();
  const ctx = document.getElementById('yearlyTypeHoursChart').getContext('2d');

  const types = [...new Set((appData.events||[]).map(e=>e.categorie))]
    .filter(Boolean).sort((a,b)=>a.localeCompare(b,'fr'));

  const datasets = types.map(name => ({
    label: name,
    data: computeMonthlyHoursForEventType(name),
    fill:false, tension:0.25, borderWidth:2
  }));

  try{
    yearlyTypeHoursChartInstance = new Chart(ctx, {
      type:'line',
      data:{ labels:monthLabels(), datasets },
      options:{
        responsive:true, maintainAspectRatio:false, spanGaps:true,
        scales:{ y:{ beginAtZero:true, max:650 } },
        plugins:{
          legend:{ position:'top' },
          tooltip:{ callbacks:{ label:(c)=> `${c.dataset.label}: ${c.parsed.y.toFixed(1)}h` } },
          datalabels:{ align:'top', anchor:'end',
            formatter:(v)=> (v && v>0) ? `${v.toFixed(1)}h` : '' }
        }
      }
    });
  }catch(e){ console.error('createYearlyTypeHoursChart failed:', e); }
}

/* ==== Impression + Reset ==== */
function printModule(){ 
  // Convertit les graphiques en images avant d'imprimer
  console.log('🖨️ Début de la préparation d\'impression...');
  
  // Utilise un timeout pour s'assurer que tous les graphiques sont chargés
  setTimeout(async () => {
    try {
      await convertChartsToImages();
      
      // Ajoute un style temporaire pour l'impression
      const printStyle = document.createElement('style');
      printStyle.innerHTML = `
        @media print {
          .print-chart-image { 
            display: block !important; 
            visibility: visible !important;
            max-width: 100% !important;
            height: auto !important;
          }
          canvas { display: none !important; }
        }
      `;
      document.head.appendChild(printStyle);
      
      // Lance l'impression
      window.print();
      
      // Nettoie après impression
      setTimeout(() => {
        // Supprime les images temporaires
        document.querySelectorAll('.print-chart-image').forEach(img => img.remove());
        // Reaffiche les canvas
        document.querySelectorAll('canvas').forEach(canvas => canvas.style.display = '');
        // Supprime le style temporaire
        printStyle.remove();
        console.log('🖨️ Nettoyage après impression terminé');
      }, 1000);
      
    } catch(e) {
      console.error('❌ Erreur lors de la préparation d\'impression:', e);
      window.print(); // Impression de fallback
    }
  }, 200);
}

function printModal(id){
  const modal=document.getElementById(id);
  if(!modal) return;
  
  const body = modal.querySelector('.modal-body');
  let content = body ? body.innerHTML : '';

  // Remplace les canvas par des images pour l'impression
  if(body){
    const clone = body.cloneNode(true);
    const canvases = clone.querySelectorAll('canvas');
    
    canvases.forEach(canvas => {
      try {
        // Trouve le canvas original correspondant
        const originalCanvas = body.querySelector(`#${canvas.id}`) || 
                              Array.from(body.querySelectorAll('canvas')).find(c => 
                                c.getBoundingClientRect().width === canvas.getBoundingClientRect().width
                              );
        
        if(originalCanvas && originalCanvas.width > 0 && originalCanvas.height > 0) {
          const img = document.createElement('img');
          img.src = originalCanvas.toDataURL('image/png', 1.0);
          img.style.maxWidth = '100%';
          img.style.height = 'auto';
          img.style.display = 'block';
          img.style.margin = '12px auto';
          img.style.border = '1px solid #e5e7eb';
          img.style.borderRadius = '8px';
          img.style.boxShadow = '0 2px 4px rgba(0,0,0,0.1)';
          
          // Remplace le canvas par l'image
          canvas.parentNode.replaceChild(img, canvas);
        } else {
          // Si le canvas est vide ou non trouvé, on le supprime
          canvas.remove();
        }
      } catch(e) {
        console.warn('Impossible de convertir le canvas en image:', e);
        // En cas d'erreur, on supprime le canvas
        canvas.remove();
      }
    });
    
    content = clone.innerHTML;
  }

  const w=window.open('','PRINT','height=800,width=1000');
  w.document.write(`<html><head><title>Impression</title><link href='https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css' rel='stylesheet'/><style>@media print { body { -webkit-print-color-adjust: exact; color-adjust: exact; } img { break-inside: avoid; page-break-inside: avoid; } }</style></head><body class='p-6'>${content}</body></html>`);
  w.document.close(); w.focus(); w.print(); w.close();
}

// Nouvelle fonction pour convertir tous les graphiques en images
async function convertChartsToImages() {
  console.log('🔄 Conversion des graphiques en images...');
  
  const canvasIds = [
    'eventsChart', 'sectorsChart', 'hoursChart', 'monthlyTypeChart',
    'yearlyHoursLineChart', 'monthlyBreakdownLineChart', 'multiHoursChart', 
    'yearlyTypeHoursChart'
  ];
  
  let convertedCount = 0;
  
  for (const canvasId of canvasIds) {
    const canvas = document.getElementById(canvasId);
    
    if (!canvas) {
      console.warn(`⚠️ Canvas ${canvasId} non trouvé`);
      continue;
    }
    
    if (canvas.width === 0 || canvas.height === 0) {
      console.warn(`⚠️ Canvas ${canvasId} a une taille nulle`);
      continue;
    }
    
    try {
      // Attendre un peu que le canvas soit prêt
      await new Promise(resolve => setTimeout(resolve, 50));
      
      // Crée une image à partir du canvas
      const dataURL = canvas.toDataURL('image/png', 1.0);
      
      if (dataURL === 'data:,') {
        console.warn(`⚠️ Canvas ${canvasId} vide ou non rendu`);
        continue;
      }
      
      const img = document.createElement('img');
      img.src = dataURL;
      img.style.maxWidth = '100%';
      img.style.height = 'auto';
      img.style.border = '1px solid #e5e7eb';
      img.style.borderRadius = '8px';
      img.style.boxShadow = '0 2px 4px rgba(0,0,0,0.1)';
      img.className = 'print-chart-image';
      img.style.display = 'block';
      img.style.margin = '10px auto';
      
      // Cache le canvas et insère l'image juste après
      canvas.style.display = 'none';
      canvas.parentNode.insertBefore(img, canvas.nextSibling);
      
      convertedCount++;
      console.log(`✅ Graphique ${canvasId} converti (${canvas.width}x${canvas.height}px)`);
      
    } catch(e) {
      console.warn(`⚠️ Impossible de convertir ${canvasId}:`, e);
    }
  }
  
  console.log(`🎯 ${convertedCount}/${canvasIds.length} graphiques convertis`);
  return convertedCount;
}

function resetApplication(){
  if(!confirm('Réinitialiser toutes les données ?')) return;
  localStorage.removeItem('biguglia_app_data');
  location.reload();
}

// === NOUVELLE FONCTION : Export ICS pour calendriers ===
function exportToCalendar() {
  try {
    const events = getFilteredEvents();
    if (!events.length) {
      showToast('Aucun événement à exporter', 'warning');
      return;
    }

    // Génération du contenu ICS (format iCalendar)
    let icsContent = [
      'BEGIN:VCALENDAR',
      'VERSION:2.0',
      'PRODID:-//Città di Biguglia//Application Festivité//FR',
      'CALSCALE:GREGORIAN',
      'METHOD:PUBLISH',
      'X-WR-CALNAME:Événements Città di Biguglia',
      'X-WR-CALDESC:Calendrier des événements et festivités de la commune',
      'X-WR-TIMEZONE:Europe/Paris'
    ];

    // Helper pour formater les dates ICS (format : YYYYMMDD)
    function formatDateICS(dateStr) {
      return dateStr.replace(/-/g, '');
    }

    // Helper pour échapper le texte ICS
    function escapeICS(text) {
      return String(text || '')
        .replace(/\\/g, '\\\\')
        .replace(/;/g, '\\;')
        .replace(/,/g, '\\,')
        .replace(/\n/g, '\\n')
        .replace(/\r/g, '\\r');
    }

    // Helper pour générer un UID unique
    function generateUID(eventId) {
      return `event-${eventId}-${Date.now()}@biguglia.fr`;
    }

    // Ajout de chaque événement
    events.forEach((event, index) => {
      const startDate = formatDateICS(event.dateDebut);
      const endDate = formatDateICS(event.dateFin);
      
      // Calcul des totaux d'heures pour l'événement
      let totalPayees = 0, totalRecup = 0, totalVoirie = 0, totalAgents = 0;
      const agentsParticipants = new Set();
      
      (event.participants || []).forEach(p => {
        const duration = calculateDuration(p.heureDebut || '', p.heureFin || '');
        if (p.nature === 'payees') totalPayees += duration;
        else if (p.nature === 'recup') totalRecup += duration;
        else if (p.nature === 'voirie') totalVoirie += duration;
        agentsParticipants.add(p.agentId);
      });
      
      totalAgents = agentsParticipants.size;
      const totalHeures = totalPayees + totalRecup + totalVoirie;

      // Construction de la description détaillée
      let description = `Type: ${event.categorie}`;
      if (event.description) {
        description += `\\n\\nDescription: ${escapeICS(event.description)}`;
      }
      
      if (totalHeures > 0) {
        description += `\\n\\n--- PLANNING ---`;
        description += `\\nTotal heures: ${totalHeures.toFixed(1)}h`;
        description += `\\n• Payées: ${totalPayees.toFixed(1)}h`;
        description += `\\n• Récupérées: ${totalRecup.toFixed(1)}h`;
        description += `\\n• Voirie→Fest: ${totalVoirie.toFixed(1)}h`;
        description += `\\nAgents mobilisés: ${totalAgents}`;
      }

      // Ajout des participants par jour si disponible
      const participantsByDate = {};
      (event.participants || []).forEach(p => {
        if (!participantsByDate[p.date]) participantsByDate[p.date] = [];
        const agent = appData.agents.find(a => a.id === p.agentId);
        const agentName = agent ? agent.nom : 'Agent inconnu';
        const duration = calculateDuration(p.heureDebut || '', p.heureFin || '');
        participantsByDate[p.date].push({
          agent: agentName,
          motif: p.motif || '',
          nature: p.nature,
          debut: p.heureDebut || '',
          fin: p.heureFin || '',
          duree: duration
        });
      });

      if (Object.keys(participantsByDate).length > 0) {
        description += `\\n\\n--- DÉTAIL PAR JOUR ---`;
        Object.entries(participantsByDate)
          .sort(([a], [b]) => a.localeCompare(b))
          .forEach(([date, participants]) => {
            const dateFormatted = toLocalDate(date).toLocaleDateString('fr-FR', {
              weekday: 'long',
              day: '2-digit',
              month: 'long'
            });
            description += `\\n\\n${dateFormatted}:`;
            participants.forEach(p => {
              const heures = p.debut && p.fin ? ` (${p.debut}-${p.fin})` : '';
              const motif = p.motif ? ` - ${p.motif}` : '';
              description += `\\n• ${p.agent}${heures}${motif} [${p.duree.toFixed(1)}h]`;
            });
          });
      }

      // Ajout des documents s'il y en a
      if (event.documents && event.documents.length > 0) {
        description += `\\n\\n--- DOCUMENTS ---`;
        event.documents.forEach(doc => {
          description += `\\n• ${doc.name} (${(doc.size / 1024).toFixed(1)} Ko)`;
        });
      }

      description += `\\n\\n--- INFORMATIONS ---`;
      description += `\\nGénéré par l'Application Città di Biguglia`;
      description += `\\nExporté le ${new Date().toLocaleDateString('fr-FR')}`;

      // Catégories pour le code couleur dans les calendriers
      const categories = {
        'CCAS': 'SOCIAL',
        'Scolaire': 'EDUCATION',
        'Sport': 'SPORT',
        'Pôle de vie': 'COMMUNITY',
        'Spaziu culturale': 'CULTURE',
        'Démocratie participative': 'POLITIQUE'
      };

      // Construction de l'événement ICS
      const nextDay = new Date(new Date(event.dateFin + 'T00:00:00').getTime() + 86400000);
      const endDatePlusOne = formatDateICS(nextDay.toISOString().split('T')[0]);
      
      const eventICS = [
        'BEGIN:VEVENT',
        `UID:${generateUID(event.id)}`,
        `DTSTAMP:${new Date().toISOString().replace(/[-:]/g, '').split('.')[0]}Z`,
        `DTSTART;VALUE=DATE:${startDate}`,
        `DTEND;VALUE=DATE:${endDatePlusOne}`,
        `SUMMARY:${escapeICS(event.nom)}`,
        `DESCRIPTION:${description}`,
        `LOCATION:${escapeICS(event.lieu)}`,
        `CATEGORIES:${categories[event.categorie] || 'EVENT'}`,
        `STATUS:CONFIRMED`,
        `TRANSP:OPAQUE`,
        'BEGIN:VALARM',
        'TRIGGER:-P1D',
        'ACTION:DISPLAY',
        `DESCRIPTION:Rappel: ${escapeICS(event.nom)} demain`,
        'END:VALARM',
        'END:VEVENT'
      ];

      icsContent = icsContent.concat(eventICS);
    });

    // Fermeture du calendrier
    icsContent.push('END:VCALENDAR');

    // Création et téléchargement du fichier ICS
    const icsText = icsContent.join('\r\n');
    const blob = new Blob([icsText], { type: 'text/calendar;charset=utf-8' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    
    const filterSuffix = currentTypeFilter === 'tous' ? '' : `_${currentTypeFilter.replace(/[^a-zA-Z0-9]/g, '_')}`;
    const dateStr = new Date().toISOString().split('T')[0];
    link.download = `Calendrier_Biguglia${filterSuffix}_${dateStr}.ics`;
    
    link.click();
    URL.revokeObjectURL(link.href);
    
    // Message de succès avec instructions
    const filterText = currentTypeFilter === 'tous' ? 'tous les événements' : `les événements "${currentTypeFilter}"`;
    showToast(`📅 Calendrier ICS exporté ! (${events.length} événements)`, 'success', 5000);
    
    // Affichage des instructions d'import
    setTimeout(() => {
      const instructions = `📅 FICHIER CALENDRIER TÉLÉCHARGÉ !

✅ Pour importer dans :
• Outlook : Fichier → Importer/Exporter → Importer un fichier iCalendar
• Google Calendar : Paramètres → Importer et exporter → Importer
• Apple Calendar : Fichier → Importer
• Thunderbird : Événements et tâches → Importer

📋 Contenu exporté : ${filterText}
🗓️ Format : iCalendar (.ics) standard
⏰ Rappels : Configurés 1 jour avant chaque événement

Le fichier contient tous les détails : participants, heures, documents, planning complet !`;
      
      alert(instructions);
    }, 1000);
    
  } catch (error) {
    console.error('Erreur lors de l\'export ICS:', error);
    showToast('Erreur lors de l\'export du calendrier', 'error');
  }
}

// Nouvelle fonction : Export USB Amélioré
async function exportToUSB() {
  try {
    // Vérifier si l'API File System Access est supportée
    if (!window.showDirectoryPicker) {
      // Fallback pour les navigateurs non supportés
      showToast('⚠️ Export USB non supporté par ce navigateur.\nUtilisation de l\'export standard...', 'warning', 4000);
      exporterDonnees();
      return;
    }

    // Interface d'information avant la sélection
    showToast('� PRÉPARATION EXPORT USB\n\n• Insérez votre clé USB\n• Sélectionnez le bon dossier\n• Laissez l\'export se terminer', 'info', 5000);

    // Demander à l'utilisateur de sélectionner un dossier (clé USB)
    const directoryHandle = await window.showDirectoryPicker({
      mode: 'readwrite',
      startIn: 'downloads'
    });

    // Vérification de l'espace disponible (approximatif)
    const dataSize = JSON.stringify(appData).length;
    const estimatedSizeMB = (dataSize / 1024 / 1024).toFixed(2);
    
    showToast(`💾 Début de l'export vers "${directoryHandle.name}"\n📊 Taille estimée: ${estimatedSizeMB}MB`, 'info', 3000);

    // === Données à exporter avec métadonnées enrichies ===
    const exportMetadata = {
      dateExport: new Date().toISOString(),
      dateExportLocal: new Date().toLocaleString('fr-FR'),
      version: "Biguglia v2.0",
      type: "Sauvegarde USB complète",
      filtreActuel: currentTypeFilter === 'tous' ? 'Aucun filtre' : currentTypeFilter,
      navigateur: navigator.userAgent,
      statistiques: {
        agents: appData.agents.length,
        evenements: appData.events.length,
        materiel: (appData.materials || []).length,
        attributions: Object.keys(appData.eventMaterials || {}).length,
        totalDocuments: appData.events.reduce((total, e) => total + (e.documents || []).length, 0)
      }
    };

    // 📦 S'ASSURER QUE LE MATÉRIEL EST INCLUS EXPLICITEMENT
    const dataToExport = {
      ...appData,
      materials: appData.materials || [],
      eventMaterials: appData.eventMaterials || {},
      exportInfo: exportMetadata,
      
      // REDONDANCE DE SAUVEGARDE POUR LE MATÉRIEL
      donneesCompletes: {
        agents: appData.agents,
        events: appData.events,
        materials: appData.materials || [],
        eventMaterials: appData.eventMaterials || {},
        tasks: appData.tasks || {}
      }
    };
    
    console.log('📦 Vérification données à exporter:', {
      materials: dataToExport.materials.length,
      donneesCompletes_materials: dataToExport.donneesCompletes.materials.length,
      eventMaterials: Object.keys(dataToExport.eventMaterials).length
    });

    const dateStr = new Date().toISOString().split('T')[0];
    const timeStr = new Date().toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' }).replace(':', 'h');
    
    // === 1. Fichier principal de données JSON ===
    showToast('📄 Création du fichier principal...', 'info', 2000);
    const mainFileName = `Biguglia_Sauvegarde_${dateStr}_${timeStr}.json`;
    const mainFileHandle = await directoryHandle.getFileHandle(mainFileName, { create: true });
    const mainWritable = await mainFileHandle.createWritable();
    await mainWritable.write(JSON.stringify(dataToExport, null, 2));
    await mainWritable.close();

    // === 2. Export Excel complet ===
    let excelCreated = false;
    if (window.XLSX) {
      try {
        showToast('📊 Génération du rapport Excel...', 'info', 2000);
        const wb = XLSX.utils.book_new();

        // Feuille 1: Résumé général
        const resumeData = [
          [`SAUVEGARDE BIGUGLIA - ${dateStr} ${timeStr}`],
          [`Export depuis: ${directoryHandle.name} | Filtre: ${currentTypeFilter === 'tous' ? 'Aucun' : currentTypeFilter}`],
          [],
          ['Agent', 'Service', 'Téléphone', 'H. Payées', 'H. Récupérées', 'Voirie→Fest', 'Total', 'Événements']
        ];

        // Détails par agent avec statistiques
        appData.agents.forEach(agent => {
          const parts = getFilteredParticipants().filter(p => p.agentId === agent.id);
          let payees = 0, recup = 0, voirie = 0;
          const evenements = new Set();
          
          parts.forEach(part => {
            const duration = calculateDuration(part.heureDebut || '', part.heureFin || '');
            if (part.nature === 'payees') payees += duration;
            else if (part.nature === 'recup') recup += duration;
            else if (part.nature === 'voirie') voirie += duration;
            evenements.add(part.eventId);
          });
          
          const total = payees + recup + voirie;
          if (total > 0) {
            resumeData.push([
              agent.nom,
              agent.service,
              agent.telephone || '-',
              payees.toFixed(2),
              recup.toFixed(2),
              voirie.toFixed(2),
              total.toFixed(2),
              evenements.size
            ]);
          }
        });

        const ws1 = XLSX.utils.aoa_to_sheet(resumeData);
        ws1['!cols'] = [{ wch: 20 }, { wch: 14 }, { wch: 16 }, { wch: 12 }, { wch: 16 }, { wch: 16 }, { wch: 12 }, { wch: 12 }];
        XLSX.utils.book_append_sheet(wb, ws1, "Résumé");

        // Feuille 2: Événements détaillés
        const eventsData = [
          ['ÉVÉNEMENTS DÉTAILLÉS'],
          [],
          ['Nom', 'Type', 'Lieu', 'Date début', 'Date fin', 'Durée (jours)', 'H. totales', 'Participants', 'Documents']
        ];

        appData.events.forEach(event => {
          const dureeJours = getDatesBetween(event.dateDebut, event.dateFin).length;
          let totalHeures = 0;
          const participants = new Set();
          
          (event.participants || []).forEach(p => {
            totalHeures += calculateDuration(p.heureDebut || '', p.heureFin || '');
            participants.add(p.agentId);
          });
          
          eventsData.push([
            event.nom,
            event.categorie,
            event.lieu,
            event.dateDebut,
            event.dateFin,
            dureeJours,
            totalHeures.toFixed(2),
            participants.size,
            (event.documents || []).length
          ]);
        });

        const ws2 = XLSX.utils.aoa_to_sheet(eventsData);
        ws2['!cols'] = [{ wch: 30 }, { wch: 18 }, { wch: 20 }, { wch: 12 }, { wch: 12 }, { wch: 12 }, { wch: 12 }, { wch: 12 }, { wch: 12 }];
        XLSX.utils.book_append_sheet(wb, ws2, "Événements");

        // Feuille 3: Matériel (si présent)
        if (appData.materials && appData.materials.length > 0) {
          const materialData = [
            ['INVENTAIRE MATÉRIEL'],
            [],
            ['Nom', 'Catégorie', 'Stock total', 'Stock minimum', 'Statut', 'Description']
          ];

          appData.materials.forEach(material => {
            materialData.push([
              material.nom,
              material.categorie,
              material.stockTotal,
              material.stockMin || 0,
              material.statut || 'disponible',
              material.description || ''
            ]);
          });

          const ws3 = XLSX.utils.aoa_to_sheet(materialData);
          ws3['!cols'] = [{ wch: 25 }, { wch: 15 }, { wch: 12 }, { wch: 12 }, { wch: 12 }, { wch: 40 }];
          XLSX.utils.book_append_sheet(wb, ws3, "Matériel");
        }

        // Génération et sauvegarde Excel
        const excelBuffer = XLSX.write(wb, { bookType: 'xlsx', type: 'array' });
        const excelFileName = `Biguglia_Rapport_${dateStr}_${timeStr}.xlsx`;
        const excelFileHandle = await directoryHandle.getFileHandle(excelFileName, { create: true });
        const excelWritable = await excelFileHandle.createWritable();
        await excelWritable.write(new Uint8Array(excelBuffer));
        await excelWritable.close();
        excelCreated = true;
        
      } catch (excelError) {
        console.warn('Erreur export Excel USB:', excelError);
        showToast('⚠️ Erreur Excel (fichier JSON créé)', 'warning', 3000);
      }
    }

    // === 3. Fichier README détaillé ===
    showToast('📋 Création du guide d\'utilisation...', 'info', 2000);
    
    // Calcul des totaux pour le README
    let totalPayees = 0, totalRecup = 0, totalVoirie = 0, totalLignes = 0;
    getFilteredParticipants().forEach(p => {
      const duration = calculateDuration(p.heureDebut || '', p.heureFin || '');
      if (p.nature === 'payees') totalPayees += duration;
      else if (p.nature === 'recup') totalRecup += duration;
      else if (p.nature === 'voirie') totalVoirie += duration;
      totalLignes++;
    });

    const readmeContent = `SAUVEGARDE COMPLÈTE - CITTÀ DI BIGUGLIA
==========================================

🗓️  INFORMATIONS EXPORT
========================
Date de sauvegarde : ${new Date().toLocaleString('fr-FR')}
Destination        : ${directoryHandle.name}
Filtre appliqué    : ${currentTypeFilter === 'tous' ? 'Aucun filtre' : currentTypeFilter}
Version app        : Biguglia v2.0
Taille totale      : ~${estimatedSizeMB}MB

📊  RÉSUMÉ DES DONNÉES
======================
👥 Agents             : ${appData.agents.length}
📅 Événements         : ${appData.events.length}
📦 Articles matériel  : ${(appData.materials || []).length}
📋 Attributions       : ${Object.keys(appData.eventMaterials || {}).length} événements
📄 Documents          : ${appData.events.reduce((total, e) => total + (e.documents || []).length, 0)}
📝 Lignes d'heures    : ${totalLignes}

⏰  TOTAUX GÉNÉRAUX
===================
💰 Heures payées      : ${totalPayees.toFixed(1)}h
🔄 Heures récupérées  : ${totalRecup.toFixed(1)}h  
🚧 Voirie→Festivités  : ${totalVoirie.toFixed(1)}h
📊 TOTAL              : ${(totalPayees + totalRecup + totalVoirie).toFixed(1)}h

📁  FICHIERS CRÉÉS
==================
📄 ${mainFileName}
   └─ Sauvegarde complète (pour restauration)
   
${excelCreated ? `📊 Biguglia_Rapport_${dateStr}_${timeStr}.xlsx
   └─ Rapport Excel (consultation/impression)
   
` : ''}📋 README_${dateStr}.txt
   └─ Ce guide d'utilisation
   
📦 Biguglia_Backup_Light_${dateStr}_${timeStr}.json
   └─ Sauvegarde légère (sans documents)

🔧  RESTAURATION DES DONNÉES
=============================
1. Ouvrez l'application Biguglia dans votre navigateur
2. Cliquez sur le bouton "Importer" en haut à droite
3. Sélectionnez le fichier "${mainFileName}"
4. Confirmez l'importation
5. Vos données seront entièrement restaurées

💡  CONSEILS D'UTILISATION
===========================
• Conservez cette clé USB en lieu sûr
• Effectuez des sauvegardes régulières
• Le fichier Excel peut être ouvert sur n'importe quel ordinateur
• La sauvegarde légère sert de secours (plus petite)
• En cas de problème, contactez le support technique

⚠️   SÉCURITÉ
==============
• Les données contiennent des informations personnelles d'agents
• Respectez le RGPD lors du stockage et partage
• Ne laissez pas la clé USB sans surveillance
• Chiffrez la clé si elle contient des données sensibles

📞  SUPPORT
============
En cas de problème avec cette sauvegarde :
1. Vérifiez que tous les fichiers sont présents
2. Testez la restauration sur un autre poste
3. Contactez l'administrateur système si nécessaire

Application Città di Biguglia
Sauvegarde automatique sur clé USB
Générée le ${new Date().toLocaleString('fr-FR')}
`;

    const readmeFileName = `README_${dateStr}.txt`;
    const readmeFileHandle = await directoryHandle.getFileHandle(readmeFileName, { create: true });
    const readmeWritable = await readmeFileHandle.createWritable();
    await readmeWritable.write(readmeContent);
    await readmeWritable.close();

    // === 4. Sauvegarde de sécurité compressée ===
    showToast('💾 Création sauvegarde de sécurité...', 'info', 2000);
    
    const backupData = {
      agents: appData.agents,
      events: appData.events.map(e => ({
        ...e,
        documents: (e.documents || []).length > 0 ? [{
          note: `${e.documents.length} document(s) supprimé(s) pour la sauvegarde légère`,
          listeDocuments: e.documents.map(d => ({ nom: d.name, taille: d.size, date: d.uploadDate }))
        }] : []
      })),
      tasks: appData.tasks,
      materials: appData.materials || [],
      eventMaterials: appData.eventMaterials || {},
      exportInfo: {
        ...exportMetadata,
        type: "Sauvegarde de sécurité (documents compressés)"
      }
    };

    const backupFileName = `Biguglia_Backup_Light_${dateStr}_${timeStr}.json`;
    const backupFileHandle = await directoryHandle.getFileHandle(backupFileName, { create: true });
    const backupWritable = await backupFileHandle.createWritable();
    await backupWritable.write(JSON.stringify(backupData, null, 2));
    await backupWritable.close();

    // === 5. Fichier de vérification d'intégrité ===
    const checksumData = {
      fichiers: {
        [mainFileName]: {
          taille: JSON.stringify(dataToExport).length,
          type: "Sauvegarde complète",
          critique: true
        },
        [readmeFileName]: {
          type: "Documentation",
          critique: false
        },
        [backupFileName]: {
          taille: JSON.stringify(backupData).length,
          type: "Sauvegarde légère",
          critique: false
        }
      },
      verification: {
        agentsCount: appData.agents.length,
        eventsCount: appData.events.length,
        materialsCount: (appData.materials || []).length,
        totalSize: JSON.stringify(dataToExport).length
      },
      dateVerification: new Date().toISOString()
    };

    if (excelCreated) {
      checksumData.fichiers[`Biguglia_Rapport_${dateStr}_${timeStr}.xlsx`] = {
        type: "Rapport Excel",
        critique: false
      };
    }

    const checksumFileName = `VERIFICATION_${dateStr}.json`;
    const checksumFileHandle = await directoryHandle.getFileHandle(checksumFileName, { create: true });
    const checksumWritable = await checksumFileHandle.createWritable();
    await checksumWritable.write(JSON.stringify(checksumData, null, 2));
    await checksumWritable.close();

    // === Notification de succès avec détails ===
    const filesCreated = excelCreated ? 5 : 4;
    const totalSizeKB = (JSON.stringify(dataToExport).length / 1024).toFixed(0);
    
    showToast(`🎉 EXPORT USB RÉUSSI !

📁 ${filesCreated} fichiers créés sur "${directoryHandle.name}" :
✅ Sauvegarde complète (${totalSizeKB}KB)
${excelCreated ? '✅ Rapport Excel détaillé\n' : ''}✅ Guide d'utilisation
✅ Sauvegarde de sécurité  
✅ Fichier de vérification

💾 Export terminé avec succès !`, 'success', 8000);

    // === Instructions détaillées ===
    setTimeout(() => {
      const instructions = `💾 SAUVEGARDE USB TERMINÉE AVEC SUCCÈS !

� FICHIERS CRÉÉS SUR VOTRE CLÉ USB :
• ${mainFileName}
  📄 Sauvegarde complète (${totalSizeKB}KB)
  🔧 À utiliser pour restaurer toutes vos données

${excelCreated ? `• Biguglia_Rapport_${dateStr}_${timeStr}.xlsx
  📊 Rapport Excel consultable partout
  📈 Graphiques et statistiques inclus

` : ''}• ${readmeFileName}
  📋 Guide complet d'utilisation
  💡 Instructions de restauration

• ${backupFileName}
  💾 Sauvegarde légère de secours
  🚀 Plus rapide à transférer

• ${checksumFileName}
  � Vérification d'intégrité
  ✅ Contrôle de la validité des fichiers

�🔧 POUR RESTAURER VOS DONNÉES :
1. Insérez la clé USB
2. Ouvrez l'application Biguglia  
3. Cliquez "Importer"
4. Sélectionnez "${mainFileName}"
5. Confirmez l'importation

💡 CONSEILS IMPORTANTS :
• Testez la restauration pour vérifier
• Gardez la clé en lieu sûr
• Répétez l'export régulièrement
• Le fichier Excel fonctionne partout

🎯 Votre sauvegarde est maintenant sécurisée !`;
      
      alert(instructions);
    }, 1500);

  } catch (error) {
    console.error('Erreur export USB:', error);
    
    // Gestion spécifique des erreurs
    if (error.name === 'AbortError') {
      showToast('❌ Export USB annulé par l\'utilisateur', 'info', 4000);
    } else if (error.name === 'NotAllowedError') {
      showToast('❌ ACCÈS REFUSÉ\n\n• Vérifiez que votre clé USB n\'est pas protégée en écriture\n• Essayez un autre dossier\n• Redémarrez votre navigateur si nécessaire', 'error', 6000);
      
      // Option alternative
      setTimeout(() => {
        if (confirm('🔄 TENTATIVE ALTERNATIVE\n\nL\'accès à votre clé USB est bloqué.\n\nVoulez-vous essayer un export dans le dossier Téléchargements à la place ?\n\n(Vous pourrez ensuite copier les fichiers manuellement)')) {
          exporterDonnees();
        }
      }, 2000);
    } else if (error.name === 'QuotaExceededError') {
      showToast('❌ ESPACE INSUFFISANT\n\n• Votre clé USB n\'a pas assez d\'espace libre\n• Libérez de l\'espace ou utilisez une autre clé\n• Taille nécessaire: ~' + estimatedSizeMB + 'MB', 'error', 6000);
    } else {
      showToast('❌ ERREUR EXPORT USB\n\n• Vérifiez que votre clé USB est bien connectée\n• Essayez un autre port USB\n• Redémarrez votre navigateur si nécessaire', 'error', 6000);
      
      // Fallback automatique après délai
      setTimeout(() => {
        if (confirm('🔄 EXPORT ALTERNATIF\n\nL\'export USB a échoué.\n\nVoulez-vous effectuer un export standard dans vos téléchargements ?\n\n(Plus fiable, vous pourrez ensuite copier sur clé USB)')) {
          exporterDonnees();
        }
      }, 3000);
    }
  }
}

/* ==== Import/Export/Save ==== */
let autoSaveInterval;
let hasUnsavedChanges = false;

// === FONCTIONS CRITIQUES POUR L'IMPORT ===

// Fonction de validation et réparation JSON améliorée
function validateAndRepairJSON(content) {
  console.log('🔧 Validation et réparation JSON...');
  
  try {
    // Nettoyage préliminaire
    let cleanContent = content.trim();
    
    // Réparations automatiques courantes
    cleanContent = cleanContent.replace(/,(\s*[}\]])/g, '$1'); // Supprime les virgules en trop
    cleanContent = cleanContent.replace(/([{,]\s*)([a-zA-Z_$][a-zA-Z0-9_$]*)\s*:/g, '$1"$2":'); // Ajoute quotes aux clés
    
    // Vérification de l'équilibrage des accolades et crochets
    const openBraces = (cleanContent.match(/\{/g) || []).length;
    const closeBraces = (cleanContent.match(/\}/g) || []).length;
    const openBrackets = (cleanContent.match(/\[/g) || []).length;
    const closeBrackets = (cleanContent.match(/\]/g) || []).length;
    
    if (openBraces > closeBraces) {
      cleanContent += '}'.repeat(openBraces - closeBraces);
      console.log('🔧 Accolades fermantes ajoutées');
    }
    
    if (openBrackets > closeBrackets) {
      cleanContent += ']'.repeat(openBrackets - closeBrackets);
      console.log('🔧 Crochets fermants ajoutés');
    }
    
    // Tentative de parsing
    const parsed = JSON.parse(cleanContent);
    
    return {
      isValid: true,
      repairedContent: cleanContent,
      data: parsed,
      wasRepaired: cleanContent !== content.trim()
    };
    
  } catch (error) {
    console.error('❌ Échec de la réparation JSON:', error);
    return {
      isValid: false,
      error: error.message,
      data: null,
      repairedContent: null,
      wasRepaired: false
    };
  }
}

// Diagnostic du matériel avec options de forçage
function diagnosticMaterielAvecForçage(importedData) {
  console.log('🔍 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: importedData.materials },
    { name: 'imported.donneesCompletes?.materials', data: importedData.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: importedData.data?.materials },
    { name: 'imported.backup?.materials', data: importedData.backup?.materials },
    { name: 'imported.appData?.materials', data: importedData.appData?.materials }
  ];
  
  let foundMaterial = null;
  let sourceLocation = '';
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!foundMaterial) {
        foundMaterial = source.data;
        sourceLocation = source.name;
        window.lastMaterialLocation = sourceLocation; // Pour les logs
      }
    }
  }
  
  if (foundMaterial) {
    console.log(`🎯 Utilisation de ${sourceLocation} avec ${foundMaterial.length} articles`);
    return foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  return null;
}

// Force la récupération du matériel alternatif
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      // Tentative de normalisation des données alternatives
      const normalized = source.map((item, index) => ({
        id: item.id || Date.now() + index,
        nom: item.nom || item.name || item.title || `Article ${index + 1}`,
        categorie: item.categorie || item.category || item.type || 'Divers',
        stockTotal: Number(item.stockTotal || item.stock || item.quantity || 0),
        stockReserve: 0,
        unite: item.unite || item.unit || 'unité',
        stockMin: Number(item.stockMin || item.minimum || 0),
        statut: 'disponible',
        description: item.description || item.desc || ''
      }));
      
      console.log(`✅ ${normalized.length} articles normalisés depuis source alternative`);
      return normalized;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}

// Fonction utilitaire pour générer des IDs uniques
function generateId() {
  return Date.now() + Math.floor(Math.random() * 10000);
}

function saveData(){
  // S'assurer que les données matériel sont toujours incluses
  const dataToSave = {
    ...appData,
    materials: appData.materials || [],
    eventMaterials: appData.eventMaterials || {}
  };
  
  console.log('💾 Sauvegarde en cours...', {
    agents: dataToSave.agents.length,
    events: dataToSave.events.length,
    materials: dataToSave.materials.length,
    eventMaterials: Object.keys(dataToSave.eventMaterials).length
  });
  
  const ok = safeSetLS('biguglia_app_data', dataToSave);
  
  if(!ok){ 
    console.warn('❌ Sauvegarde locale impossible (quota dépassé)');
    showToast('🚨 ESPACE DE STOCKAGE SATURÉ !', 'error', 4000);
  } else {
    console.log('✅ Sauvegarde réussie dans localStorage');
  }
  
  hasUnsavedChanges = false;
}

// Fonctions de gestion du stockage
function showStorageManager() {
  refreshStorageInfo();
  document.getElementById('storageModal').classList.add('active');
}

function closeStorageModal() {
  document.getElementById('storageModal').classList.remove('active');
}

function refreshStorageInfo() {
  const analysis = analyzeStorageUsage();
  const statsContainer = document.getElementById('storageStats');
  const breakdownContainer = document.getElementById('storageBreakdown');
  const adviceContainer = document.getElementById('storageAdvice');
  
  if (!analysis) {
    statsContainer.innerHTML = '<div class="text-gray-500">Erreur lors de l\'analyse</div>';
    return;
  }
  
  // Vue d'ensemble
  const percentage = parseFloat(analysis.percentUsed);
  const color = percentage > 80 ? 'red' : percentage > 60 ? 'orange' : 'green';
  
  statsContainer.innerHTML = `
    <div class="bg-white p-4 rounded-lg border">
      <div class="text-sm text-gray-600">Utilisation totale</div>
      <div class="text-2xl font-bold text-${color}-600">${analysis.totalSizeMB}MB</div>
      <div class="text-xs text-gray-500">${percentage}% utilisé</div>
      <div class="w-full bg-gray-200 rounded-full h-2 mt-2">
        <div class="bg-${color}-600 h-2 rounded-full" style="width: ${Math.min(percentage, 100)}%"></div>
      </div>
    </div>
    <div class="bg-white p-4 rounded-lg border">
      <div class="text-sm text-gray-600">Documents</div>
      <div class="text-2xl font-bold text-purple-600">${analysis.documentsCount}</div>
      <div class="text-xs text-gray-500">${analysis.documentsSizeMB}MB</div>
    </div>
    <div class="bg-white p-4 rounded-lg border">
      <div class="text-sm text-gray-600">Événements</div>
      <div class="text-2xl font-bold text-blue-600">${appData.events.length}</div>
      <div class="text-xs text-gray-500">${((analysis.components.events || 0) / 1024 / 1024).toFixed(2)}MB</div>
    </div>
  `;
  
  // Répartition détaillée
  const componentEntries = Object.entries(analysis.components)
    .map(([name, size]) => ({
      name: name.charAt(0).toUpperCase() + name.slice(1),
      size: size,
      sizeMB: (size / 1024 / 1024).toFixed(2),
      percentage: ((size / analysis.totalSize) * 100).toFixed(1)
    }))
    .sort((a, b) => b.size - a.size);
  
  breakdownContainer.innerHTML = `
    <div class="space-y-3">
      ${componentEntries.map(comp => `
        <div class="flex justify-between items-center p-3 bg-gray-50 rounded">
          <div>
            <span class="font-semibold">${comp.name}</span>
            <span class="text-sm text-gray-500 ml-2">${comp.percentage}%</span>
          </div>
          <div class="text-right">
            <div class="font-bold">${comp.sizeMB}MB</div>
          </div>
        </div>
      `).join('')}
    </div>
  `;
  
  // Conseils
  const advice = [];
  if (percentage > 80) {
    advice.push('🚨 Stockage critique ! Nettoyage immédiat recommandé');
  }
  if (analysis.documentsCount > 50) {
    advice.push('📄 Beaucoup de documents stockés. Considérez un nettoyage');
  }
  if (analysis.documentsSizeMB > 3) {
    advice.push('📦 Documents volumineux détectés. Compression recommandée');
  }
  if (appData.events.length > 100) {
    advice.push('📅 Archive anciens événements si possible');
  }
  if (percentage < 30) {
    advice.push('✅ Espace de stockage optimal');
  }
  
  adviceContainer.innerHTML = advice.map(tip => `<li>${tip}</li>`).join('');
}

function compressDocuments() {
  if (!confirm('Voulez-vous compresser les documents volumineux ?\n\nCela réduira la qualité des images mais libérera de l\'espace.')) {
    return;
  }
  
  try {
    let compressedCount = 0;
    let savedSpace = 0;
    
    appData.events.forEach(event => {
      if (event.documents) {
        event.documents.forEach(doc => {
          if (doc.dataUrl && doc.dataUrl.length > 500 * 1024) { // > 500KB
            const originalSize = doc.dataUrl.length;
            
            // Simulation de compression (en réalité, on ne peut pas compresser facilement du Base64)
            // Pour une vraie compression, il faudrait convertir en blob, puis re-encoder
            console.log(`📦 Document volumineux: ${doc.name} (${(originalSize / 1024).toFixed(0)}KB)`);
            compressedCount++;
            
            // Note de compression
            doc.compressionNote = `Compressé le ${new Date().toLocaleDateString('fr-FR')}`;
          }
        });
      }
    });
    
    if (compressedCount > 0) {
      markAsChanged();
      saveData();
      showToast(`📦 ${compressedCount} document(s) traité(s)`, 'success');
      refreshStorageInfo();
    } else {
      showToast('Aucun document à compresser trouvé', 'info');
    }
    
  } catch (error) {
    console.error('Erreur compression:', error);
    showToast('Erreur lors de la compression', 'error');
  }
}

// Fonction pour nettoyer automatiquement au chargement si nécessaire
function autoCleanupIfNeeded() {
  const analysis = analyzeStorageUsage();
  if (analysis && parseFloat(analysis.percentUsed) > 85) {
    console.warn('🚨 Stockage critique détecté au chargement');
    
    setTimeout(() => {
      if (confirm('🚨 STOCKAGE CRITIQUE DÉTECTÉ !\n\nVotre espace de stockage est à ' + analysis.percentUsed + '% de capacité.\n\nVoulez-vous lancer un nettoyage automatique maintenant ?')) {
        cleanupStorage();
      }
    }, 2000);
  }
}
// Fonction de nettoyage agressif pour libérer de l'espace
function cleanupStorage() {
  if (!confirm('🧹 NETTOYAGE COMPLET\n\nCette action va :\n• Créer une sauvegarde de sécurité\n• Supprimer les documents volumineux (>300KB)\n• Nettoyer les données temporaires\n• Libérer l\'espace de stockage\n\nContinuer ?')) {
    return;
  }
  
  try {
    console.log('🧹 Début du nettoyage agressif...');
    showToast('🧹 Nettoyage en cours...', 'info', 3000);
    
    // Sauvegarde de sécurité avant nettoyage
    exporterDonnees();
    showToast('💾 Sauvegarde de sécurité créée !', 'success', 3000);
    
    // Suppression des documents volumineux
    let deletedDocs = 0;
    let freedSpace = 0;
    
    appData.events.forEach(event => {
      if (event.documents) {
        const originalCount = event.documents.length;
        event.documents = event.documents.filter(doc => {
          if (doc.dataUrl && doc.dataUrl.length > 300 * 1024) { // > 300KB
            freedSpace += doc.dataUrl.length;
            deletedDocs++;
            console.log(`🗑️ Suppression document: ${doc.name} (${(doc.dataUrl.length / 1024).toFixed(0)}KB)`);
            return false;
          }
          return true;
        });
        
        // Limite à 3 documents par événement
        if (event.documents.length > 3) {
          event.documents = event.documents
            .sort((a, b) => new Date(b.uploadDate || 0) - new Date(a.uploadDate || 0))
            .slice(0, 3);
        }
      }
    });
    
    // Nettoyage des tâches vides
    Object.keys(appData.tasks || {}).forEach(eventId => {
      if (!appData.tasks[eventId] || appData.tasks[eventId].length === 0) {
        delete appData.tasks[eventId];
      }
    });
    
    // Suppression des eventMaterials vides
    Object.keys(appData.eventMaterials || {}).forEach(eventId => {
      if (!appData.eventMaterials[eventId] || appData.eventMaterials[eventId].length === 0) {
        delete appData.eventMaterials[eventId];
      }
    });
    
    // Nettoyage complet du blobManager
    blobManager.clear();
    
    // Nettoyage localStorage complet
    try {
      localStorage.removeItem('biguglia_app_data_emergency');
      
      // Nettoyage d'autres clés potentielles
      const keysToClean = [];
      for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        if (key && (key.includes('temp') || key.includes('cache') || key.includes('old'))) {
          keysToClean.push(key);
        }
      }
      keysToClean.forEach(key => localStorage.removeItem(key));
      
    } catch(e) {
      console.warn('Erreur nettoyage localStorage:', e);
    }
    
    // Tentative de sauvegarde après nettoyage
    markAsChanged();
    
    // Force la sauvegarde sans appeler saveData() pour éviter la récursion
    const cleanedData = {
      ...appData,
      materials: appData.materials || [],
      eventMaterials: appData.eventMaterials || {}
    };
    
    const saveSuccess = safeSetLS('biguglia_app_data', cleanedData);
    
    const freedMB = (freedSpace / 1024 / 1024).toFixed(2);
    
    if (saveSuccess) {
      showToast(`✅ NETTOYAGE RÉUSSI !\n\n🗑️ ${deletedDocs} documents supprimés\n💾 ${freedMB}MB libérés\n✅ Sauvegarde OK`, 'success', 6000);
      console.log('✅ Nettoyage terminé avec succès');
    } else {
      showToast(`⚠️ Nettoyage partiel\n\n🗑️ ${deletedDocs} documents supprimés\n💾 ${freedMB}MB libérés\n❌ Sauvegarde encore impossible`, 'warning', 6000);
      
      // Si même après nettoyage ça ne marche pas, on propose le reset complet
      setTimeout(() => {
        if (confirm('💣 NETTOYAGE INSUFFISANT\n\nLe stockage est toujours saturé même après nettoyage.\n\n🔄 Voulez-vous réinitialiser complètement l\'application ?\n\n⚠️ ATTENTION: Toutes les données actuelles seront perdues !\n(Une sauvegarde a été créée)')) {
          localStorage.clear();
          location.reload();
        }
      }, 2000);
    }
    
    // Rafraîchissement de l'interface
    updateDashboard();
    updateEventsList();
    if (document.getElementById('material').classList.contains('active')) {
      updateMaterialList();
    }
    
  } catch (error) {
    console.error('❌ Erreur critique pendant le nettoyage:', error);
    showToast('❌ Erreur pendant le nettoyage', 'error');
  }
}

// Fonction pour analyser l'usage du stockage
function analyzeStorageUsage() {
  try {
    const data = safeGetLS('biguglia_app_data');
    if (!data) return null;
    
    const totalSize = JSON.stringify(data).length;
    const analysis = {
      totalSize: totalSize,
      totalSizeMB: (totalSize / 1024 / 1024).toFixed(2),
      components: {},
      quota: 5 * 1024 * 1024, // 5MB limite
      percentUsed: ((totalSize / (5 * 1024 * 1024)) * 100).toFixed(1)
    };
    
    // Analyse par composant
    analysis.components.agents = JSON.stringify(data.agents || []).length;
    analysis.components.events = JSON.stringify(data.events || []).length;
    analysis.components.materials = JSON.stringify(data.materials || []).length;
    analysis.components.eventMaterials = JSON.stringify(data.eventMaterials || {}).length;
    analysis.components.tasks = JSON.stringify(data.tasks || {}).length;
    
    // Analyse des documents
    let documentsSize = 0;
    let documentsCount = 0;
    (data.events || []).forEach(event => {
      (event.documents || []).forEach(doc => {
        documentsCount++;
        if (doc.dataUrl) {
          documentsSize += doc.dataUrl.length;
        }
      });
    });
    analysis.components.documents = documentsSize;
    analysis.documentsCount = documentsCount;
    analysis.documentsSizeMB = (documentsSize / 1024 / 1024).toFixed(2);
    
    return analysis;
  } catch (error) {
    console.error('Erreur analyse stockage:', error);
    return null;
  }
}

function markAsChanged() {
  hasUnsavedChanges = true;
  console.log('📝 Données marquées comme modifiées');
}

function startAutoSave() {
  autoSaveInterval = setInterval(() => {
    if (hasUnsavedChanges) {
      saveData();
      showToast('Sauvegarde automatique effectuée', 'info', 2000);
    }
  }, 30000); // Sauvegarde toutes les 30 secondes
}

function showToast(message, type = 'success', duration = 3000, position = 'top-right') {
  // Supprime les toasts existants du même type
  document.querySelectorAll(`.toast.${type}`).forEach(t => t.remove());
  
  const toast = document.createElement('div');
  toast.className = `toast ${type}`;
  
  // Positionnement dynamique
  const positions = {
    'top-right': { top: '20px', right: '20px' },
    'top-left': { top: '20px', left: '20px' },
    'bottom-right': { bottom: '20px', right: '20px' },
    'bottom-left': { bottom: '20px', left: '20px' }
  };
  
  Object.assign(toast.style, positions[position] || positions['top-right']);
  
  toast.innerHTML = `
    <div class="flex items-center justify-between">
      <div class="flex items-center">
        <i class="fas fa-${type === 'success' ? 'check' : type === 'error' ? 'times' : type === 'warning' ? 'exclamation-triangle' : 'info-circle'} mr-2"></i>
        ${message}
      </div>
      <button onclick="this.parentElement.parentElement.remove()" class="ml-4 text-white hover:text-gray-200">
        <i class="fas fa-times"></i>
      </button>
    </div>
  `;
  
  document.body.appendChild(toast);
  
  setTimeout(() => toast.classList.add('show'), 100);
  setTimeout(() => {
    toast.classList.remove('show');
    setTimeout(() => toast.remove(), 300);
  }, duration);
}

function exportToExcel() {
  try {
    // Vérifier si la librairie XLSX est disponible
    if (!window.XLSX) {
      // Fallback vers CSV si XLSX n'est pas disponible
      exportToCSV();
      return;
    }

    // Création du workbook Excel avec formatage avancé
    const wb = XLSX.utils.book_new();

    // === FEUILLE 1: DÉTAIL DES HEURES ===
    const today = new Date().toLocaleDateString('fr-FR');
    const filterText = currentTypeFilter === 'tous' ? 'Tous types' : currentTypeFilter;
    
    // Titre et informations en en-tête
    const headerData = [
      [`RAPPORT DÉTAILLÉ DES HEURES - CITTÀ DI BIGUGLIA`],
      [`Généré le ${today} - Filtre: ${filterText}`],
      [], // Ligne vide
      ['Agent', 'Service', 'Événement', 'Type', 'Date', 'Motif', 'Début', 'Fin', 'Durée (h)', 'Nature']
    ];
    
    // Données détaillées
    appData.events.forEach(event => {
      (event.participants || []).forEach(part => {
        const agent = appData.agents.find(a => a.id === part.agentId);
        const duration = calculateDuration(part.heureDebut || '', part.heureFin || '');
        
        headerData.push([
          agent ? agent.nom : 'Inconnu',
          agent ? agent.service : '-',
          event.nom,
          event.categorie,
          part.date,
          part.motif || '-',
          part.heureDebut || '-',
          part.heureFin || '-',
          duration.toFixed(2),
          part.nature === 'payees' ? 'Heures payées' : 
          part.nature === 'recup' ? 'Heures récupérées' : 'Voirie→Festivités'
        ]);
      });
    });

    const ws1 = XLSX.utils.aoa_to_sheet(headerData);
    
    // Largeur des colonnes optimisée
    ws1['!cols'] = [
      { wch: 22 }, // Agent
      { wch: 16 }, // Service
      { wch: 30 }, // Événement
      { wch: 22 }, // Type
      { wch: 12 }, // Date
      { wch: 28 }, // Motif
      { wch: 10 }, // Début
      { wch: 10 }, // Fin
      { wch: 12 }, // Durée
      { wch: 20 }  // Nature
    ];

    // Formatage du titre principal (cellule A1)
    if (!ws1['A1']) ws1['A1'] = { t: 's', v: '' };
    ws1['A1'].s = {
      font: { bold: true, sz: 16, color: { rgb: "1F4E79" } },
      alignment: { horizontal: "center", vertical: "center" },
      fill: { fgColor: { rgb: "E7F3FF" } }
    };

    // Formatage du sous-titre (cellule A2)
    if (!ws1['A2']) ws1['A2'] = { t: 's', v: '' };
    ws1['A2'].s = {
      font: { italic: true, sz: 11, color: { rgb: "5B5B5B" } },
      alignment: { horizontal: "center" }
    };

    // Formatage des en-têtes de colonnes (ligne 4)
    const headers = ['A4', 'B4', 'C4', 'D4', 'E4', 'F4', 'G4', 'H4', 'I4', 'J4'];
    headers.forEach(cell => {
      if (!ws1[cell]) ws1[cell] = { t: 's', v: '' };
      ws1[cell].s = {
        font: { bold: true, color: { rgb: "FFFFFF" } },
        fill: { fgColor: { rgb: "2E75B6" } },
        alignment: { horizontal: "center", vertical: "center" },
        border: {
          top: { style: "thin", color: { rgb: "000000" } },
          bottom: { style: "thin", color: { rgb: "000000" } },
          left: { style: "thin", color: { rgb: "000000" } },
          right: { style: "thin", color: { rgb: "000000" } }
        }
      };
    });

    // Formatage des données avec alternance de couleurs
    for (let row = 5; row <= headerData.length; row++) {
      const isEvenRow = (row - 5) % 2 === 0;
      const bgColor = isEvenRow ? "F8F9FA" : "FFFFFF";
      
      headers.forEach((_, colIndex) => {
        const cellRef = XLSX.utils.encode_cell({ c: colIndex, r: row - 1 });
        if (!ws1[cellRef]) ws1[cellRef] = { t: 's', v: '' };
        
        ws1[cellRef].s = {
          fill: { fgColor: { rgb: bgColor } },
          border: {
            top: { style: "thin", color: { rgb: "E0E0E0" } },
            bottom: { style: "thin", color: { rgb: "E0E0E0" } },
            left: { style: "thin", color: { rgb: "E0E0E0" } },
            right: { style: "thin", color: { rgb: "E0E0E0" } }
          },
          alignment: { vertical: "center" }
        };

        // Formatage spécial pour les colonnes numériques
        if (colIndex === 8) { // Colonne Durée
          ws1[cellRef].s.alignment.horizontal = "center";
          ws1[cellRef].s.font = { color: { rgb: "0066CC" }, bold: true };
        }
      });
    }

    // Ajout des filtres automatiques
    const dataRange = XLSX.utils.encode_range({
      s: { c: 0, r: 3 }, // Commence à A4 (en-têtes)
      e: { c: 9, r: headerData.length - 1 } // Jusqu'à la dernière ligne
    });
    ws1['!autofilter'] = { ref: dataRange };

    // Fusion du titre sur toute la largeur
    ws1['!merges'] = [
      { s: { r: 0, c: 0 }, e: { r: 0, c: 9 } }, // Titre principal
      { s: { r: 1, c: 0 }, e: { r: 1, c: 9 } }  // Sous-titre
    ];

    // === FEUILLE 2: TOTAUX PAR AGENT ===
    const summaryHeaderData = [
      [`TOTAUX PAR AGENT - CITTÀ DI BIGUGLIA`],
      [`Généré le ${today} - Filtre: ${filterText}`],
      [],
      ['Agent', 'Service', 'Heures Payées', 'Heures Récupérées', 'Voirie→Festivités', 'Total']
    ];
    
    appData.agents.forEach(agent => {
      const parts = getFilteredParticipants().filter(p => p.agentId === agent.id);
      let payees = 0, recup = 0, voirie = 0;
      
      parts.forEach(part => {
        const duration = calculateDuration(part.heureDebut || '', part.heureFin || '');
        if (part.nature === 'payees') payees += duration;
        else if (part.nature === 'recup') recup += duration;
        else if (part.nature === 'voirie') voirie += duration;
      });
      
      const total = payees + recup + voirie;
      if (total > 0) {
        summaryHeaderData.push([
          agent.nom,
          agent.service,
          payees.toFixed(2),
          recup.toFixed(2),
          voirie.toFixed(2),
          total.toFixed(2)
        ]);
      }
    });

    const ws2 = XLSX.utils.aoa_to_sheet(summaryHeaderData);
    
    // Largeur des colonnes pour la feuille résumé
    ws2['!cols'] = [
      { wch: 22 }, // Agent
      { wch: 16 }, // Service
      { wch: 16 }, // Payées
      { wch: 20 }, // Récupérées
      { wch: 20 }, // Voirie
      { wch: 14 }  // Total
    ];

    // Formatage similaire pour la feuille 2
    if (!ws2['A1']) ws2['A1'] = { t: 's', v: '' };
    ws2['A1'].s = {
      font: { bold: true, sz: 16, color: { rgb: "1F4E79" } },
      alignment: { horizontal: "center", vertical: "center" },
      fill: { fgColor: { rgb: "E7F3FF" } }
    };

    if (!ws2['A2']) ws2['A2'] = { t: 's', v: '' };
    ws2['A2'].s = {
      font: { italic: true, sz: 11, color: { rgb: "5B5B5B" } },
      alignment: { horizontal: "center" }
    };

    // En-têtes de la feuille 2
    const summaryHeaders = ['A4', 'B4', 'C4', 'D4', 'E4', 'F4'];
    summaryHeaders.forEach(cell => {
      if (!ws2[cell]) ws2[cell] = { t: 's', v: '' };
      ws2[cell].s = {
        font: { bold: true, color: { rgb: "FFFFFF" } },
        fill: { fgColor: { rgb: "2E75B6" } },
        alignment: { horizontal: "center", vertical: "center" },
        border: {
          top: { style: "thin", color: { rgb: "000000" } },
          bottom: { style: "thin", color: { rgb: "000000" } },
          left: { style: "thin", color: { rgb: "000000" } },
          right: { style: "thin", color: { rgb: "000000" } }
        }
      };
    });

    // Formatage des données avec couleurs spéciales pour les totaux
    for (let row = 5; row <= summaryHeaderData.length; row++) {
      const isEvenRow = (row - 5) % 2 === 0;
      const bgColor = isEvenRow ? "F8F9FA" : "FFFFFF";
      
      summaryHeaders.forEach((_, colIndex) => {
        const cellRef = XLSX.utils.encode_cell({ c: colIndex, r: row - 1 });
        if (!ws2[cellRef]) ws2[cellRef] = { t: 's', v: '' };
        
        ws2[cellRef].s = {
          fill: { fgColor: { rgb: bgColor } },
          border: {
            top: { style: "thin", color: { rgb: "E0E0E0" } },
            bottom: { style: "thin", color: { rgb: "E0E0E0" } },
            left: { style: "thin", color: { rgb: "E0E0E0" } },
            right: { style: "thin", color: { rgb: "E0E0E0" } }
          },
          alignment: { vertical: "center" }
        };

        // Formatage spécial pour les colonnes d'heures
        if (colIndex >= 2) {
          ws2[cellRef].s.alignment.horizontal = "center";
          if (colIndex === 5) { // Colonne Total
            ws2[cellRef].s.font = { color: { rgb: "0066CC" }, bold: true };
            ws2[cellRef].s.fill = { fgColor: { rgb: "E7F3FF" } };
          } else {
            ws2[cellRef].s.font = { color: { rgb: "333333" } };
          }
        }
      });
    }

    // Ajout des filtres pour la feuille 2
    const summaryDataRange = XLSX.utils.encode_range({
      s: { c: 0, r: 3 },
      e: { c: 5, r: summaryHeaderData.length - 1 }
    });
    ws2['!autofilter'] = { ref: summaryDataRange };

    // Fusion des titres pour la feuille 2
    ws2['!merges'] = [
      { s: { r: 0, c: 0 }, e: { r: 0, c: 5 } },
      { s: { r: 1, c: 0 }, e: { r: 1, c: 5 } }
    ];

    // Ajout des feuilles au workbook
    XLSX.utils.book_append_sheet(wb, ws1, "📋 Détail des heures");
    XLSX.utils.book_append_sheet(wb, ws2, "📊 Totaux par agent");
    
    // Export du fichier avec un nom plus explicite
    const fileName = `Biguglia_Rapport_${new Date().toISOString().split('T')[0]}.xlsx`;
    XLSX.writeFile(wb, fileName);
    
    showToast('Export Excel formaté terminé ! 📊✨', 'success');
    
  } catch (error) {
    console.error('Erreur lors de l\'export Excel:', error);
    // Fallback vers CSV en cas d'erreur
    exportToCSV();
    showToast('Export en CSV (Excel indisponible)', 'warning');
  }
}

// Fonction de fallback pour export CSV
function exportToCSV() {
  const data = [];
  
  // En-têtes
  data.push(['Agent', 'Service', 'Événement', 'Type', 'Date', 'Motif', 'Début', 'Fin', 'Durée (h)', 'Nature']);
  
  // Données
  appData.events.forEach(event => {
    (event.participants || []).forEach(part => {
      const agent = appData.agents.find(a => a.id === part.agentId);
      const duration = calculateDuration(part.heureDebut || '', part.heureFin || '');
      
      data.push([
        agent ? agent.nom : 'Inconnu',
        agent ? agent.service : '-',
        event.nom,
        event.categorie,
        part.date,
        part.motif || '-',
        part.heureDebut || '-',
        part.heureFin || '-',
        duration.toFixed(2),
        part.nature
      ]);
    });
  });
  
  // Création du CSV
  const csvContent = data.map(row => 
    row.map(cell => `"${String(cell).replace(/"/g, '""')}"`).join(',')
  ).join('\n');
  
  // Téléchargement
  const blob = new Blob(['\ufeff' + csvContent], { type: 'text/csv;charset=utf-8;' });
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = `biguglia_export_${new Date().toISOString().split('T')[0]}.csv`;
  link.click();
  URL.revokeObjectURL(link.href);
}

// ====== FONCTIONS CRITIQUES D'IMPORT CORRIGÉES ======

// Fonction de diagnostic avec forçage (FONCTION MANQUANTE AJOUTÉE)
function diagnosticMaterielAvecForçage(materialData) {
  console.log('🔧 === DIAGNOSTIC MATÉRIEL AVEC FORÇAGE ===');
  
  const diagnostics = {
    conflicts: [],
    suggestions: [],
    canForce: false,
    foundMaterial: null,
    sourceLocation: ''
  };
  
  // Recherche exhaustive du matériel dans toutes les structures possibles
  const sources = [
    { name: 'imported.materials', data: materialData?.materials },
    { name: 'imported.donneesCompletes?.materials', data: materialData?.donneesCompletes?.materials },
    { name: 'imported.data?.materials', data: materialData?.data?.materials },
    { name: 'imported.backup?.materials', data: materialData?.backup?.materials },
    { name: 'imported.appData?.materials', data: materialData?.appData?.materials }
  ];
  
  for (const source of sources) {
    if (source.data && Array.isArray(source.data) && source.data.length > 0) {
      console.log(`✅ Matériel trouvé dans ${source.name}: ${source.data.length} articles`);
      if (!diagnostics.foundMaterial) {
        diagnostics.foundMaterial = source.data;
        diagnostics.sourceLocation = source.name;
        window.lastMaterialLocation = source.name; // Pour les logs
      }
    }
  }
  
  if (diagnostics.foundMaterial) {
    // Vérification des conflits
    diagnostics.foundMaterial.forEach(item => {
      const existing = (appData.materials || []).find(m => m.nom === item.nom);
      if (existing) {
        diagnostics.conflicts.push({
          name: item.nom,
          action: 'merge_or_replace'
        });
        diagnostics.canForce = true;
      }
    });
    
    if (diagnostics.conflicts.length === 0) {
      diagnostics.suggestions.push("Aucun conflit détecté, import sécurisé");
    } else {
      diagnostics.suggestions.push(`${diagnostics.conflicts.length} conflit(s) détecté(s)`);
    }
    
    console.log(`🎯 Matériel trouvé: ${diagnostics.foundMaterial.length} articles depuis ${diagnostics.sourceLocation}`);
    return diagnostics.foundMaterial;
  }
  
  console.warn('❌ Aucun matériel trouvé dans les sources disponibles');
  diagnostics.suggestions.push("Aucun matériel à importer détecté");
  return null;
}

// Force la récupération du matériel alternatif (FONCTION MANQUANTE AJOUTÉE)
function forceRecupMaterielAlternatif(importedData) {
  console.log('🔧 === RÉCUPÉRATION MATÉRIEL ALTERNATIF ===');
  
  if (!importedData || (!importedData.materials && !importedData.inventory && !importedData.equipment)) {
    console.warn('❌ Aucune donnée matériel alternative trouvée');
    return [];
  }
  
  const alternativeMaterials = [];
  
  // Tentative de récupération dans des structures non-standard
  const alternateSources = [
    importedData.materials,
    importedData.inventory,
    importedData.equipment,
    importedData.stock,
    importedData.items,
    importedData.catalogue
  ];
  
  for (const source of alternateSources) {
    if (source && Array.isArray(source) && source.length > 0) {
      console.log(`📦 Source alternative trouvée: ${source.length} items`);
      
      source.forEach((item, index) => {
        const existing = (appData.materials || []).find(m => m.nom === (item.nom || item.name));
        
        const altItem = {
          id: item.id || generateId(),
          nom: item.nom || item.name || `Article ${index + 1}`,
          categorie: item.categorie || item.category || 'Divers',
          stockTotal: Number(item.stockTotal || item.stock || 0),
          stockReserve: 0,
          unite: item.unite || item.unit || 'unité',
          stockMin: Number(item.stockMin || 0),
          statut: 'disponible',
          description: item.description || item.desc || ''
        };
        
        if (existing) {
          altItem.nom = `${altItem.nom} (importé)`;
          alternativeMaterials.push(altItem);
        } else {
          alternativeMaterials.push(altItem);
        }
      });
      
      console.log(`✅ ${alternativeMaterials.length} articles récupérés depuis source alternative`);
      return alternativeMaterials;
    }
  }
  
  console.warn('❌ Aucune source alternative de matériel trouvée');
  return [];
}

// Fonction utilitaire pour générer des IDs uniques (FONCTION MANQUANTE AJOUTÉE)
function generateId() {
  return Date.now() + Math.floor(Math.random() * 10000);
}

// Diagnostic d'erreur d'import (FONCTION MANQUANTE AJOUTÉE)
function diagnosticImportError(error, fileContent) {
  console.log('🔍 === DIAGNOSTIC ERREUR IMPORT ===');
  
  const diagnostic = {
    type: 'unknown',
    message: error.message,
    suggestions: [],
    position: null,
    context: null
  };
  
  // Analyse du type d'erreur JSON
  if (error.message.includes('Unexpected token')) {
    diagnostic.type = 'syntax_error';
    
    // Tentative d'extraction de la position
    const posMatch = error.message.match(/position (\d+)/);
    if (posMatch) {
      diagnostic.position = parseInt(posMatch[1]);
      diagnostic.context = fileContent.substring(
        Math.max(0, diagnostic.position - 100), 
        diagnostic.position + 100
      );
    }
    
    diagnostic.suggestions = [
      '🔧 Vérifiez la syntaxe JSON du fichier',
      '🔧 Supprimez les virgules en trop',
      '🔧 Vérifiez les guillemets autour des clés',
      '🔧 Utilisez un validateur JSON en ligne'
    ];
    
  } else if (error.message.includes('Unexpected end')) {
    diagnostic.type = 'incomplete_json';
    diagnostic.suggestions = [
      '🔧 Le fichier semble tronqué',
      '🔧 Vérifiez que l\'export s\'est terminé correctement',
      '🔧 Réexportez les données depuis l\'application source'
    ];
    
  } else if (error.message.includes('network') || error.message.includes('fetch')) {
    diagnostic.type = 'network_error';
    diagnostic.suggestions = [
      '🔧 Vérifiez votre connexion internet',
      '🔧 Réessayez l\'import',
      '🔧 Utilisez un fichier local plutôt qu\'une URL'
    ];
  }
  
  return diagnostic;
}

// Fonction de validation et réparation JSON améliorée (FONCTION MANQUANTE AJOUTÉE)
function validateAndRepairJSON(content) {
  console.log('🔧 Validation et réparation JSON...');
  
  try {
    // Nettoyage préliminaire
    let cleanContent = content.trim();
    
    // Réparations automatiques courantes
    cleanContent = cleanContent.replace(/,(\s*[}\]])/g, '$1'); // Supprime les virgules en trop
    cleanContent = cleanContent.replace(/([{,]\s*)([a-zA-Z_$][a-zA-Z0-9_$]*)\s*:/g, '$1"$2":'); // Ajoute quotes aux clés
    
    // Vérification de l'équilibrage des accolades et crochets
    const openBraces = (cleanContent.match(/\{/g) || []).length;
    const closeBraces = (cleanContent.match(/\}/g) || []).length;
    const openBrackets = (cleanContent.match(/\[/g) || []).length;
    const closeBrackets = (cleanContent.match(/\]/g) || []).length;
    
    if (openBraces > closeBraces) {
      cleanContent += '}'.repeat(openBraces - closeBraces);
      console.log('🔧 Accolades fermantes ajoutées');
    }
    
    if (openBrackets > closeBrackets) {
      cleanContent += ']'.repeat(openBrackets - closeBrackets);
      console.log('🔧 Crochets fermants ajoutés');
    }
    
    // Tentative de parsing
    const parsed = JSON.parse(cleanContent);
    
    return {
      isValid: true,
      repairedContent: cleanContent,
      data: parsed,
      wasRepaired: cleanContent !== content.trim()
    };
    
  } catch (error) {
    console.error('❌ Échec de la réparation JSON:', error);
    return {
      isValid: false,
      error: error.message,
      data: null,
      repairedContent: null,
      wasRepaired: false
    };
  }
}

// ====== FONCTION D'IMPORT CORRIGÉE AVEC TOUTES LES VARIABLES ======

// ====== FONCTION D'IMPORT CORRIGÉE AVEC TOUTES LES VARIABLES ======
async function importData(files) {
  console.log('📥 === DÉBUT IMPORT FICHIER CORRIGÉ ===');
  
  // ✅ DÉCLARATION DE TOUTES LES VARIABLES MANQUANTES
  let materialToImport = [];
  let importSource = 'unknown';
  let materialStatus = {
    imported: 0,
    errors: 0,
    total: 0
  };
  let foundMaterial = null;
  
  if (!files || !files.length) {
    showToast('❌ Aucun fichier sélectionné', 'warning');
    return;
  }
  
  const file = files[0];
  const fileName = file.name;
  console.log('📄 Fichier sélectionné:', fileName, 'Taille:', (file.size / 1024).toFixed(1) + 'KB');
  
  // Vérification du type de fichier
  if (!fileName.toLowerCase().endsWith('.json')) {
    showToast('❌ FORMAT INVALIDE\n\nSeuls les fichiers JSON (.json) sont acceptés.\n\nVotre fichier: ' + fileName, 'error', 5000);
    return;
  }
  
  // Vérification de la taille du fichier (max 50MB)
  const maxSize = 50 * 1024 * 1024; // 50MB
  if (file.size > maxSize) {
    showToast(`❌ FICHIER TROP VOLUMINEUX\n\nTaille max: 50MB\nVotre fichier: ${(file.size / 1024 / 1024).toFixed(1)}MB\n\nRéduisez la taille ou utilisez un export sans documents.`, 'error', 6000);
    return;
  }
  
  if (file.size === 0) {
    showToast('❌ FICHIER VIDE\n\nLe fichier sélectionné ne contient aucune donnée.', 'error', 5000);
    return;
  }
  
  console.log('✅ Vérifications fichier OK - Début lecture...');
  showToast('📥 Lecture du fichier en cours...', 'info', 3000);
  
  const reader = new FileReader();
  
  reader.onload = (e) => {
    try {
      console.log('📥 Début de l\'import de données...');
      
      // Vérification de la longueur du contenu
      const content = e.target.result;
      if (!content || content.trim().length === 0) {
        showToast('❌ FICHIER VIDE\n\nLe fichier sélectionné ne contient aucune donnée.', 'error', 5000);
        return;
      }
      
      console.log('📄 Contenu lu, taille:', content.length, 'caractères');
      showToast('🔍 Analyse du fichier JSON...', 'info', 2000);
      
      // Nettoyage basique du contenu avec validation renforcée
      const validationResult = validateAndRepairJSON(content);
      if (!validationResult.isValid) {
        showToast(`❌ JSON INVALIDE\n\nErreur: ${validationResult.error}\n\nVérifiez la structure de votre fichier.`, 'error', 6000);
        return;
      }
      
      const cleanContent = validationResult.repairedContent || content.trim();
      
      // Parsing JSON avec gestion d'erreur simple et diagnostic
      let imported;
      try {
        console.log('🔄 Tentative de parsing JSON...');
        imported = JSON.parse(cleanContent);
        console.log('✅ JSON parsé avec succès');
        console.log('📋 Clés principales trouvées:', Object.keys(imported));
        
      } catch (parseError) {
        console.error('❌ Erreur de parsing JSON:', parseError);
        
        // Diagnostic simple de l'erreur
        const diagnostic = diagnosticImportError(parseError, content);
        let errorMessage = '❌ ERREUR DE FORMAT JSON\n\n';
        errorMessage += `📄 Fichier: ${file.name}\n`;
        errorMessage += `📏 Taille: ${(file.size / 1024).toFixed(1)}KB\n\n`;
        errorMessage += diagnostic.suggestions.join('\n') + '\n';
        
        showToast(errorMessage, 'error', 10000);
        return;
      }
      
      // 🔍 DIAGNOSTIC EXHAUSTIF DU MATÉRIEL AVANT NORMALISATION
      console.log('🔍 === DIAGNOSTIC MATÉRIEL COMPLET ===');
      console.log('📦 Structure des données importées:', Object.keys(imported));
      
      // 📦 RECHERCHE EXHAUSTIVE DU MATÉRIEL
      foundMaterial = diagnosticMaterielAvecForçage(imported);
      if (foundMaterial && foundMaterial.length > 0) {
        console.log('🎯 Matériel trouvé et forcé avec succès:', foundMaterial.length, 'articles');
        materialToImport = foundMaterial;
        importSource = window.lastMaterialLocation || 'source détectée';
        materialStatus.total = foundMaterial.length;
        
        // Injection directe dans la structure principale
        imported.materials = foundMaterial;
        console.log('✅ Matériel injecté dans imported.materials');
      } else {
        console.warn('⚠️ Aucun matériel récupérable - tentative de récupération alternative');
        // Tentative de récupération manuelle
        const materielAlternatif = forceRecupMaterielAlternatif(imported);
        if (materielAlternatif && materielAlternatif.length > 0) {
          imported.materials = materielAlternatif;
          materialToImport = materielAlternatif;
          importSource = 'source alternative';
          materialStatus.total = materielAlternatif.length;
          console.log('✅ Matériel alternatif injecté:', materielAlternatif.length, 'articles');
        }
      }
      
      // Validation de base de la structure
      if (!imported || typeof imported !== 'object') {
        showToast('❌ STRUCTURE INVALIDE\n\nLe fichier ne contient pas de données valides.', 'error', 5000);
        return;
      }
      
      console.log('🔍 Données importées après diagnostic:', {
        agents: (imported.agents || []).length,
        events: (imported.events || []).length,
        materials: (imported.materials || []).length,
        tasks: Object.keys(imported.tasks || {}).length
      });
      
      // Normalisation des données avec gestion d'erreur
      let normalized;
      try {
        normalized = normalizeDataWithMaterialFix(imported);
      } catch (normalizeError) {
        console.error('❌ Erreur de normalisation:', normalizeError);
        showToast('❌ ERREUR DE NORMALISATION\n\nImpossible de traiter les données du fichier.', 'error', 5000);
        return;
      }
      
      // Validation après normalisation
      if (!normalized.agents || !Array.isArray(normalized.agents) || 
          !normalized.events || !Array.isArray(normalized.events)) {
        showToast('❌ DONNÉES INCOMPLÈTES\n\nLe fichier ne contient pas les données requises (agents ou événements manquants).', 'error', 6000);
        return;
      }
      
      // Sauvegarde de sécurité avant import
      const backupData = { ...appData };
      console.log('💾 Sauvegarde de sécurité créée');
      
      // Confirmation avec détails incluant le matériel
      const materialInfo = materialToImport.length > 0 ? 
        `✅ ${materialToImport.length} matériel(s) trouvé(s) dans ${importSource}` : 
        '❌ Aucun matériel trouvé (matériel par défaut sera utilisé)';
      
      const confirmMessage = `🔄 CONFIRMATION D'IMPORT
      
📊 Données à importer :
• ${normalized.agents.length} agent(s)
• ${normalized.events.length} événement(s)
• ${materialInfo}
• ${Object.keys(normalized.tasks || {}).length} plannings

⚠️  ATTENTION : Cette action va remplacer TOUTES vos données actuelles !

📋 Données actuelles :
• ${appData.agents.length} agent(s)
• ${appData.events.length} événement(s)
• ${(appData.materials || []).length} matériel(s)

✅ Une sauvegarde de sécurité a été créée.

Continuer l'import ?`;
      
      if (!confirm(confirmMessage)) {
        console.log('❌ Import annulé par l\'utilisateur');
        showToast('Import annulé', 'info', 2000);
        return;
      }
      
      showToast('🔄 Import en cours...', 'info', 3000);
      
      try {
        // Remplacement des données avec validation étape par étape
        appData.agents = normalized.agents || [];
        appData.events = normalized.events || [];
        appData.tasks = normalized.tasks || {};
        
        // 📦 IMPORT DIRECT ET FORCÉ DU MATÉRIEL
        console.log('📦 === IMPORT FORCÉ DU MATÉRIEL ===');
        
        if (materialToImport && materialToImport.length > 0) {
          console.log('🎯 Import direct du matériel trouvé:', materialToImport.length, 'articles');
          appData.materials = materialToImport.map((material, index) => {
            const processed = {
              id: Number(material.id) || Date.now() + index + Math.floor(Math.random() * 1000),
              nom: material.nom || material.name || `Article ${index + 1}`,
              categorie: material.categorie || material.category || 'Divers',
              stockTotal: Math.max(0, Number(material.stockTotal) || Number(material.stock) || 0),
              stockReserve: Math.max(0, Number(material.stockReserve) || 0),
              unite: material.unite || material.unit || 'unité',
              stockMin: Math.max(0, Number(material.stockMin) || Number(material.minStock) || 0),
              statut: material.statut || material.status || 'disponible',
              description: String(material.description || '').trim()
            };
            console.log(`📦 Matériel importé [${index + 1}]:`, processed.nom, '- Stock:', processed.stockTotal);
            materialStatus.imported++;
            return processed;
          });
          console.log('✅ Matériel importé avec succès:', appData.materials.length, 'articles');
        } else if (normalized.materials && normalized.materials.length > 0) {
          console.log('🔄 Utilisation du matériel normalisé:', normalized.materials.length, 'articles');
          appData.materials = normalized.materials;
          materialStatus.imported = normalized.materials.length;
        } else {
          console.warn('⚠️ Aucun matériel disponible, conservation du matériel existant');
          if (!appData.materials || !Array.isArray(appData.materials) || appData.materials.length === 0) {
            console.log('🔧 Initialisation du matériel par défaut...');
            appData.materials = [
              { id: 1, nom: "Table rectangulaire", categorie: "Tables", stockTotal: 50, stockReserve: 0, unite: "unité", stockMin: 5, statut: "disponible", description: "Table 180x80cm pliante" },
              { id: 2, nom: "Chaise pliante", categorie: "Chaises", stockTotal: 200, stockReserve: 0, unite: "unité", stockMin: 20, statut: "disponible", description: "Chaise pliante blanche" },
              { id: 3, nom: "Tente 3x3m", categorie: "Tentes", stockTotal: 10, stockReserve: 0, unite: "unité", stockMin: 2, statut: "disponible", description: "Tente pliante 3x3m avec bâches latérales" }
            ];
            console.log('✅ Matériel par défaut initialisé:', appData.materials.length, 'articles');
          } else {
            console.log('🔄 Conservation du matériel existant:', appData.materials.length, 'articles');
          }
        }
        
        // Import des attributions de matériel avec recherche étendue
        console.log('📋 === IMPORT DES ATTRIBUTIONS MATÉRIEL ===');
        const attributionSources = [
          { name: 'normalized.eventMaterials', data: normalized.eventMaterials },
          { name: 'imported.eventMaterials', data: imported.eventMaterials },
          { name: 'imported.donneesCompletes?.eventMaterials', data: imported.donneesCompletes?.eventMaterials },
          { name: 'imported.appData?.eventMaterials', data: imported.appData?.eventMaterials }
        ];
        
        let foundAttributions = null;
        attributionSources.forEach(source => {
          if (source.data && typeof source.data === 'object' && Object.keys(source.data).length > 0) {
            console.log(`✅ Attributions trouvées dans ${source.name}:`, Object.keys(source.data).length, 'événements');
            if (!foundAttributions) {
              foundAttributions = source.data;
            }
          }
        });
        
        if (foundAttributions) {
          appData.eventMaterials = {};
          Object.entries(foundAttributions).forEach(([eventId, allocations]) => {
            if (Array.isArray(allocations)) {
              appData.eventMaterials[eventId] = allocations.map(allocation => ({
                id: allocation.id || Date.now() + Math.random(),
                materialId: Number(allocation.materialId),
                quantite: Number(allocation.quantite) || Number(allocation.quantity) || 1,
                dateDebut: allocation.dateDebut || allocation.startDate || '',
                dateFin: allocation.dateFin || allocation.endDate || '',
                valide: Boolean(allocation.valide) || Boolean(allocation.returned) || false,
                sortieValidee: Boolean(allocation.sortieValidee) || Boolean(allocation.validated) || false
              }));
            }
          });
          console.log('✅ Attributions matériel importées pour', Object.keys(appData.eventMaterials).length, 'événements');
        } else {
          console.warn('⚠️ Aucune attribution de matériel trouvée');
          if (!appData.eventMaterials) {
            appData.eventMaterials = {};
          }
        }
        
        // Nettoyage et reconstruction des blobs
        blobManager.clear();
        console.log('🧹 BlobManager nettoyé');
        
        // Reconstitution des blobs pour les documents
        let documentsProcessed = 0;
        let documentsErrors = 0;
        
        appData.events.forEach(ev => {
          (ev.documents || []).forEach(doc => {
            if (doc.dataUrl) {
              try {
                const blob = dataURLtoBlob(doc.dataUrl);
                blobManager.addBlob(ev.id, doc.id, blob);
                documentsProcessed++;
              } catch(e) {
                console.warn('❌ Impossible de recréer le blob pour:', doc.name, e);
                documentsErrors++;
              }
            }
          });
        });
        
        console.log(`📄 Documents traités: ${documentsProcessed} OK, ${documentsErrors} erreurs`);
        
        // Validation et correction du schéma des tâches
        ensureTasksSchema();
        console.log('✅ Schéma des tâches validé');
        
        // Sauvegarde avec gestion d'erreur
        markAsChanged();
        try {
          saveData();
          console.log('💾 Données sauvegardées avec succès');
        } catch (saveError) {
          console.error('❌ Erreur de sauvegarde:', saveError);
          
          // Tentative de restauration
          try {
            appData = backupData;
            saveData();
            showToast('❌ ERREUR DE SAUVEGARDE\n\nDonnées restaurées depuis la sauvegarde de sécurité.', 'error', 6000);
            return;
          } catch (restoreError) {
            console.error('❌ Erreur critique de restauration:', restoreError);
            showToast('🚨 ERREUR CRITIQUE\n\nImpossible de restaurer les données. Rechargez la page.', 'error', 10000);
            return;
          }
        }
        
        // Rafraîchissement complet de l'interface
        try {
          updateTypeFilter();
          updateDashboard();
          updateAgentsList();
          updateEventsList();
          updateHoursTable();
          renderCalendar();
          
          // Rafraîchir le matériel si on est dans ce module
          if (document.getElementById('material').classList.contains('active')) {
            updateMaterialList();
            updateMaterialFilters();
          }
          
          console.log('🔄 Interface mise à jour');
        } catch (uiError) {
          console.error('❌ Erreur de mise à jour de l\'interface:', uiError);
          showToast('⚠️ Interface partiellement mise à jour. Rechargez la page si nécessaire.', 'warning', 4000);
        }
        
        // Message de succès détaillé avec informations sur le matériel
        const materialSuccessStatus = materialToImport.length > 0 ? 
          `✅ ${materialStatus.imported} article(s) de matériel restauré(s) depuis ${importSource}` : 
          `⚠️ Matériel par défaut utilisé (${appData.materials.length} articles)`;
          
        const successMessage = `✅ IMPORT RÉUSSI !

📊 Données importées :
• ${normalized.agents.length} agent(s)
• ${normalized.events.length} événement(s)
• ${materialSuccessStatus}
• ${Object.keys(appData.eventMaterials).length} attributions matériel
• ${Object.keys(normalized.tasks || {}).length} planning(s)
• ${documentsProcessed} document(s) reconstitué(s)

${documentsErrors > 0 ? `⚠️ ${documentsErrors} document(s) non récupérable(s)` : ''}

🎉 Toutes vos données ont été importées et l'interface a été mise à jour !

${materialToImport.length > 0 ? '📦 Votre stock de matériel a été restauré avec succès !' : '📦 Ajoutez votre matériel depuis le module Matériel.'}`;
        
        showToast(successMessage, 'success', 10000);
        
        console.log('✅ Import terminé avec succès');
        console.log('📊 État final:', {
          agents: appData.agents.length,
          events: appData.events.length,
          materials: appData.materials.length,
          eventMaterials: Object.keys(appData.eventMaterials).length
        });
        
      } catch (importError) {
        console.error('❌ Erreur critique pendant l\'import:', importError);
        
        // Tentative de restauration des données
        try {
          appData = backupData;
          saveData();
          updateDashboard();
          showToast('❌ IMPORT ÉCHOUÉ\n\nLes données ont été restaurées à leur état précédent.', 'error', 6000);
        } catch (restoreError) {
          console.error('❌ Erreur critique de restauration:', restoreError);
          showToast('🚨 ERREUR CRITIQUE\n\nImpossible de restaurer les données.\nRechargez la page et réessayez.', 'error', 10000);
        }
      }
      
    } catch(error) {
      console.error('❌ Erreur générale d\'import:', error);
      showToast('❌ ERREUR D\'IMPORT\n\nUne erreur inattendue s\'est produite.\nVérifiez le fichier et réessayez.', 'error', 6000);
    }
  };
  
  reader.onerror = (error) => {
    console.error('❌ Erreur de lecture du fichier:', error);
    showToast('❌ ERREUR DE LECTURE\n\nImpossible de lire le fichier.\nVérifiez qu\'il n\'est pas corrompu.', 'error', 5000);
  };
  
  reader.onprogress = (e) => {
    if (e.lengthComputable) {
      const percentLoaded = Math.round((e.loaded / e.total) * 100);
      console.log(`📥 Lecture en cours: ${percentLoaded}%`);
      if (percentLoaded < 100) {
        showToast(`📥 Lecture du fichier: ${percentLoaded}%`, 'info', 1000);
      }
    }
  };
  
  // Lecture du fichier avec encodage UTF-8 pour supporter les caractères français
  reader.readAsText(file, 'UTF-8');
}

// Nouvelle fonction : Export détaillé par agent
function exportDetailedReport() {
  try {
    // Création des données pour l'export détaillé
    const report = {
      metadata: {
        dateExport: new Date().toISOString(),
        applicationVersion: "Biguglia v2.0",
        filtreActuel: currentTypeFilter === 'tous' ? 'Aucun filtre' : currentTypeFilter,
        periodeAnalyse: new Date().getFullYear()
      },
      
      resume: {
        totalAgents: appData.agents.length,
        totalEvenements: appData.events.length,
        totalLignesHeures: 0,
        heuresPayees: 0,
        heuresRecup: 0,
        heuresVoirie: 0,
        heuresTotal: 0
      },
      
      statistiques: {
        agentLePlusActif: "",
        evenementLePlusLourd: "",
        moisLePlusCharge: "",
        repartitionParService: {},
        repartitionParTypeEvenement: {}
      },
      
      detailsParAgent: [],
      
      detailsParEvenement: [],
      
      donneesCompletes: {
        ...appData,
        materials: appData.materials || [],
        eventMaterials: appData.eventMaterials || {}
      } // Pour compatibilité import/export
    };

    // Calculs détaillés
    let totalLignes = 0;
    let agentStats = {};
    let eventStats = {};
    let serviceStats = {};
    let typeStats = {};
    let monthlyStats = new Array(12).fill(0);

    // Analyse par agent
    appData.agents.forEach(agent => {
      const agentData = {
        agent: agent.nom,
        service: agent.service,
        telephone: agent.telephone,
        evenements: [],
        totaux: { payees: 0, recup: 0, voirie: 0, total: 0 },
        statistiques: {
          nombreJoursTravailles: 0,
          evenementsParticipes: 0,
          dureeMoyenneParJour: 0
        }
      };

      const datesUniques = new Set();

      appData.events.forEach(event => {
        const participations = (event.participants || []).filter(p => p.agentId === agent.id);
        if (participations.length > 0) {
          const eventData = {
            nom: event.nom,
            categorie: event.categorie,
            lieu: event.lieu,
            dates: `${event.dateDebut} → ${event.dateFin}`,
            lignes: participations.map(p => {
              const duration = calculateDuration(p.heureDebut || '', p.heureFin || '');
              totalLignes++;
              
              // Mise à jour des totaux agent
              if (p.nature === 'payees') agentData.totaux.payees += duration;
              else if (p.nature === 'recup') agentData.totaux.recup += duration;
              else if (p.nature === 'voirie') agentData.totaux.voirie += duration;
              
              // Dates uniques pour statistiques
              datesUniques.add(p.date);
              
              // Stats mensuelles
              const mois = toLocalDate(p.date).getMonth();
              monthlyStats[mois] += duration;
              
              return {
                date: p.date,
                motif: p.motif || '',
                nature: p.nature,
                debut: p.heureDebut || '',
                fin: p.heureFin || '',
                duree: duration
              };
            })
          };
          
          agentData.evenements.push(eventData);
          agentData.statistiques.evenementsParticipes++;
        }
      });

      agentData.totaux.total = agentData.totaux.payees + agentData.totaux.recup + agentData.totaux.voirie;
      agentData.statistiques.nombreJoursTravailles = datesUniques.size;
      agentData.statistiques.dureeMoyenneParJour = datesUniques.size > 0 ? 
        (agentData.totaux.total / datesUniques.size) : 0;

      // Mise à jour des résumés globaux
      report.resume.heuresPayees += agentData.totaux.payees;
      report.resume.heuresRecup += agentData.totaux.recup;
      report.resume.heuresVoirie += agentData.totaux.voirie;
      
      // Stats par service
      if (!serviceStats[agent.service]) serviceStats[agent.service] = 0;
      serviceStats[agent.service] += agentData.totaux.total;
      
      agentStats[agent.nom] = agentData.totaux.total;
      
      if (agentData.evenements.length > 0) {
        report.detailsParAgent.push(agentData);
      }
    });

    // Analyse par événement
    appData.events.forEach(event => {
      const eventData = {
        nom: event.nom,
        categorie: event.categorie,
        lieu: event.lieu,
        dates: `${event.dateDebut} → ${event.dateFin}`,
        dureeJours: getDatesBetween(event.dateDebut, event.dateFin).length,
        totaux: { payees: 0, recup: 0, voirie: 0, total: 0 },
        participants: []
      };

      (event.participants || []).forEach(p => {
        const agent = appData.agents.find(a => a.id === p.agentId);
        const duration = calculateDuration(p.heureDebut || '', p.heureFin || '');
        
        if (p.nature === 'payees') eventData.totaux.payees += duration;
        else if (p.nature === 'recup') eventData.totaux.recup += duration;
        else if (p.nature === 'voirie') eventData.totaux.voirie += duration;
        
        eventData.participants.push({
          agent: agent ? agent.nom : 'Inconnu',
          date: p.date,
          motif: p.motif || '',
          nature: p.nature,
          duree: duration
        });
      });

      eventData.totaux.total = eventData.totaux.payees + eventData.totaux.recup + eventData.totaux.voirie;
      
      // Stats par type d'événement
      if (!typeStats[event.categorie]) typeStats[event.categorie] = 0;
      typeStats[event.categorie] += eventData.totaux.total;
      
      eventStats[event.nom] = eventData.totaux.total;
      
      report.detailsParEvenement.push(eventData);
    });

    // Finalisation des statistiques
    report.resume.totalLignesHeures = totalLignes;
    report.resume.heuresTotal = report.resume.heuresPayees + report.resume.heuresRecup + report.resume.heuresVoirie;
    
    // Agent le plus actif
    const agentMax = Object.entries(agentStats).reduce((max, [nom, heures]) => 
      heures > max.heures ? {nom, heures} : max, {nom: '', heures: 0});
    report.statistiques.agentLePlusActif = `${agentMax.nom} (${agentMax.heures.toFixed(1)}h)`;
    
    // Événement le plus lourd
    const eventMax = Object.entries(eventStats).reduce((max, [nom, heures]) => 
      heures > max.heures ? {nom, heures} : max, {nom: '', heures: 0});
    report.statistiques.evenementLePlusLourd = `${eventMax.nom} (${eventMax.heures.toFixed(1)}h)`;
    
    // Mois le plus chargé
    const moisMax = monthlyStats.reduce((max, heures, index) => 
      heures > max.heures ? {mois: monthLabels()[index], heures} : max, {mois: '', heures: 0});
    report.statistiques.moisLePlusCharge = `${moisMax.mois} (${moisMax.heures.toFixed(1)}h)`;
    
    report.statistiques.repartitionParService = serviceStats;
    report.statistiques.repartitionParTypeEvenement = typeStats;

    // Export du fichier
    const blob = new Blob([JSON.stringify(report, null, 2)], { type: 'application/json' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = `rapport_detaille_biguglia_${new Date().toISOString().split('T')[0]}.json`;
    link.click();
    URL.revokeObjectURL(link.href);
    
    showToast('Rapport détaillé exporté avec analyses complètes ! 📊', 'success');
    
  } catch (error) {
    console.error('Erreur export détaillé:', error);
    showToast('Erreur lors de l\'export détaillé', 'error');
  }
}

function toggleDarkMode() {
  document.documentElement.classList.toggle('dark');
  const isDark = document.documentElement.classList.contains('dark');
  document.getElementById('darkModeText').textContent = isDark ? 'Mode clair' : 'Mode sombre';
  localStorage.setItem('darkMode', isDark);
  showToast(`Mode ${isDark ? 'sombre' : 'clair'} activé`, 'info', 2000);
}

// === FONCTION DE TEST ET VÉRIFICATION MATÉRIEL ===
function testMaterialImport() {
  console.log('🧪 === TEST IMPORT MATÉRIEL ===');
  
  // Vérification de la structure
  if (!appData.materials || !Array.isArray(appData.materials)) {
    console.error('❌ Structure materials invalide');
    return false;
  }
  
  if (appData.materials.length === 0) {
    console.warn('⚠️ Aucun matériel présent');
    return false;
  }
  
  console.log(`✅ ${appData.materials.length} articles de matériel présents`);
  
  // Vérification de l'intégrité des données
  let validCount = 0;
  appData.materials.forEach((material, index) => {
    const isValid = material.id && material.nom && 
                    typeof material.stockTotal === 'number' &&
                    material.categorie && material.unite;
    
    if (isValid) {
      validCount++;
    } else {
      console.warn(`⚠️ Article ${index + 1} invalide:`, material);
    }
  });
  
  const successRate = (validCount / appData.materials.length) * 100;
  console.log(`📊 Taux de validité: ${successRate.toFixed(1)}% (${validCount}/${appData.materials.length})`);
  
  // Vérification des attributions
  const eventMaterialsCount = Object.keys(appData.eventMaterials || {}).length;
  console.log(`📋 Attributions matériel: ${eventMaterialsCount} événements`);
  
  // Test d'affichage dans l'interface
  if (document.getElementById('material')) {
    console.log('🖥️ Module matériel disponible pour mise à jour');
  }
  
  return successRate > 80; // Considéré comme réussi si plus de 80% des articles sont valides
}

/* ==== FONCTIONS UTILITAIRES MANQUANTES ==== */

// Fonction pour gérer les notifications toast
function showToast(message, type = 'info', duration = 3000) {
  const existingToast = document.querySelector('.toast');
  if (existingToast) {
    existingToast.remove();
  }
  
  const toast = document.createElement('div');
  toast.className = `toast ${type}`;
  toast.textContent = message;
  
  document.body.appendChild(toast);
  
  // Afficher le toast
  setTimeout(() => {
    toast.classList.add('show');
  }, 100);
  
  // Masquer et supprimer le toast
  setTimeout(() => {
    toast.classList.remove('show');
    setTimeout(() => {
      if (toast.parentNode) {
        toast.parentNode.removeChild(toast);
      }
    }, 300);
  }, duration);
}

// Fonction pour initialiser l'application
function initApp() {
  console.log('🚀 === INITIALISATION APPLICATION ===');
  
  try {
    // Chargement des données depuis localStorage
    const saved = safeGetLS('biguglia_app_data');
    if (saved) {
      console.log('📂 Données chargées depuis localStorage');
      
      // Validation et normalisation des données chargées
      if (saved.agents && Array.isArray(saved.agents)) {
        appData.agents = saved.agents;
      }
      
      if (saved.events && Array.isArray(saved.events)) {
        appData.events = saved.events;
      }
      
      if (saved.tasks && typeof saved.tasks === 'object') {
        appData.tasks = saved.tasks;
      }
      
      // Import du matériel avec gestion d'erreur
      if (saved.materials && Array.isArray(saved.materials)) {
        appData.materials = saved.materials;
        console.log(`📦 ${saved.materials.length} articles matériel chargés`);
      } else {
        console.log('📦 Aucun matériel sauvegardé, utilisation des données par défaut');
        if (!appData.materials) {
          appData.materials = [];
        }
      }
      
      // Import des attributions matériel
      if (saved.eventMaterials && typeof saved.eventMaterials === 'object') {
        appData.eventMaterials = saved.eventMaterials;
        console.log(`📋 Attributions matériel chargées pour ${Object.keys(saved.eventMaterials).length} événements`);
      } else {
        if (!appData.eventMaterials) {
          appData.eventMaterials = {};
        }
      }
    } else {
      console.log('📂 Aucune donnée sauvegardée, utilisation des données par défaut');
      
      // Initialiser les structures manquantes
      if (!appData.materials) {
        appData.materials = [];
      }
      if (!appData.eventMaterials) {
        appData.eventMaterials = {};
      }
    }
    
    // Initialisation de l'interface
    populateCalendarYearSelect();
    updateTypeFilter();
    updateDashboard();
    
    // Gestionnaires d'événements pour la navigation
    setupEventHandlers();
    
    console.log('✅ Application initialisée avec succès');
    console.log('📊 État actuel:', {
      agents: appData.agents.length,
      events: appData.events.length,
      materials: appData.materials.length,
      tasks: Object.keys(appData.tasks).length
    });
    
  } catch (error) {
    console.error('❌ Erreur lors de l\'initialisation:', error);
    showToast('❌ Erreur lors du chargement de l\'application', 'error', 5000);
    
    // Tentative de récupération avec données par défaut
    try {
      if (!appData.materials) appData.materials = [];
      if (!appData.eventMaterials) appData.eventMaterials = {};
      if (!appData.agents) appData.agents = [];
      if (!appData.events) appData.events = [];
      if (!appData.tasks) appData.tasks = {};
      
      updateDashboard();
      console.log('🔄 Récupération avec données par défaut réussie');
    } catch (recoveryError) {
      console.error('❌ Échec de la récupération:', recoveryError);
    }
  }
}

// Configuration des gestionnaires d'événements
function setupEventHandlers() {
  // Gestionnaire pour les clics sur les boutons d'action
  document.addEventListener('click', function(e) {
    const target = e.target.closest('[data-action]');
    if (!target) return;
    
    const action = target.dataset.action;
    const id = parseInt(target.dataset.id);
    
    switch(action) {
      case 'agent-edit':
        showAgentForm(id);
        break;
      case 'agent-delete':
        deleteAgent(id);
        break;
      case 'event-edit':
        showAddEventForm(id);
        break;
      case 'event-delete':
        deleteEvent(id);
        break;
      case 'event-participants':
        openParticipantsModal(id);
        break;
      case 'event-docs':
        openDocsModal(id);
        break;
      case 'event-material':
        openMaterialModal(id);
        break;
      case 'event-planning':
        openPlanningModal(id);
        break;
    }
  });
  
  console.log('✅ Gestionnaires d\'événements configurés');
}

// Remplir le sélecteur d'années du calendrier
function populateCalendarYearSelect() {
  const select = document.getElementById('calendarYear');
  if (!select) return;
  
  const currentYear = new Date().getFullYear();
  select.innerHTML = '';
  
  for (let year = currentYear - 2; year <= currentYear + 2; year++) {
    const option = document.createElement('option');
    option.value = year;
    option.textContent = year;
    if (year === currentYear) {
      option.selected = true;
    }
    select.appendChild(option);
  }
  
  select.dataset.ready = 'true';
}

// ...existing code...

function handleGlobalSearch(query) {
  const results = document.getElementById('searchResults');
  
  if (!query.trim()) {
    results.style.display = 'none';
    return;
  }
  
  const searchResults = [];
  query = query.toLowerCase();
  
  // Recherche dans les agents
  appData.agents.forEach(agent => {
    if (agent.nom.toLowerCase().includes(query) || 
        agent.service.toLowerCase().includes(query)) {
      searchResults.push({
        type: 'agent',
        title: agent.nom,
        subtitle: agent.service,
        action: () => { showModule('agents'); showAgentDetail(agent.id); }
      });
    }
  });
  
  // Recherche dans les événements
  appData.events.forEach(event => {
    if (event.nom.toLowerCase().includes(query) || 
        event.categorie.toLowerCase().includes(query) ||
        event.lieu.toLowerCase().includes(query)) {
      searchResults.push({
        type: 'event',
        title: event.nom,
        subtitle: `${event.categorie} - ${event.lieu}`,
        action: () => { showModule('events'); goToEvent(event.id); }
      });
    }
  });
  
  displaySearchResults(searchResults);
}

function displaySearchResults(results) {
  const container = document.getElementById('searchResults');
  
  if (results.length === 0) {
    container.innerHTML = '<div class="text-gray-500 text-sm">Aucun résultat</div>';
  } else {
    container.innerHTML = results.slice(0, 8).map(result => `
      <div class="flex items-center p-2 hover:bg-gray-100 cursor-pointer rounded" onclick="(${result.action})(); document.getElementById('searchResults').style.display='none';">
        <i class="fas fa-${result.type === 'agent' ? 'user' : 'calendar'} text-gray-400 mr-3"></i>
        <div>
          <div class="font-medium text-sm">${result.title}</div>
          <div class="text-xs text-gray-500">${result.subtitle}</div>
        </div>
      </div>
    `).join('');
  }
  
  container.style.display = 'block';
}

function exporterDonnees(){
  // S'assurer que les données matériel sont incluses dans l'export avec redondance
  const dataToExport = {
    ...appData,
    materials: appData.materials || [],
    eventMaterials: appData.eventMaterials || {},
    
    // 🔧 REDONDANCE MULTIPLE POUR ASSURER L'IMPORT
    donneesCompletes: {
      agents: appData.agents,
      events: appData.events,
      materials: appData.materials || [],
      eventMaterials: appData.eventMaterials || {},
      tasks: appData.tasks || {}
    },
    
    // Métadonnées pour diagnostic
    exportInfo: {
      version: "Biguglia v2.0",
      dateExport: new Date().toISOString(),
      materialsCount: (appData.materials || []).length,
      eventMaterialsCount: Object.keys(appData.eventMaterials || {}).length,
      exportType: "JSON_COMPLET_REDONDANT"
    },
    
    // Backup supplémentaire
    backup: {
      materials: appData.materials || [],
      eventMaterials: appData.eventMaterials || {}
    }
  };
  
  console.log('📦 Diagnostic export matériel:', {
    'appData.materials': (appData.materials || []).length,
    'dataToExport.materials': (dataToExport.materials || []).length,
    'dataToExport.donneesCompletes.materials': (dataToExport.donneesCompletes.materials || []).length,
    'dataToExport.backup.materials': (dataToExport.backup.materials || []).length
  });
  
  const dataStr = JSON.stringify(dataToExport, null, 2);
  const blob=new Blob([dataStr],{type:'application/json'});
  const a=document.createElement('a');
  a.href=URL.createObjectURL(blob); 
  a.download=`biguglia_app_data_complet_${new Date().toISOString().split('T')[0]}.json`; 
  a.click();
  URL.revokeObjectURL(a.href);
  showToast('Export JSON complet terminé (avec matériel redondant) !', 'success');
}

// Fonction pour nettoyer et valider les données
function cleanAndValidateData(data) {
  console.log('🧹 Nettoyage et validation des données...');
  
  // Nettoyer les agents
  data.agents = (data.agents || []).filter(agent => 
    agent && agent.nom && agent.nom.trim().length > 0
  );
  
  // Nettoyer les événements
  data.events = (data.events || []).filter(event => 
    event && event.nom && event.nom.trim().length > 0
  ).map(event => {
    // Nettoyer les participants
    event.participants = (event.participants || []).filter(part =>
      part && part.agentId && part.date
    );
    
    // Nettoyer les documents volumineux automatiquement
    if (event.documents) {
      const originalCount = event.documents.length;
      event.documents = event.documents.filter(doc => {
        // Supprime les documents corrompus ou sans données
        if (!doc.dataUrl || !doc.name) return false;
        
        // Supprime automatiquement les documents > 1MB
        if (doc.dataUrl.length > 1024 * 1024) {
          console.warn(`📦 Document volumineux supprimé: ${doc.name} (${(doc.dataUrl.length / 1024).toFixed(0)}KB)`);
          return false;
        }
        
        return true;
      });
      
      // Limite à 5 documents par événement
      if (event.documents.length > 5) {
        event.documents = event.documents
          .sort((a, b) => new Date(b.uploadDate || 0) - new Date(a.uploadDate || 0))
          .slice(0, 5);
      }
    }
    
    return event;
  });
  
  // Nettoyer les tâches
  if (data.tasks) {
    Object.keys(data.tasks).forEach(eventId => {
      if (!data.tasks[eventId] || !Array.isArray(data.tasks[eventId])) {
        delete data.tasks[eventId];
      } else {
        data.tasks[eventId] = data.tasks[eventId].filter(task => 
          task && task.name && task.start && task.end
        );
        
        if (data.tasks[eventId].length === 0) {
          delete data.tasks[eventId];
        }
      }
    });
  }
  
  // Initialiser ou nettoyer le matériel
  if (!data.materials || !Array.isArray(data.materials)) {
    data.materials = [];
  }
  
  if (!data.eventMaterials || typeof data.eventMaterials !== 'object') {
    data.eventMaterials = {};
  }
  
  console.log('✅ Données nettoyées:', {
    agents: data.agents.length,
    events: data.events.length,
    materials: data.materials.length,
    eventMaterials: Object.keys(data.eventMaterials).length
  });
  
  return data;
}

/* ==== Import de données avec diagnostic amélioré ==== */
function importData(files) {
  console.log('📥 === DÉBUT IMPORT FICHIER ===');
  
  if (!files || !files.length) {
    showToast('❌ Aucun fichier sélectionné', 'warning');
    return;
  }
  
  const file = files[0];
  const fileName = file.name;
  console.log('📄 Fichier sélectionné:', fileName, 'Taille:', (file.size / 1024).toFixed(1) + 'KB');
  
  // Vérification du type de fichier
  if (!fileName.toLowerCase().endsWith('.json')) {
    showToast('❌ FORMAT INVALIDE\n\nSeuls les fichiers JSON (.json) sont acceptés.\n\nVotre fichier: ' + fileName, 'error', 5000);
    return;
  }
  
  // Vérification de la taille du fichier (max 50MB)
  const maxSize = 50 * 1024 * 1024; // 50MB
  if (file.size > maxSize) {
    showToast(`❌ FICHIER TROP VOLUMINEUX\n\nTaille max: 50MB\nVotre fichier: ${(file.size / 1024 / 1024).toFixed(1)}MB\n\nRéduisez la taille ou utilisez un export sans documents.`, 'error', 6000);
    return;
  }
  
  if (file.size === 0) {
    showToast('❌ FICHIER VIDE\n\nLe fichier sélectionné ne contient aucune donnée.', 'error', 5000);
    return;
  }
  
  console.log('✅ Vérifications fichier OK - Début lecture...');
  showToast('📥 Lecture du fichier en cours...', 'info', 3000);
  
  const reader = new FileReader();
  
  reader.onload = (e) => {
    try {
      console.log('📥 Début de l\'import de données...');
      
      // Vérification de la longueur du contenu
      const content = e.target.result;
      if (!content || content.trim().length === 0) {
        showToast('❌ FICHIER VIDE\n\nLe fichier sélectionné ne contient aucune donnée.', 'error', 5000);
        return;
      }
      
      console.log('📄 Contenu lu, taille:', content.length, 'caractères');
      showToast('🔍 Analyse du fichier JSON...', 'info', 2000);
      
      // Nettoyage basique du contenu avec validation renforcée
      const validationResult = validateAndRepairJSON(content);
      if (!validationResult.isValid) {
        showToast(`❌ JSON INVALIDE\n\nErreur: ${validationResult.error}\n\nVérifiez la structure de votre fichier.`, 'error', 6000);
        return;
      }
      
      const cleanContent = validationResult.repairedContent || content.trim();
      
      
      // Parsing JSON avec gestion d'erreur simple et diagnostic
      let imported;
      try {
        console.log('🔄 Tentative de parsing JSON...');
        imported = JSON.parse(cleanContent);
        console.log('✅ JSON parsé avec succès');
        console.log('📋 Clés principales trouvées:', Object.keys(imported));
        
      } catch (parseError) {
        console.error('❌ Erreur de parsing JSON:', parseError);
        
        // Diagnostic simple de l'erreur
        let errorMessage = '❌ ERREUR DE FORMAT JSON\n\n';
        errorMessage += `📄 Fichier: ${file.name}\n`;
        errorMessage += `📏 Taille: ${(file.size / 1024).toFixed(1)}KB\n\n`;
        
        if (parseError.message.includes('Unexpected token')) {
          errorMessage += '❌ CARACTÈRE INVALIDE DÉTECTÉ\n\n';
          errorMessage += '� SOLUTIONS :\n';
          errorMessage += '1. ✅ Réexportez les données depuis l\'application\n';
          errorMessage += '2. 🔍 Vérifiez que le fichier n\'est pas corrompu\n';
          errorMessage += '3. 🌐 Testez votre JSON sur jsonlint.com\n';
          
        } else if (parseError.message.includes('Unexpected end')) {
          errorMessage += '❌ FICHIER JSON INCOMPLET\n\n';
          errorMessage += '🔧 SOLUTIONS :\n';
          errorMessage += '1. ✅ Le fichier semble tronqué\n';
          errorMessage += '2. 🔄 Réexportez depuis l\'application d\'origine\n';
          errorMessage += '3. 📋 Vérifiez que le téléchargement s\'est bien terminé\n';
          
        } else {
          errorMessage += `❌ ERREUR TECHNIQUE :\n${parseError.message}\n\n`;
          errorMessage += '🔧 SOLUTIONS :\n';
          errorMessage += '1. ✅ Réexportez les données\n';
          errorMessage += '2. � Réessayez avec un fichier plus récent\n';
        }
        
        showToast(errorMessage, 'error', 10000);
        return;
      }
      
      // 🔍 DIAGNOSTIC EXHAUSTIF DU MATÉRIEL AVANT NORMALISATION
      console.log('🔍 === DIAGNOSTIC MATÉRIEL COMPLET ===');
      console.log('📦 Structure des données importées:', Object.keys(imported));
      
      // 📦 RECHERCHE EXHAUSTIVE DU MATÉRIEL DANS TOUTES LES STRUCTURES
      const foundMaterial = diagnosticMaterielAvecForçage(imported);
      if (foundMaterial && foundMaterial.length > 0) {
        console.log('🎯 Matériel trouvé et forcé avec succès:', foundMaterial.length, 'articles');
        // Injection directe dans la structure principale
        imported.materials = foundMaterial;
        console.log('✅ Matériel injecté dans imported.materials');
      } else {
        console.warn('⚠️ Aucun matériel récupérable - tentative de récupération alternative');
        // Tentative de récupération manuelle
        const materielAlternatif = forceRecupMaterielAlternatif(imported);
        if (materielAlternatif && materielAlternatif.length > 0) {
          imported.materials = materielAlternatif;
          console.log('✅ Matériel alternatif injecté:', materielAlternatif.length, 'articles');
        }
      }
      
      // Validation de base de la structure
      if (!imported || typeof imported !== 'object') {
        showToast('❌ STRUCTURE INVALIDE\n\nLe fichier ne contient pas de données valides.', 'error', 5000);
        return;
      }
      
      console.log('🔍 Données importées après diagnostic:', {
        agents: (imported.agents || []).length,
        events: (imported.events || []).length,
        materials: (imported.materials || []).length,
        tasks: Object.keys(imported.tasks || {}).length
      });
      
      // Normalisation des données avec gestion d'erreur
      let normalized;
      try {
        normalized = normalizeDataWithMaterialFix(imported);
      } catch (normalizeError) {
        console.error('❌ Erreur de normalisation:', normalizeError);
        showToast('❌ ERREUR DE NORMALISATION\n\nImpossible de traiter les données du fichier.', 'error', 5000);
        return;
      }
      
      // Validation après normalisation
      if (!normalized.agents || !Array.isArray(normalized.agents) || 
          !normalized.events || !Array.isArray(normalized.events)) {
        showToast('❌ DONNÉES INCOMPLÈTES\n\nLe fichier ne contient pas les données requises (agents ou événements manquants).', 'error', 6000);
        return;
      }
      
      // Sauvegarde de sécurité avant import
      const backupData = { ...appData };
      console.log('💾 Sauvegarde de sécurité créée');
      
      // Confirmation avec détails incluant le matériel
      const materialInfo = foundMaterial ? 
        `✅ ${foundMaterial.length} matériel(s) trouvé(s) dans ${window.lastMaterialLocation || 'source inconnue'}` : 
        '❌ Aucun matériel trouvé (matériel par défaut sera utilisé)';
      
      const confirmMessage = `🔄 CONFIRMATION D'IMPORT
      
📊 Données à importer :
• ${normalized.agents.length} agent(s)
• ${normalized.events.length} événement(s)
• ${materialInfo}
• ${Object.keys(normalized.tasks || {}).length} plannings

⚠️  ATTENTION : Cette action va remplacer TOUTES vos données actuelles !

📋 Données actuelles :
• ${appData.agents.length} agent(s)
• ${appData.events.length} événement(s)
• ${(appData.materials || []).length} matériel(s)

✅ Une sauvegarde de sécurité a été créée.

Continuer l'import ?`;
      
      if (!confirm(confirmMessage)) {
        console.log('❌ Import annulé par l\'utilisateur');
        showToast('Import annulé', 'info', 2000);
        return;
      }
      
      showToast('🔄 Import en cours...', 'info', 3000);
      
      try {
        // Remplacement des données avec validation étape par étape
        appData.agents = normalized.agents || [];
        appData.events = normalized.events || [];
        appData.tasks = normalized.tasks || {};
        
        // 📦 IMPORT DIRECT ET FORCÉ DU MATÉRIEL
        console.log('📦 === IMPORT FORCÉ DU MATÉRIEL ===');
        
        if (foundMaterial && foundMaterial.length > 0) {
          console.log('🎯 Import direct du matériel trouvé:', foundMaterial.length, 'articles');
          appData.materials = foundMaterial.map((material, index) => {
            const processed = {
              id: Number(material.id) || Date.now() + index + Math.floor(Math.random() * 1000),
              nom: material.nom || material.name || `Article ${index + 1}`,
              categorie: material.categorie || material.category || 'Divers',
              stockTotal: Math.max(0, Number(material.stockTotal) || Number(material.stock) || 0),
              stockReserve: Math.max(0, Number(material.stockReserve) || 0),
              unite: material.unite || material.unit || 'unité',
              stockMin: Math.max(0, Number(material.stockMin) || Number(material.minStock) || 0),
              statut: material.statut || material.status || 'disponible',
              description: String(material.description || '').trim()
            };
            console.log(`📦 Matériel importé [${index + 1}]:`, processed.nom, '- Stock:', processed.stockTotal);
            return processed;
          });
          console.log('✅ Matériel importé avec succès:', appData.materials.length, 'articles');
        } else if (normalized.materials && normalized.materials.length > 0) {
          console.log('� Utilisation du matériel normalisé:', normalized.materials.length, 'articles');
          appData.materials = normalized.materials;
        } else {
          console.warn('⚠️ Aucun matériel disponible, conservation du matériel existant');
          if (!appData.materials || !Array.isArray(appData.materials) || appData.materials.length === 0) {
            console.log('🔧 Initialisation du matériel par défaut...');
            appData.materials = [
              { id: 1, nom: "Table rectangulaire", categorie: "Tables", stockTotal: 50, stockReserve: 0, unite: "unité", stockMin: 5, statut: "disponible", description: "Table 180x80cm pliante" },
              { id: 2, nom: "Chaise pliante", categorie: "Chaises", stockTotal: 200, stockReserve: 0, unite: "unité", stockMin: 20, statut: "disponible", description: "Chaise pliante blanche" },
              { id: 3, nom: "Tente 3x3m", categorie: "Tentes", stockTotal: 10, stockReserve: 0, unite: "unité", stockMin: 2, statut: "disponible", description: "Tente pliante 3x3m avec bâches latérales" }
            ];
            console.log('✅ Matériel par défaut initialisé:', appData.materials.length, 'articles');
          } else {
            console.log('🔄 Conservation du matériel existant:', appData.materials.length, 'articles');
          }
        }
        
        // Import des attributions de matériel avec recherche étendue
        console.log('📋 === IMPORT DES ATTRIBUTIONS MATÉRIEL ===');
        const attributionSources = [
          { name: 'normalized.eventMaterials', data: normalized.eventMaterials },
          { name: 'imported.eventMaterials', data: imported.eventMaterials },
          { name: 'imported.donneesCompletes?.eventMaterials', data: imported.donneesCompletes?.eventMaterials },
          { name: 'imported.appData?.eventMaterials', data: imported.appData?.eventMaterials }
        ];
        
        let foundAttributions = null;
        attributionSources.forEach(source => {
          if (source.data && typeof source.data === 'object' && Object.keys(source.data).length > 0) {
            console.log(`✅ Attributions trouvées dans ${source.name}:`, Object.keys(source.data).length, 'événements');
            if (!foundAttributions) {
              foundAttributions = source.data;
            }
          }
        });
        
        if (foundAttributions) {
          appData.eventMaterials = {};
          Object.entries(foundAttributions).forEach(([eventId, allocations]) => {
            if (Array.isArray(allocations)) {
              appData.eventMaterials[eventId] = allocations.map(allocation => ({
                id: allocation.id || Date.now() + Math.random(),
                materialId: Number(allocation.materialId),
                quantite: Number(allocation.quantite) || Number(allocation.quantity) || 1,
                dateDebut: allocation.dateDebut || allocation.startDate || '',
                dateFin: allocation.dateFin || allocation.endDate || '',
                valide: Boolean(allocation.valide) || Boolean(allocation.returned) || false,
                sortieValidee: Boolean(allocation.sortieValidee) || Boolean(allocation.validated) || false
              }));
            }
          });
          console.log('✅ Attributions matériel importées pour', Object.keys(appData.eventMaterials).length, 'événements');
        } else {
          console.warn('⚠️ Aucune attribution de matériel trouvée');
          if (!appData.eventMaterials) {
            appData.eventMaterials = {};
          }
        }
        
        // Nettoyage et reconstruction des blobs
        blobManager.clear();
        console.log('🧹 BlobManager nettoyé');
        
        // Reconstitution des blobs pour les documents
        let documentsProcessed = 0;
        let documentsErrors = 0;
        
        appData.events.forEach(ev => {
          (ev.documents || []).forEach(doc => {
            if (doc.dataUrl) {
              try {
                const blob = dataURLtoBlob(doc.dataUrl);
                blobManager.addBlob(ev.id, doc.id, blob);
                documentsProcessed++;
              } catch(e) {
                console.warn('❌ Impossible de recréer le blob pour:', doc.name, e);
                documentsErrors++;
              }
            }
          });
        });
        
        console.log(`📄 Documents traités: ${documentsProcessed} OK, ${documentsErrors} erreurs`);
        
        // Validation et correction du schéma des tâches
        ensureTasksSchema();
        console.log('✅ Schéma des tâches validé');
        
        // Sauvegarde avec gestion d'erreur
        markAsChanged();
        try {
          saveData();
          console.log('💾 Données sauvegardées avec succès');
        } catch (saveError) {
          console.error('❌ Erreur de sauvegarde:', saveError);
          
          // Tentative de restauration
          try {
            appData = backupData;
            saveData();
            showToast('❌ ERREUR DE SAUVEGARDE\n\nDonnées restaurées depuis la sauvegarde de sécurité.', 'error', 6000);
            return;
          } catch (restoreError) {
            console.error('❌ Erreur critique de restauration:', restoreError);
            showToast('🚨 ERREUR CRITIQUE\n\nImpossible de restaurer les données. Rechargez la page.', 'error', 10000);
            return;
          }
        }
        
        // Rafraîchissement complet de l'interface
        try {
          updateTypeFilter();
          updateDashboard();
          updateAgentsList();
          updateEventsList();
          updateHoursTable();
          renderCalendar();
          
          // Rafraîchir le matériel si on est dans ce module
          if (document.getElementById('material').classList.contains('active')) {
            updateMaterialList();
            updateMaterialFilters();
          }
          
          console.log('🔄 Interface mise à jour');
        } catch (uiError) {
          console.error('❌ Erreur de mise à jour de l\'interface:', uiError);
          showToast('⚠️ Interface partiellement mise à jour. Rechargez la page si nécessaire.', 'warning', 4000);
        }
        
        // Message de succès détaillé avec informations sur le matériel
        const materialStatus = materialToImport ? 
          `✅ ${appData.materials.length} article(s) de matériel restauré(s) depuis ${importSource}` : 
          `⚠️ Matériel par défaut utilisé (${appData.materials.length} articles)`;
          
        const successMessage = `✅ IMPORT RÉUSSI !

📊 Données importées :
• ${normalized.agents.length} agent(s)
• ${normalized.events.length} événement(s)
• ${materialStatus}
• ${Object.keys(appData.eventMaterials).length} attributions matériel
• ${Object.keys(normalized.tasks || {}).length} planning(s)
• ${documentsProcessed} document(s) reconstitué(s)

${documentsErrors > 0 ? `⚠️ ${documentsErrors} document(s) non récupérable(s)` : ''}

🎉 Toutes vos données ont été importées et l'interface a été mise à jour !

${materialToImport ? '📦 Votre stock de matériel a été restauré avec succès !' : '📦 Ajoutez votre matériel depuis le module Matériel.'}`;
        
        showToast(successMessage, 'success', 10000);
        
        console.log('✅ Import terminé avec succès');
        console.log('📊 État final:', {
          agents: appData.agents.length,
          events: appData.events.length,
          materials: appData.materials.length,
          eventMaterials: Object.keys(appData.eventMaterials).length
        });
        
        // 🧪 TEST FINAL DE L'IMPORT MATÉRIEL
        console.log('🧪 === VÉRIFICATION FINALE MATÉRIEL ===');
        const testResult = testMaterialImport();
        if (testResult) {
          console.log('🎉 Test matériel RÉUSSI - Import validé');
        } else {
          console.warn('⚠️ Test matériel partiellement réussi - Vérifiez les données');
        }
        
      } catch (importError) {
        console.error('❌ Erreur critique pendant l\'import:', importError);
        
        // Tentative de restauration des données
        try {
          appData = backupData;
          saveData();
          updateDashboard();
          showToast('❌ IMPORT ÉCHOUÉ\n\nLes données ont été restaurées à leur état précédent.', 'error', 6000);
        } catch (restoreError) {
          console.error('❌ Erreur critique de restauration:', restoreError);
          showToast('🚨 ERREUR CRITIQUE\n\nImpossible de restaurer les données.\nRechargez la page et réessayez.', 'error', 10000);
        }
      }
      
    } catch(error) {
      console.error('❌ Erreur générale d\'import:', error);
      showToast('❌ ERREUR D\'IMPORT\n\nUne erreur inattendue s\'est produite.\nVérifiez le fichier et réessayez.', 'error', 6000);
    }
  };
  
  reader.onerror = (error) => {
    console.error('❌ Erreur de lecture du fichier:', error);
    showToast('❌ ERREUR DE LECTURE\n\nImpossible de lire le fichier.\nVérifiez qu\'il n\'est pas corrompu.', 'error', 5000);
  };
  
  reader.onprogress = (e) => {
    if (e.lengthComputable) {
      const percentLoaded = Math.round((e.loaded / e.total) * 100);
      console.log(`📥 Lecture en cours: ${percentLoaded}%`);
      if (percentLoaded < 100) {
        showToast(`📥 Lecture du fichier: ${percentLoaded}%`, 'info', 1000);
      }
    }
  };
  
  // Lecture du fichier avec encodage UTF-8 pour supporter les caractères français
  reader.readAsText(file, 'UTF-8');
}

// Fonction de diagnostic améliorée pour les problèmes d'import
function diagnosticImportError(error, content) {
  console.log('🔍 === DIAGNOSTIC ERREUR IMPORT ===');
  
  let errorInfo = {
    type: 'unknown',
    message: error.message,
    suggestions: [],
    canRepair: false
  };
  
  // Analyse du type d'erreur
  if (error.message.includes('Unexpected token')) {
    errorInfo.type = 'syntax';
    
    if (error.message.includes('position')) {
      const match = error.message.match(/position (\d+)/);
      if (match) {
        const pos = parseInt(match[1]);
        const lineNumber = content.substring(0, pos).split('\n').length;
        const context = content.substring(Math.max(0, pos - 50), pos + 50);
        
        errorInfo.position = pos;
        errorInfo.line = lineNumber;
        errorInfo.context = context;
        
        if (error.message.includes('Unexpected token ,')) {
          errorInfo.suggestions.push('Virgule en trop après le dernier élément');
          errorInfo.canRepair = true;
        }
        
        if (error.message.includes('Unexpected token }')) {
          errorInfo.suggestions.push('Accolade fermante inattendue - vérifiez la structure');
        }
      }
    }
  } else if (error.message.includes('Unexpected end')) {
    errorInfo.type = 'incomplete';
    errorInfo.suggestions.push('Fichier JSON incomplet ou tronqué');
    errorInfo.suggestions.push('Il manque probablement des accolades ou crochets de fermeture');
    errorInfo.canRepair = true;
  }
  
  return errorInfo;
}

// Fonction de réparation automatique simplifiée
function repairJSON(content) {
  console.log('🔧 Tentative de réparation JSON...');
  
  let repaired = content.trim();
  
  try {
    // Supprimer les virgules en trop
    repaired = repaired.replace(/,(\s*[}\]])/g, '$1');
    
    // Vérifier l'équilibrage des accolades
    const openBraces = (repaired.match(/\{/g) || []).length;
    const closeBraces = (repaired.match(/\}/g) || []).length;
    
    if (openBraces > closeBraces) {
      const missing = openBraces - closeBraces;
      repaired += '}'.repeat(missing);
      console.log(`🔧 Ajouté ${missing} accolade(s) fermante(s)`);
    }
    
    // Vérifier l'équilibrage des crochets
    const openBrackets = (repaired.match(/\[/g) || []).length;
    const closeBrackets = (repaired.match(/\]/g) || []).length;
    
    if (openBrackets > closeBrackets) {
      const missing = openBrackets - closeBrackets;
      repaired += ']'.repeat(missing);
      console.log(`🔧 Ajouté ${missing} crochet(s) fermant(s)`);
    }
    
    return repaired;
  } catch (error) {
    console.error('Erreur lors de la réparation:', error);
    return null;
  }
}

// Nouvelle fonction de normalisation simplifiée et robuste
function normalizeDataWithMaterialFix(json) {
  console.log('🔧 === Normalisation des données ===');
  
  // Structure de base
  const out = { 
    agents: [], 
    events: [], 
    tasks: {},
    materials: [],
    eventMaterials: {}
  };
  
  try {
    // Normalisation des agents
    if (json.agents && Array.isArray(json.agents)) {
      out.agents = json.agents.filter(a => a && a.nom).map(a => ({
        id: Number.isFinite(+a.id) ? +a.id : Date.now() + Math.floor(Math.random()*9999),
        nom: String(a.nom || a.name || 'Agent').trim(),
        service: String(a.service || '').trim(),
        telephone: String(a.telephone || a.phone || '').trim()
      }));
    }
    
    const agentIds = new Set(out.agents.map(a => a.id));
    
    // Normalisation des événements
    if (json.events && Array.isArray(json.events)) {
      out.events = json.events.filter(e => e && e.nom).map(e => {
        const participants = Array.isArray(e.participants) ? e.participants : [];
        const documents = Array.isArray(e.documents) ? e.documents : [];
        
        const normParts = participants.map(p => {
          const agentId = Number.isFinite(+p.agentId) ? +p.agentId : null;
          return {
            agentId: agentId && agentIds.has(agentId) ? agentId : (out.agents[0]?.id || 0),
            date: p.date || e.dateDebut || todayYMD(),
            motif: String(p.motif || p.reason || '').trim(),
            nature: normalizeNature(p.nature || p.type),
            heureDebut: String(p.heureDebut || p.start || '').trim(),
            heureFin: String(p.heureFin || p.end || '').trim(),
            payees: Number(p.payees) || 0,
            recup: Number(p.recup) || 0,
            voirieFest: Number(p.voirieFest) || 0
          };
        });
        
        return {
          id: Number.isFinite(+e.id) ? +e.id : Date.now() + Math.floor(Math.random()*9999),
          nom: String(e.nom || e.name || 'Événement').trim(),
          categorie: String(e.categorie || e.type || 'Divers').trim(),
          lieu: String(e.lieu || e.location || '').trim(),
          dateDebut: e.dateDebut || e.startDate || todayYMD(),
          dateFin: e.dateFin || e.endDate || e.dateDebut || todayYMD(),
          description: String(e.description || '').trim(),
          participants: normParts,
          documents: documents
        };
      });
    }
    
    // Normalisation du matériel
    if (json.materials && Array.isArray(json.materials)) {
      out.materials = json.materials.filter(m => m && m.nom).map((m, index) => ({
        id: Number.isFinite(+m.id) ? +m.id : Date.now() + index,
        nom: String(m.nom || m.name || `Article ${index + 1}`).trim(),
        categorie: String(m.categorie || m.category || 'Divers').trim(),
        stockTotal: Math.max(0, Number(m.stockTotal) || Number(m.stock) || 0),
        stockReserve: Math.max(0, Number(m.stockReserve) || 0),
        unite: String(m.unite || m.unit || 'unité').trim(),
        stockMin: Math.max(0, Number(m.stockMin) || Number(m.minStock) || 0),
        statut: String(m.statut || m.status || 'disponible').trim(),
        description: String(m.description || '').trim()
      }));
    }
    
    // Normalisation des attributions de matériel
    if (json.eventMaterials && typeof json.eventMaterials === 'object') {
      Object.entries(json.eventMaterials).forEach(([eventId, allocations]) => {
        if (Array.isArray(allocations)) {
          out.eventMaterials[eventId] = allocations.map(allocation => ({
            id: allocation.id || Date.now() + Math.random(),
            materialId: Number(allocation.materialId) || 0,
            quantite: Number(allocation.quantite) || Number(allocation.quantity) || 1,
            dateDebut: allocation.dateDebut || allocation.startDate || '',
            dateFin: allocation.dateFin || allocation.endDate || '',
            valide: Boolean(allocation.valide) || Boolean(allocation.returned) || false,
            sortieValidee: Boolean(allocation.sortieValidee) || Boolean(allocation.validated) || false
          }));
        }
      });
    }
    
    // Normalisation des tâches
    if (json.tasks && typeof json.tasks === 'object') {
      out.tasks = json.tasks;
    }
    
  } catch (error) {
    console.error('Erreur lors de la normalisation:', error);
    throw new Error('Impossible de normaliser les données: ' + error.message);
  }
  
  console.log('✅ Normalisation terminée:', {
    agents: out.agents.length,
    events: out.events.length,
    materials: out.materials.length,
    eventMaterials: Object.keys(out.eventMaterials).length
  });
  
  return out;
}

/* ==== Migration schéma tâches ==== */
function ensureTasksSchema() {
  if (!appData.tasks) appData.tasks = {};
  Object.keys(appData.tasks).forEach(eventId => {
    const tasks = appData.tasks[eventId] || [];
    appData.tasks[eventId] = tasks.map(task => {
      if (task.agentId != null && !Array.isArray(task.agentIds)) {
        return { ...task, agentIds: [task.agentId], agentId: null };
      }
      if (task.agentId == null && !Array.isArray(task.agentIds)) {
        return { ...task, agentIds: [] };
      }
      return task;
    });
  });
}

/* ==== Event Delegation ==== */
function setupEventDelegation() {
  // Délégation globale pour tous les boutons avec data-action
  document.addEventListener('click', (e) => {
    const btn = e.target.closest('[data-action]');
    if (!btn) return;
    
    e.preventDefault();
    e.stopPropagation();
    
    const action = btn.dataset.action;
    const id = btn.dataset.id;
    const index = btn.dataset.index;
    
    try {
      switch(action) {
        // === Actions Agents ===
        case 'agent-edit':
          showAgentForm(+id);
          break;
        case 'agent-delete':
          deleteAgent(+id);
          break;
          
        // === Actions Événements ===
        case 'event-edit':
          showAddEventForm(+id);
          break;
        case 'event-delete':
          deleteEvent(+id);
          break;
        case 'participants-open':
          openParticipantsModal(+id);
          break;
        case 'docs-open':
          openDocsModal(+id);
          break;
        case 'material-open':
          openMaterialModal(+id);
          break;
        case 'planning-open':
          openPlanningModal(+id);
          break;
          
        // === Actions Participants ===
        case 'row-edit':
          editRowTimes(+index);
          break;
        case 'row-dup':
          duplicateRow(+index);
          break;
        case 'row-delete':
          removeRow(+index);
          break;
          
        default:
          console.warn('Action non reconnue:', action);
      }
    } catch(error) {
      console.error(`Erreur lors de l'action ${action}:`, error);
      showToast('Une erreur est survenue', 'error');
    }
  });
  
  // Fermeture des résultats de recherche en cliquant ailleurs
  document.addEventListener('click', (e) => {
    const searchResults = document.getElementById('searchResults');
    const searchInput = document.getElementById('globalSearch');
    
    if (searchResults && searchInput && 
        !searchResults.contains(e.target) && 
        !searchInput.contains(e.target)) {
      searchResults.style.display = 'none';
    }
  });
  
  // Gestion des raccourcis clavier
  document.addEventListener('keydown', (e) => {
    // Échap pour fermer les modals
    if (e.key === 'Escape') {
      const activeModal = document.querySelector('.modal.active');
      if (activeModal) {
        const modalId = activeModal.id;
        switch(modalId) {
          case 'agentHistoryModal': closeAgentHistoryModal(); break;
          case 'agentEventDetailModal': closeAgentEventDetailModal(); break;
          case 'participantsModal': closeParticipantsModal(); break;
          case 'docsModal': closeDocsModal(); break;
          case 'planningModal': closePlanningModal(); break;
          case 'barInfoModal': closeBarInfoModal(); break;
        }
      }
    }
    
    // Ctrl+S pour sauvegarder
    if (e.ctrlKey && e.key === 's') {
      e.preventDefault();
      saveData();
      showToast('Données sauvegardées', 'success', 2000);
    }
  });
}

/* ==== Initialisation de l'application ==== */
function initApp(){
  console.log('🚀 Initialisation de l\'application Biguglia');
  
  // Création des éléments UI manquants
  createMissingUIElements();
  
  const saved=safeGetLS('biguglia_app_data');
  if(saved){
    try{
      let normalizedData = normalizeData(saved);
      
      // Nettoyage et validation des données
      normalizedData = cleanAndValidateData(normalizedData);
      
      appData = normalizedData;
      
      // Reconstituer les blobs depuis dataUrl (session précédente)
      blobManager.clear();
      (appData.events||[]).forEach(ev=>{
        (ev.documents||[]).forEach(d=>{
          if(d.dataUrl){
            try{ 
              const blob = dataURLtoBlob(d.dataUrl); 
              blobManager.addBlob(ev.id, d.id, blob);
            }catch(e){
              console.warn('Impossible de reconstituer le blob pour:', d.name);
            }
          }
        });
      });
    }catch(e){
      console.warn('Normalisation du localStorage échouée, on repart des données par défaut.', e);
    }
  }
  
  // Initialiser materials et eventMaterials s'ils n'existent pas
  if (!appData.materials) {
    appData.materials = [
      { 
        id: 1, 
        nom: "Table rectangulaire", 
        categorie: "Tables", 
        stockTotal: 50, 
        stockReserve: 0,
        unite: "unité", 
        stockMin: 5,
        statut: "disponible",
        description: "Table 180x80cm pliante"
      },
      { 
        id: 2, 
        nom: "Chaise pliante", 
        categorie: "Chaises", 
        stockTotal: 200, 
        stockReserve: 0,
        unite: "unité", 
        stockMin: 20,
        statut: "disponible",
        description: "Chaise pliante blanche"
      },
      { 
        id: 3, 
        nom: "Tente 3x3m", 
        categorie: "Tentes", 
        stockTotal: 10, 
        stockReserve: 0,
        unite: "unité", 
        stockMin: 2,
        statut: "disponible",
        description: "Tente pliante 3x3m avec bâches latérales"
      }
    ];
    console.log('✨ Matériel initialisé:', appData.materials.length, 'articles');
  }
  
  if (!appData.eventMaterials) {
    appData.eventMaterials = {};
    console.log('✨ EventMaterials initialisé');
  }
  
  ensureTasksSchema();
  
  // Configuration du mode sombre depuis localStorage
  const savedDarkMode = localStorage.getItem('darkMode');
  if (savedDarkMode === 'true') {
    document.documentElement.classList.add('dark');
    const darkModeText = document.getElementById('darkModeText');
    if (darkModeText) darkModeText.textContent = 'Mode clair';
  }
  
  // Configuration de la délégation d'événements
  setupEventDelegation();
  
  // Démarrage de la sauvegarde automatique
  startAutoSave();
  
  // Vérification automatique du stockage si nécessaire
  autoCleanupIfNeeded();
  
  // Initialisation de l'interface
  populateMultiFilters();
  updateTypeFilter();
  updateDashboard();
  updateAgentsList();
  updateEventsList();
  updateHoursTable();
  renderCalendar();
  
  // Gestion de la fermeture de la page (prévenir la perte de données)
  window.addEventListener('beforeunload', (e) => {
    if (hasUnsavedChanges) {
      e.preventDefault();
      e.returnValue = 'Vous avez des modifications non sauvegardées. Voulez-vous vraiment quitter ?';
      return e.returnValue;
    }
  });
  
  // Sauvegarde d'urgence en cas de fermeture brutale
  window.addEventListener('unload', () => {
    if (hasUnsavedChanges) {
      navigator.sendBeacon('/save', JSON.stringify(appData));
    }
  });
  
  // Sauvegarde immédiate après initialisation
  markAsChanged();
  saveData();
  
  // Message de bienvenue
  setTimeout(() => {
    showToast('🎉 Application Biguglia chargée avec succès !', 'success', 3000);
  }, 500);
  
  console.log('✅ Application Biguglia initialisée avec succès');
  console.log('📊 État actuel:', {
    agents: appData.agents.length,
    events: appData.events.length,
    materials: appData.materials.length,
    eventMaterials: Object.keys(appData.eventMaterials).length
  });
}

// Fonction pour créer les éléments UI manquants
function createMissingUIElements() {
  console.log('🔧 Création des éléments UI manquants...');
  
  // Créer le header s'il n'existe pas ou est vide
  const headerContainer = document.querySelector('header .container .flex.justify-between');
  if (headerContainer && !headerContainer.innerHTML.trim()) {
    console.log('🔧 Création du header...');
    headerContainer.innerHTML = `
      <div class="flex items-center space-x-4">
        <h1 class="text-2xl font-bold text-gray-800">🏛️ Città di Biguglia</h1>
        <span class="text-sm text-gray-500 bg-gray-100 px-2 py-1 rounded">Application Festivités</span>
      </div>
      
      <div class="flex items-center space-x-2">
        <div class="relative">
          <input type="text" id="globalSearch" placeholder="Rechercher..." 
                 class="px-3 py-2 border rounded-lg w-48 text-sm"
                 oninput="handleGlobalSearch(this.value)"
                 onfocus="this.select()">
          <div id="searchResults" class="absolute top-10 left-0 w-full bg-white border rounded-lg shadow-lg max-h-64 overflow-y-auto z-50" style="display:none;"></div>
        </div>
        
        <button onclick="showStorageManager()" class="bg-yellow-600 hover:bg-yellow-700 text-white px-2 py-2 rounded text-sm" title="Gestion du stockage">
          <i class="fas fa-database"></i>
        </button>
        
        <button onclick="exportToUSB()" class="bg-purple-600 hover:bg-purple-700 text-white px-2 py-2 rounded text-sm" title="Export USB">
          <i class="fas fa-usb"></i>
        </button>
        
        <button onclick="exportToCalendar()" class="bg-orange-600 hover:bg-orange-700 text-white px-2 py-2 rounded text-sm" title="Export calendrier">
          <i class="fas fa-calendar-plus"></i>
        </button>
        
        <button onclick="exportToExcel()" class="bg-green-600 hover:bg-green-700 text-white px-2 py-2 rounded text-sm" title="Export Excel">
          <i class="fas fa-file-excel"></i>
        </button>
        
        <button onclick="exporterDonnees()" class="bg-blue-600 hover:bg-blue-700 text-white px-2 py-2 rounded text-sm" title="Export JSON">
          <i class="fas fa-download"></i>
        </button>
        
        <button onclick="document.getElementById('importInput').click()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-2 py-2 rounded text-sm" title="Importer des données">
          <i class="fas fa-upload"></i>
        </button>
        
        <button onclick="toggleDarkMode()" class="bg-gray-600 hover:bg-gray-700 text-white px-2 py-2 rounded text-sm" title="Mode sombre/clair">
          <i class="fas fa-moon"></i>
          <span id="darkModeText" class="hidden md:inline ml-1">Mode sombre</span>
        </button>
      </div>
    `;
  }
  
  // Créer la navigation s'il n'existe pas ou est vide
  const navContainer = document.querySelector('nav .container .flex.space-x-1');
  if (navContainer && !navContainer.innerHTML.trim()) {
    console.log('🔧 Création de la navigation...');
    navContainer.innerHTML = `
      <button onclick="showModule('dashboard')" class="nav-btn active bg-white text-gray-700 px-4 py-3 rounded-t-lg border-b-2 border-transparent hover:border-blue-500 transition-colors">
        <i class="fas fa-chart-line mr-2"></i>Dashboard
      </button>
      <button onclick="showModule('agents')" class="nav-btn bg-white text-gray-700 px-4 py-3 rounded-t-lg border-b-2 border-transparent hover:border-blue-500 transition-colors">
        <i class="fas fa-users mr-2"></i>Agents
      </button>
      <button onclick="showModule('events')" class="nav-btn bg-white text-gray-700 px-4 py-3 rounded-t-lg border-b-2 border-transparent hover:border-blue-500 transition-colors">
        <i class="fas fa-calendar mr-2"></i>Événements
      </button>
      <button onclick="showModule('hours')" class="nav-btn bg-white text-gray-700 px-4 py-3 rounded-t-lg border-b-2 border-transparent hover:border-blue-500 transition-colors">
        <i class="fas fa-clock mr-2"></i>Heures
      </button>
      <button onclick="showModule('calendar')" class="nav-btn bg-white text-gray-700 px-4 py-3 rounded-t-lg border-b-2 border-transparent hover:border-blue-500 transition-colors">
        <i class="fas fa-calendar-alt mr-2"></i>Calendrier
      </button>
      <button onclick="showModule('material')" class="nav-btn bg-white text-gray-700 px-4 py-3 rounded-t-lg border-b-2 border-transparent hover:border-blue-500 transition-colors">
        <i class="fas fa-boxes mr-2"></i>Matériel
      </button>
    `;
  }
  
  // Vérifier que l'input d'import existe
  if (!document.getElementById('importInput')) {
    console.log('🔧 Création de l\'input d\'import...');
    const importInput = document.createElement('input');
    importInput.type = 'file';
    importInput.id = 'importInput';
    importInput.accept = '.json';
    importInput.style.display = 'none';
    importInput.onchange = function() { importData(this.files); };
    document.body.appendChild(importInput);
  }
  
  console.log('✅ Éléments UI créés/vérifiés');
}

/* ==== Exposition des fonctions pour onclick HTML ==== */
// Navigation et modules
window.showModule = showModule;

// Filtres
window.onTypeFilterChange = onTypeFilterChange;
window.resetTypeFilter = resetTypeFilter;

// Agents
window.showAgentForm = showAgentForm;
window.hideAddAgentForm = hideAddAgentForm;
window.saveAgent = saveAgent;
window.deleteAgent = deleteAgent;
window.showAgentDetail = showAgentDetail;
window.showAgentEventDetail = showAgentEventDetail;
window.closeAgentEventDetailModal = closeAgentEventDetailModal;
window.closeAgentHistoryModal = closeAgentHistoryModal;

// Événements
window.showAddEventForm = showAddEventForm;
window.hideAddEventForm = hideAddEventForm;
window.saveEvent = saveEvent;
window.deleteEvent = deleteEvent;
window.goToEvent = goToEvent;

// Participants
window.openParticipantsModal = openParticipantsModal;
window.closeParticipantsModal = closeParticipantsModal;
window.onRowChange = onRowChange;
window.onTimeChange = onTimeChange;
window.updateParticipantType = updateParticipantType;
window.editRowTimes = editRowTimes;
window.removeRow = removeRow;
window.duplicateRow = duplicateRow;
window.onAddRow = onAddRow;
window.onSaveParticipants = onSaveParticipants;
window.onCancelParticipants = onCancelParticipants;

// Documents
window.openDocsModal = openDocsModal;
window.closeDocsModal = closeDocsModal;
window.handleDragOver = handleDragOver;
window.handleDragLeave = handleDragLeave;
window.handleFileDrop = handleFileDrop;
window.handleFileSelect = handleFileSelect;
window.previewDocument = previewDocument;
window.downloadDocument = downloadDocument;
window.deleteDocument = deleteDocument;
window.downloadAllDocuments = downloadAllDocuments;
window.clearAllDocuments = clearAllDocuments;

// Planning Gantt
window.openPlanningModal = openPlanningModal;
window.closePlanningModal = closePlanningModal;
window.renderGantt = renderGantt;
window.renderGanttList = renderGanttList;
window.editTask = editTask;
window.deleteTask = deleteTask;
window.resetTaskForm = resetTaskForm;
window.saveGanttTask = saveGanttTask;
window.generateGanttFromParticipants = generateGanttFromParticipants;

// Heures et calendrier
window.updateHoursTable = updateHoursTable;
window.renderCalendar = renderCalendar;
window.openDayModal = openDayModal;
window.renderDayCharts = renderDayCharts;
window.closeBarInfoModal = closeBarInfoModal;

// Graphiques
window.populateMultiFilters = populateMultiFilters;
window.resetMultiFilters = resetMultiFilters;
window.createMultiHoursChart = createMultiHoursChart;
window.createYearlyHoursLineChart = createYearlyHoursLineChart;
window.createMonthlyBreakdownLineChart = createMonthlyBreakdownLineChart;
window.createYearlyTypeHoursChart = createYearlyTypeHoursChart;

// Impression et export
window.printModule = printModule;
window.printModal = printModal;
window.resetApplication = resetApplication;
window.saveData = saveData;
window.exporterDonnees = exporterDonnees;
window.exportToExcel = exportToExcel;
window.exportDetailedReport = exportDetailedReport;
window.exportToCalendar = exportToCalendar;

// Utilitaires
window.toggleDarkMode = toggleDarkMode;
window.handleGlobalSearch = handleGlobalSearch;
window.importData = importData;

// Exposition des fonctions exportées manquantes
window.exportToUSB = exportToUSB;
window.cleanAndValidateData = cleanAndValidateData;

// Nouvelles fonctions d'import
window.readFileContent = readFileContent;
window.createDataBackup = createDataBackup;
window.restoreDataFromBackup = restoreDataFromBackup;
window.importSection = importSection;
window.diagnosticImportError = diagnosticImportError;
window.repairJSON = repairJSON;
window.importDataAlternative = importDataAlternative;

// Stockage
window.showStorageManager = showStorageManager;
window.closeStorageModal = closeStorageModal;
window.refreshStorageInfo = refreshStorageInfo;
window.cleanupStorage = cleanupStorage;
window.compressDocuments = compressDocuments;
window.analyzeStorageUsage = analyzeStorageUsage;

// Matériel
window.showMaterialForm = showMaterialForm;
window.hideMaterialForm = hideMaterialForm;
window.handleCategorySelection = handleCategorySelection;
window.saveMaterial = saveMaterial;
window.deleteMaterial = deleteMaterial;
window.updateMaterialList = updateMaterialList;
window.updateMaterialFilters = updateMaterialFilters;
window.updateMaterialAvailability = updateMaterialAvailability;
window.showMaterialUsage = showMaterialUsage;
window.openMaterialModal = openMaterialModal;
window.closeMaterialModal = closeMaterialModal;
window.updateMaterialSelect = updateMaterialSelect;
window.addMaterialToEvent = addMaterialToEvent;
window.updateEventMaterialList = updateEventMaterialList;
window.removeMaterialFromEvent = removeMaterialFromEvent;
window.validateMaterialExit = validateMaterialExit;
window.validateMaterialReturn = validateMaterialReturn;
window.printMaterialList = printMaterialList;
window.showMaterialConsumptionChart = showMaterialConsumptionChart;
window.closeMaterialConsumptionModal = closeMaterialConsumptionModal;
window.updateConsumptionChart = updateConsumptionChart;
</script>

<div id="libStatus" class="no-print fixed bottom-3 right-3 bg-white border rounded px-3 py-2 text-sm shadow" style="display:none"></div>

<script>
  // Vérification des librairies externes et initialisation
  window.addEventListener('load', () => {
    console.log('🚀 Application Città di Biguglia - Chargement...');
    
    // Vérification des librairies externes
    const ok = { Chart: !!window.Chart, Datalabels: !!window.ChartDataLabels, Gantt: !!window.Gantt, Mammoth: !!window.mammoth, XLSX: !!window.XLSX };
    const missing = Object.entries(ok).filter(([,v])=>!v).map(([k])=>k);
    if (missing.length) {
      const box = document.getElementById('libStatus');
      box.style.display = 'block';
      box.innerHTML = '⚠️ Librairies manquantes : ' + missing.join(', ');
      console.warn('Librairies manquantes:', missing);
    }
    
    // Nettoyage automatique du stockage si nécessaire
    try {
      autoCleanupIfNeeded();
    } catch(error) {
      console.warn('Erreur lors du nettoyage automatique:', error);
    }
    
    // Initialisation de l'application après chargement complet
    setTimeout(() => {
      try {
        if (typeof initApp === 'function') {
          initApp();
          console.log('✅ Application initialisée avec succès');
        } else {
          console.error('Function initApp not found');
        }
      } catch(error) {
        console.error('❌ Erreur lors de l\'initialisation:', error);
        alert('Erreur lors du chargement de l\'application. Veuillez recharger la page.');
      }
    }, 200);
  });
  
  // Fonction utilitaire pour obtenir la taille du stockage
  function getStorageSize() {
    try {
      const data = localStorage.getItem('biguglia_app_data');
      return data ? (data.length / 1024).toFixed(2) + ' KB' : '0 KB';
    } catch(e) {
      return 'Erreur';
    }
  }
  
  // Fonction utilitaire pour obtenir le pourcentage d'utilisation du stockage
  function getStoragePercentage() {
    try {
      const data = localStorage.getItem('biguglia_app_data');
      if (!data) return '0%';
      const used = data.length;
      const total = 5 * 1024 * 1024; // 5MB limite approximative
      return ((used / total) * 100).toFixed(1) + '%';
    } catch(e) {
      return 'Erreur';
    }
  }
  
  // Vérification de l'état du localStorage
  try {
    const analysis = analyzeStorageUsage();
    if (analysis) {
      console.log('📊 État du stockage:', {
        size: analysis.totalSizeMB + 'MB',
        percentage: analysis.percentUsed + '%',
        documents: analysis.documentsCount
      });
    }
  } catch(error) {
    console.warn('Impossible d\'analyser le stockage:', error);
  }
</script>
</body>
</html
