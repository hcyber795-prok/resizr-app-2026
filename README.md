[index.html.txt](https://github.com/user-attachments/files/31869736/index.html.txt)
# resizr-app-2026
free image resizer
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Resizr — Free Private Image Resizer</title>
<meta name="description" content="Resize images to any dimension, percentage or aspect ratio — instantly, privately, and free. 100% on-device processing via Web Workers." />
<meta name="theme-color" content="#0B0F19" />
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Cdefs%3E%3ClinearGradient id='g' x1='0' y1='0' x2='1' y2='1'%3E%3Cstop stop-color='%233B82F6'/%3E%3Cstop offset='1' stop-color='%238B5CF6'/%3E%3C/linearGradient%3E%3C/defs%3E%3Crect width='64' height='64' rx='14' fill='url(%23g)'/%3E%3Cpath d='M22 14v22a6 6 0 0 0 6 6h22M42 50V28a6 6 0 0 0-6-6H14' stroke='white' stroke-width='4' fill='none' stroke-linecap='round'/%3E%3C/svg%3E" />
<script>
  (function () { try { if (localStorage.getItem('resizr-theme') === 'light') document.documentElement.setAttribute('data-theme', 'light'); } catch (e) {} })();
</script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fontsource/inter@5/400.css" />
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fontsource/inter@5/500.css" />
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fontsource/inter@5/600.css" />
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fontsource/inter@5/700.css" />
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fontsource/inter@5/800.css" />
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: { mut: 'var(--muted)', ink: 'var(--text)' },
        fontFamily: { sans: ['Inter', 'ui-sans-serif', 'system-ui', 'Segoe UI', 'Roboto', 'sans-serif'] }
      }
    }
  };
</script>
<style>
  .hidden { display: none !important; }
  .sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0; }

  /* ============ THEME TOKENS ============ */
  [data-theme="dark"] {
    --bg: #0B0F19; --text: #F1F5FB; --muted: #8B95AB;
    --surface: rgba(255,255,255,.045); --surface-2: rgba(255,255,255,.075);
    --border: rgba(255,255,255,.09);
    --field: rgba(8,12,20,.55); --field-border: rgba(255,255,255,.11);
    --track: rgba(255,255,255,.09);
    --checker: rgba(148,163,184,.12);
    --dot: rgba(148,163,184,.13);
    --overlay: rgba(7,10,18,.58);
    --shadow: rgba(0,0,0,.55);
    --glow-a: .17; --glow-b: .15;
    color-scheme: dark;
  }
  [data-theme="light"] {
    --bg: #F3F5FA; --text: #101828; --muted: #5D6879;
    --surface: rgba(255,255,255,.72); --surface-2: rgba(255,255,255,.94);
    --border: rgba(16,24,40,.09);
    --field: rgba(255,255,255,.9); --field-border: rgba(16,24,40,.13);
    --track: rgba(16,24,40,.1);
    --checker: rgba(16,24,40,.055);
    --dot: rgba(16,24,40,.07);
    --overlay: rgba(243,245,250,.62);
    --shadow: rgba(16,24,40,.2);
    --glow-a: .12; --glow-b: .1;
    color-scheme: light;
  }

  html, body { height: 100%; }
  body {
    font-family: 'Inter', ui-sans-serif, system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
    background: var(--bg); color: var(--text);
    min-height: 100vh; display: flex; flex-direction: column;
    overflow-x: hidden; -webkit-font-smoothing: antialiased;
    transition: background-color .4s ease, color .4s ease;
  }
  ::selection { background: rgba(139,92,246,.35); }
  ::-webkit-scrollbar { width: 10px; height: 10px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--track); border-radius: 8px; }
  :focus-visible { outline: 2px solid #6366F1; outline-offset: 2px; }

  /* ============ BACKGROUND FX ============ */
  .orb { position: fixed; border-radius: 9999px; filter: blur(110px); pointer-events: none; z-index: 0; }
  .orb-a { width: 560px; height: 560px; background: #3B82F6; top: -180px; left: -140px; opacity: var(--glow-a); }
  .orb-b { width: 640px; height: 640px; background: #8B5CF6; bottom: -240px; right: -180px; opacity: var(--glow-b); }
  .grid-overlay {
    position: fixed; inset: 0; z-index: 0; pointer-events: none;
    background-image: radial-gradient(circle, var(--dot) 1px, transparent 1px);
    background-size: 28px 28px;
    -webkit-mask-image: radial-gradient(ellipse 90% 60% at 50% 0%, black 20%, transparent 80%);
    mask-image: radial-gradient(ellipse 90% 60% at 50% 0%, black 20%, transparent 80%);
  }

  /* ============ AD SLOTS (top & bottom) ============ */
  .ad-slot {
    position: relative; z-index: 10;
    display: flex; align-items: center; justify-content: center;
    width: 100%; max-width: 760px; height: 60px;
    margin: 0 auto; padding: 0 16px;
    border: 1px dashed var(--border); border-radius: 12px;
    background: var(--surface); color: var(--muted);
    overflow: hidden;
  }
  @media (min-width: 640px) { .ad-slot { height: 90px; } }
  .ad-slot > * { max-width: 100%; max-height: 100%; }
  .ad-label { font-size: 10px; letter-spacing: .2em; text-transform: uppercase; font-weight: 700; opacity: .65; }
  .ad-wrap-top { padding: 16px 20px 0; }
  .ad-wrap-bottom { padding: 4px 20px 16px; }

  /* ============ GLASS / CARDS ============ */
  .glass {
    background: var(--surface);
    border: 1px solid var(--border);
    backdrop-filter: blur(20px) saturate(1.25);
    -webkit-backdrop-filter: blur(20px) saturate(1.25);
    border-radius: 20px;
    box-shadow: 0 24px 70px -34px var(--shadow);
  }
  .section-title {
    font-size: 11px; letter-spacing: .14em; text-transform: uppercase;
    color: var(--muted); font-weight: 700; margin-bottom: 12px;
    display: flex; align-items: center; gap: 8px;
  }
  .section-title::after { content: ''; height: 1px; flex: 1; background: linear-gradient(to right, var(--border), transparent); }

  /* ============ BUTTONS ============ */
  .btn-grad {
    display: flex; align-items: center; justify-content: center; gap: 10px;
    background-image: linear-gradient(135deg, #3B82F6 0%, #7C5CFA 55%, #8B5CF6 100%);
    color: #fff; border-radius: 14px; text-decoration: none; cursor: pointer;
    box-shadow: 0 10px 28px -12px rgba(124,92,250,.8);
    transition: transform .15s ease, box-shadow .2s ease, filter .2s ease, opacity .2s ease;
  }
  .btn-grad:hover:not(.is-disabled) { filter: brightness(1.09); transform: translateY(-1px); box-shadow: 0 14px 32px -12px rgba(124,92,250,.95); }
  .btn-grad:active:not(.is-disabled) { transform: translateY(0) scale(.99); }
  .btn-grad.is-disabled { opacity: .4; pointer-events: none; box-shadow: none; cursor: not-allowed; }
  .dl-pill { background: rgba(255,255,255,.2); padding: 2px 9px; border-radius: 999px; font-size: 11px; font-weight: 700; }

  .icon-btn {
    width: 40px; height: 40px; display: grid; place-items: center;
    border-radius: 12px; border: 1px solid var(--border);
    background: var(--surface); color: var(--muted);
    backdrop-filter: blur(12px); transition: all .2s ease;
  }
  .icon-btn:hover { color: var(--text); border-color: rgba(99,102,241,.5); transform: translateY(-1px); }

  .chip {
    border: 1px solid var(--border); background: transparent; color: var(--muted);
    border-radius: 10px; font-size: 11.5px; font-weight: 600;
    padding: 8px 6px; text-align: center; line-height: 1.15;
    transition: all .18s ease; cursor: pointer;
  }
  .chip small { display: block; font-weight: 500; opacity: .75; font-size: 10px; margin-top: 2px; }
  .chip:hover { border-color: rgba(99,102,241,.55); background: rgba(99,102,241,.09); color: var(--text); transform: translateY(-1px); }
  .chip:active { transform: translateY(0) scale(.98); }
  .chip-flash { animation: chipPop .45s ease; }
  @keyframes chipPop { 0% { box-shadow: 0 0 0 0 rgba(139,92,246,.5); } 100% { box-shadow: 0 0 0 10px rgba(139,92,246,0); } }

  .ghost-btn {
    display: inline-flex; align-items: center; gap: 5px;
    border: 1px solid var(--border); background: var(--surface-2); color: var(--muted);
    border-radius: 9px; padding: 6px 10px; font-size: 11px; font-weight: 600; transition: all .18s ease;
  }
  .ghost-btn:hover { color: var(--text); border-color: rgba(99,102,241,.5); }
  .ghost-btn.danger:hover { color: #F87171; border-color: rgba(248,113,113,.5); }

  /* ============ DROPZONE ============ */
  .dropzone {
    border: 1.5px dashed var(--field-border); border-radius: 16px;
    background: rgba(99,102,241,.035); cursor: pointer;
    transition: all .25s ease;
  }
  .dropzone:hover, .dropzone:focus-visible { border-color: rgba(99,102,241,.55); box-shadow: 0 0 0 4px rgba(99,102,241,.08); }
  .dropzone.dragover {
    border-color: rgba(99,102,241,.9); transform: scale(1.012);
    background: rgba(99,102,241,.1);
    box-shadow: 0 0 0 2px rgba(59,130,246,.75), 0 0 46px -8px rgba(139,92,246,.65);
  }
  .dz-icon {
    width: 50px; height: 50px; border-radius: 15px; display: grid; place-items: center;
    background: linear-gradient(135deg, rgba(59,130,246,.16), rgba(139,92,246,.16));
    border: 1px solid rgba(99,102,241,.25); color: #818CF8;
  }
  .accent-text { color: #818CF8; font-weight: 600; }

  /* ============ FIELDS ============ */
  .dim-field {
    flex: 1; display: flex; align-items: center; gap: 7px;
    background: var(--field); border: 1px solid var(--field-border);
    border-radius: 12px; padding: 0 12px; min-width: 0;
    transition: border-color .2s ease, box-shadow .2s ease;
  }
  .dim-field:focus-within { border-color: #6366F1; box-shadow: 0 0 0 3px rgba(99,102,241,.16); }
  .dim-field .lbl { font-size: 10px; text-transform: uppercase; letter-spacing: .08em; color: var(--muted); font-weight: 700; flex: none; }
  .dim-field input { width: 100%; min-width: 0; background: transparent; border: none; outline: none; color: var(--text); font-weight: 600; font-size: 15px; padding: 11px 0; }
  .dim-field .suffix { font-size: 11px; color: var(--muted); flex: none; }
  input[type="number"]::-webkit-outer-spin-button, input[type="number"]::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
  input[type="number"] { -moz-appearance: textfield; appearance: textfield; }

  .select-wrap { position: relative; }
  .select-wrap select {
    width: 100%; appearance: none; -webkit-appearance: none;
    background: var(--field); border: 1px solid var(--field-border);
    color: var(--text); border-radius: 12px; padding: 11px 38px 11px 13px;
    font-size: 13.5px; font-weight: 600; cursor: pointer;
    transition: border-color .2s ease, box-shadow .2s ease;
  }
  .select-wrap select:focus { outline: none; border-color: #6366F1; box-shadow: 0 0 0 3px rgba(99,102,241,.16); }
  .select-wrap .chev { position: absolute; right: 13px; top: 50%; transform: translateY(-50%); color: var(--muted); pointer-events: none; }

  .lock-btn {
    flex: none; width: 44px; height: 44px; display: grid; place-items: center;
    border-radius: 12px; border: 1px solid var(--field-border);
    background: var(--surface-2); color: var(--muted); transition: all .22s ease;
  }
  .lock-btn:hover { transform: translateY(-1px); }
  .lock-btn svg { transition: transform .3s ease; }
  .lock-btn[aria-pressed="true"] {
    color: #fff; background-image: linear-gradient(135deg, #3B82F6, #8B5CF6);
    border-color: transparent; box-shadow: 0 6px 18px -6px rgba(124,92,250,.7);
  }
  .lock-btn[aria-pressed="false"] svg { transform: rotate(-30deg); }

  /* ============ RANGES ============ */
  .range {
    -webkit-appearance: none; appearance: none; width: 100%; height: 6px; border-radius: 999px;
    background-color: var(--track);
    background-image: linear-gradient(90deg, #3B82F6, #8B5CF6);
    background-repeat: no-repeat;
    background-size: var(--fill, 50%) 100%;
    cursor: pointer; transition: opacity .2s ease;
  }
  .range::-webkit-slider-thumb {
    -webkit-appearance: none; width: 18px; height: 18px; border-radius: 50%;
    background: #fff; border: 2px solid #7C5CFA;
    box-shadow: 0 2px 10px rgba(0,0,0,.4); transition: transform .15s ease;
  }
  .range::-webkit-slider-thumb:hover { transform: scale(1.18); }
  .range::-moz-range-thumb { width: 18px; height: 18px; border-radius: 50%; background: #fff; border: 2px solid #7C5CFA; box-shadow: 0 2px 10px rgba(0,0,0,.4); }
  .range:disabled { opacity: .35; cursor: not-allowed; }

  .hint { font-size: 11px; color: var(--muted); margin-top: 7px; line-height: 1.45; }

  /* ============ FILE CARD ============ */
  .file-card {
    display: flex; align-items: center; gap: 12px; padding: 10px 12px;
    border-radius: 14px; background: var(--surface-2); border: 1px solid var(--border);
    animation: fadeScale .3s ease;
  }
  .file-thumb { width: 52px; height: 52px; border-radius: 10px; object-fit: cover; border: 1px solid var(--border); background: var(--field); flex: none; }

  /* ============ PREVIEW ============ */
  .checker {
    background-image: conic-gradient(var(--checker) 25%, transparent 0 50%, var(--checker) 0 75%, transparent 0);
    background-size: 22px 22px;
  }
  .preview-img {
    max-width: 100%; max-height: 100%; width: auto; height: auto; object-fit: contain;
    border-radius: 10px; border: 1px solid var(--border);
    box-shadow: 0 24px 70px -24px var(--shadow);
  }
  .anim-pop { animation: fadeScale .35s ease; }
  @keyframes fadeScale { from { opacity: 0; transform: scale(.985); } to { opacity: 1; transform: scale(1); } }

  .badge {
    position: absolute; z-index: 5; display: inline-flex; align-items: center; gap: 6px;
    padding: 6px 11px; border-radius: 999px; font-size: 11px; font-weight: 700;
    background: var(--surface); backdrop-filter: blur(14px); -webkit-backdrop-filter: blur(14px);
    border: 1px solid var(--border); color: var(--text);
    box-shadow: 0 4px 18px -8px var(--shadow); animation: fadeScale .3s ease;
  }

  .proc {
    position: absolute; inset: 0; z-index: 10;
    display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 16px;
    background: var(--overlay); backdrop-filter: blur(6px); -webkit-backdrop-filter: blur(6px);
  }
  .spinner { width: 34px; height: 34px; border-radius: 50%; border: 3px solid var(--border); border-top-color: #8B5CF6; animation: spin .8s linear infinite; }
  @keyframes spin { to { transform: rotate(360deg); } }
  .progress-track { width: 170px; height: 4px; border-radius: 999px; overflow: hidden; background: var(--track); }
  .progress-bar { width: 40%; height: 100%; border-radius: 999px; background: linear-gradient(90deg, #3B82F6, #8B5CF6); animation: slide 1.1s ease-in-out infinite; }
  @keyframes slide { 0% { transform: translateX(-110%); } 100% { transform: translateX(380%); } }

  .empty-ring {
    width: 112px; height: 112px; margin: 0 auto; border-radius: 50%;
    border: 1.5px dashed var(--field-border); display: grid; place-items: center;
    color: var(--muted); animation: floaty 4s ease-in-out infinite;
  }
  @keyframes floaty { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-8px); } }

  /* ============ STATS / DELTA ============ */
  .statsbar { display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between; gap: 10px; padding: 13px 18px; border-top: 1px solid var(--border); font-size: 12px; }
  .stat { display: flex; align-items: center; gap: 7px; color: var(--muted); }
  .stat b { color: var(--text); font-weight: 600; }
  .delta { padding: 4px 11px; border-radius: 999px; font-size: 11px; font-weight: 700; }
  .delta-down { color: #34D399; background: rgba(52,211,153,.12); border: 1px solid rgba(52,211,153,.3); }
  .delta-up { color: #FBBF24; background: rgba(251,191,36,.12); border: 1px solid rgba(251,191,36,.3); }
  .delta-flat { color: var(--muted); background: var(--surface-2); border: 1px solid var(--border); }

  /* ============ MISC ============ */
  .logomark {
    width: 42px; height: 42px; border-radius: 13px; display: grid; place-items: center; color: #fff;
    background-image: linear-gradient(135deg, #3B82F6, #8B5CF6);
    box-shadow: 0 8px 22px -8px rgba(124,92,250,.85);
  }
  .privacy-pill {
    display: inline-flex; align-items: center; gap: 7px; padding: 8px 14px; border-radius: 999px;
    border: 1px solid var(--border); background: var(--surface); color: var(--muted);
    font-size: 11.5px; font-weight: 600; backdrop-filter: blur(12px);
  }
  .privacy-pill .pulse-dot { width: 7px; height: 7px; border-radius: 50%; background: #34D399; box-shadow: 0 0 0 0 rgba(52,211,153,.6); animation: pulse 2.2s infinite; }
  @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(52,211,153,.55); } 70% { box-shadow: 0 0 0 7px rgba(52,211,153,0); } 100% { box-shadow: 0 0 0 0 rgba(52,211,153,0); } }

  .controls-locked { opacity: .38; pointer-events: none; filter: saturate(.6); }
  #controls { transition: opacity .3s ease, filter .3s ease; }

  .toast {
    position: fixed; bottom: 26px; left: 50%; z-index: 60;
    transform: translateX(-50%) translateY(18px); opacity: 0; pointer-events: none;
    display: flex; align-items: center; gap: 9px;
    background: var(--surface-2); border: 1px solid var(--border); color: var(--text);
    backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px);
    border-radius: 12px; padding: 11px 17px; font-size: 13px; font-weight: 500;
    box-shadow: 0 18px 44px -14px var(--shadow);
    transition: transform .3s ease, opacity .3s ease; max-width: calc(100vw - 40px);
  }
  .toast.show { transform: translateX(-50%) translateY(0); opacity: 1; }
  .toast-dot { width: 8px; height: 8px; border-radius: 50%; flex: none; background: #818CF8; }
  .toast.error .toast-dot { background: #F87171; }
  .toast.success .toast-dot { background: #34D399; }
</style>
</head>
<body>

<!-- ambient background -->
<div class="orb orb-a" aria-hidden="true"></div>
<div class="orb orb-b" aria-hidden="true"></div>
<div class="grid-overlay" aria-hidden="true"></div>

<!-- ================= HEADER ================= -->
<header class="relative z-10 mx-auto flex w-full max-w-7xl items-center justify-between px-5 pt-6 sm:px-8">
  <div class="flex items-center gap-3.5">
    <div class="logomark" aria-hidden="true">
      <svg width="21" height="21" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M6 2v14a2 2 0 0 0 2 2h14"/><path d="M18 22V8a2 2 0 0 0-2-2H2"/></svg>
    </div>
    <div>
      <h1 class="text-lg font-extrabold tracking-tight leading-none">Resizr</h1>
      <p class="text-xs mt-1" style="color:var(--muted)">Studio-grade image resizing</p>
    </div>
  </div>
  <div class="flex items-center gap-3">
    <span class="privacy-pill hidden sm:inline-flex"><span class="pulse-dot" aria-hidden="true"></span>100% private · on-device</span>
    <button id="themeToggle" type="button" class="icon-btn" aria-label="Toggle color theme" aria-pressed="false">
      <svg id="iconSun" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="4"/><path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M6.34 17.66l-1.41 1.41M19.07 4.93l-1.41 1.41"/></svg>
      <svg id="iconMoon" class="hidden" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 3a6 6 0 0 0 9 9 9 9 0 1 1-9-9Z"/></svg>
    </button>
  </div>
</header>

<!-- ================= TOP AD SLOT ================= -->
<div class="ad-wrap-top relative z-10">
  <div class="ad-slot" role="complementary" aria-label="Advertisement">
    <span class="ad-label">Advertisement</span>
  </div>
</div>

<!-- ================= MAIN ================= -->
<main class="relative z-10 mx-auto w-full max-w-7xl flex-1 px-5 py-6 sm:px-8">
  <div class="grid grid-cols-1 gap-6 lg:grid-cols-[390px_minmax(0,1fr)]">

    <!-- ===== LEFT: CONTROLS ===== -->
    <aside class="glass flex flex-col gap-6 p-5 sm:p-6" aria-label="Resize controls">

      <!-- 1 · SOURCE -->
      <section>
        <h2 class="section-title">1 · Source</h2>
        <div id="dropzone" role="button" tabindex="0"
             aria-label="Upload an image. Drag and drop a file here, press Enter to browse, or paste from the clipboard."
             class="dropzone flex select-none flex-col items-center justify-center gap-2.5 px-4 py-9 text-center">
          <div class="dz-icon" aria-hidden="true">
            <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 14.899A7 7 0 1 1 15.71 8h1.79a4.5 4.5 0 0 1 2.5 8.242"/><path d="M12 12v9"/><path d="m16 16-4-4-4 4"/></svg>
          </div>
          <p class="text-sm font-bold">Drop your image here</p>
          <p class="text-xs" style="color:var(--muted)">or <span class="accent-text">browse files</span> · paste with Ctrl/⌘+V</p>
          <p class="text-[10px] font-bold uppercase tracking-widest" style="color:var(--muted);opacity:.8">JPEG · PNG · WebP · SVG</p>
        </div>
        <input id="fileInput" type="file" class="hidden" aria-hidden="true"
               accept=".jpg,.jpeg,.png,.webp,.svg,image/jpeg,image/png,image/webp,image/svg+xml" />

        <div id="fileCard" class="file-card hidden">
          <img id="fileThumb" class="file-thumb" alt="" />
          <div class="min-w-0 flex-1">
            <p id="fileName" class="truncate text-[13px] font-semibold"></p>
            <p id="fileMeta" class="mt-0.5 text-[11px]" style="color:var(--muted)"></p>
          </div>
          <div class="flex flex-none items-center gap-1.5">
            <button id="replaceBtn" type="button" class="ghost-btn" aria-label="Replace image">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 12a9 9 0 0 1 9-9 9.75 9.75 0 0 1 6.74 2.74L21 8"/><path d="M21 3v5h-5"/><path d="M21 12a9 9 0 0 1-9 9 9.75 9.75 0 0 1-6.74-2.74L3 16"/><path d="M3 21v-5h5"/></svg>
              Replace
            </button>
            <button id="removeBtn" type="button" class="ghost-btn danger" aria-label="Remove image">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><path d="M18 6 6 18M6 6l12 12"/></svg>
            </button>
          </div>
        </div>
      </section>

      <!-- LOCKABLE CONTROLS -->
      <div id="controls" class="controls-locked flex flex-col gap-6" aria-disabled="true">

        <!-- 2 · DIMENSIONS -->
        <section>
          <h2 class="section-title">2 · Dimensions</h2>
          <div class="flex items-center gap-2">
            <label class="dim-field" for="widthInput">
              <span class="lbl">W</span>
              <input id="widthInput" type="number" min="1" max="20000" step="1" inputmode="numeric" placeholder="0" aria-label="Width in pixels" />
              <span class="suffix">px</span>
            </label>
            <button id="lockBtn" type="button" class="lock-btn" aria-pressed="true" aria-label="Lock aspect ratio" title="Lock / unlock aspect ratio">
              <svg id="iconLink" width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"/><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"/></svg>
              <svg id="iconUnlink" class="hidden" width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"/><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"/><path d="m4 4 16 16"/></svg>
            </button>
            <label class="dim-field" for="heightInput">
              <span class="lbl">H</span>
              <input id="heightInput" type="number" min="1" max="20000" step="1" inputmode="numeric" placeholder="0" aria-label="Height in pixels" />
              <span class="suffix">px</span>
            </label>
          </div>

          <div class="mt-4">
            <div class="mb-2 flex items-center justify-between text-xs">
              <label for="pctRange" style="color:var(--muted);font-weight:600">Scale by percentage</label>
              <span id="pctLabel" class="font-bold" style="background:linear-gradient(90deg,#3B82F6,#8B5CF6);-webkit-background-clip:text;background-clip:text;color:transparent">100%</span>
            </div>
            <input id="pctRange" type="range" min="10" max="200" step="1" value="100" class="range" aria-label="Scale percentage from 10 to 200" />
          </div>

          <div class="mt-5">
            <p class="mb-2 text-xs font-semibold" style="color:var(--muted)">Quick presets</p>
            <div class="grid grid-cols-2 gap-2">
              <button type="button" class="chip preset" data-w="0" data-h="0">Original size</button>
              <button type="button" class="chip preset" data-w="1080" data-h="1080">IG Post<small>1080 × 1080</small></button>
              <button type="button" class="chip preset" data-w="1080" data-h="1350">IG Portrait<small>1080 × 1350 · 4:5</small></button>
              <button type="button" class="chip preset" data-w="1080" data-h="1920">Story / Reel<small>1080 × 1920 · 9:16</small></button>
              <button type="button" class="chip preset" data-w="1280" data-h="720">YT Thumbnail<small>1280 × 720</small></button>
              <button type="button" class="chip preset" data-w="1600" data-h="900">X / Twitter Post<small>1600 × 900</small></button>
              <button type="button" class="chip preset" data-w="1500" data-h="500">X Header<small>1500 × 500</small></button>
              <button type="button" class="chip preset" data-w="1920" data-h="1080">Full HD<small>1920 × 1080</small></button>
            </div>
          </div>
        </section>

        <!-- 3 · OUTPUT -->
        <section>
          <h2 class="section-title">3 · Output</h2>
          <div class="select-wrap">
            <select id="formatSelect" aria-label="Output format">
              <option value="original">Keep original format</option>
              <option value="jpeg">JPEG — best for photos</option>
              <option value="png">PNG — lossless + transparency</option>
              <option value="webp">WebP — modern &amp; compact</option>
            </select>
            <span class="chev" aria-hidden="true">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m6 9 6 6 6-6"/></svg>
            </span>
          </div>
          <p id="svgHint" class="hint hidden">SVG sources are rasterized and exported as PNG.</p>

          <div class="mt-4">
            <div class="mb-2 flex items-center justify-between text-xs">
              <label for="qualityRange" style="color:var(--muted);font-weight:600">Quality / compression</label>
              <span id="qualityLabel" class="font-bold" style="background:linear-gradient(90deg,#3B82F6,#8B5CF6);-webkit-background-clip:text;background-clip:text;color:transparent">90%</span>
            </div>
            <input id="qualityRange" type="range" min="1" max="100" step="1" value="90" class="range" aria-label="Compression quality from 1 to 100" />
            <p id="qualityNote" class="hint">Lower quality = smaller file (JPEG &amp; WebP only).</p>
          </div>
        </section>

        <!-- DOWNLOAD -->
        <section class="mt-auto pt-1">
          <a id="downloadBtn" class="btn-grad is-disabled w-full px-5 py-3.5 text-[15px] font-bold" rel="noopener" aria-disabled="true">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><path d="m7 10 5 5 5-5"/><path d="M12 15V3"/></svg>
            <span>Download</span>
            <span id="dlSize" class="dl-pill hidden"></span>
          </a>
          <p class="mt-3 flex items-center justify-center gap-1.5 text-center text-[11px]" style="color:var(--muted)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><rect x="3" y="11" width="18" height="11" rx="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>
            Processed locally via Web Workers — never uploaded.
          </p>
        </section>
      </div>
    </aside>

    <!-- ===== RIGHT: PREVIEW ===== -->
    <section class="glass relative flex min-h-[440px] flex-col overflow-hidden lg:min-h-[620px]" aria-label="Image preview">
      <div id="previewWrap" class="checker relative flex min-h-[360px] flex-1 items-center justify-center overflow-hidden p-5 sm:p-8" aria-busy="false">

        <!-- empty state -->
        <div id="emptyState" class="px-6 text-center">
          <div class="empty-ring" aria-hidden="true">
            <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>
          </div>
          <p class="mt-5 text-sm font-bold">No image loaded</p>
          <p class="mt-1.5 text-xs leading-relaxed" style="color:var(--muted)">Drop an image anywhere in this panel,<br/>browse files, or paste from your clipboard.</p>
        </div>

        <!-- live preview -->
        <img id="previewImg" class="preview-img hidden" alt="Live preview of the resized image — you can right-click to save it" />

        <!-- overlay badges -->
        <div id="sizeBadge" class="badge left-4 top-4 hidden" title="Original size → resized size">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14"/><path d="m19 12-7 7-7-7"/></svg>
          <span id="sizeBadgeText"></span>
        </div>
        <div id="dimsBadge" class="badge right-4 top-4 hidden"></div>
        <div id="fmtBadge" class="badge bottom-4 left-4 hidden"></div>

        <!-- processing overlay -->
        <div id="processing" class="proc hidden" role="status">
          <div class="spinner" aria-hidden="true"></div>
          <p class="text-sm font-bold">Rendering pixels…</p>
          <div class="progress-track" aria-hidden="true"><div class="progress-bar"></div></div>
        </div>
      </div>

      <!-- stats bar -->
      <div class="statsbar">
        <div class="flex flex-wrap items-center gap-x-5 gap-y-2">
          <span class="stat">Original <b id="origMeta">—</b></span>
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="color:var(--muted)" aria-hidden="true"><path d="M5 12h14"/><path d="m12 5 7 7-7 7"/></svg>
          <span class="stat">Output <b id="outMeta">—</b></span>
        </div>
        <span id="deltaChip" class="delta delta-flat hidden">—</span>
      </div>
    </section>
  </div>
</main>

<!-- ================= BOTTOM AD SLOT ================= -->
<div class="ad-wrap-bottom relative z-10">
  <div class="ad-slot" role="complementary" aria-label="Advertisement">
    <span class="ad-label">Advertisement</span>
  </div>
</div>

<!-- ================= FOOTER ================= -->
<footer class="relative z-10 mx-auto w-full max-w-7xl px-5 pb-8 pt-2 sm:px-8">
  <div class="flex flex-col items-center justify-between gap-2 text-[11px] sm:flex-row" style="color:var(--muted)">
    <p>© 2026 Resizr — every pixel stays on your device.</p>
    <p class="flex items-center gap-1.5">
      <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 13c0 5-3.5 7.5-7.66 8.95a1 1 0 0 1-.67-.01C7.5 20.5 4 18 4 13V6a1 1 0 0 1 1-1c2 0 4.5-1.2 6.24-2.72a1.17 1.17 0 0 1 1.52 0C14.51 3.81 17 5 19 5a1 1 0 0 1 1 1z"/></svg>
      Zero uploads · Web Worker rendering · Canvas API
    </p>
  </div>
</footer>

<!-- toast -->
<div id="toast" class="toast" role="alert"><span class="toast-dot" aria-hidden="true"></span><span id="toastMsg"></span></div>
<p id="statusLive" class="sr-only" role="status" aria-live="polite"></p>

<!-- ================= WEB WORKER (image processing) ================= -->
<script id="resize-worker-src" type="text/plain">
self.onmessage = async function (event) {
  var data = event.data;
  var id = data.id;
  try {
    if (typeof OffscreenCanvas === 'undefined') {
      self.postMessage({ id: id, fallback: true });
      return;
    }
    var bitmap = await createImageBitmap(data.file);
    var sw = bitmap.width;
    var sh = bitmap.height;
    var tw = data.width;
    var th = data.height;
    var canvas = new OffscreenCanvas(tw, th);
    var ctx = canvas.getContext('2d');
    ctx.imageSmoothingEnabled = true;
    ctx.imageSmoothingQuality = 'high';
    if (data.mime === 'image/jpeg') {
      ctx.fillStyle = '#ffffff';
      ctx.fillRect(0, 0, tw, th);
    }
    var cw = sw;
    var ch = sh;
    var source = bitmap;
    while (cw > tw * 2 && ch > th * 2) {
      cw = Math.max(tw, Math.round(cw / 2));
      ch = Math.max(th, Math.round(ch / 2));
      var step = new OffscreenCanvas(cw, ch);
      var sctx = step.getContext('2d');
      sctx.imageSmoothingEnabled = true;
      sctx.imageSmoothingQuality = 'high';
      sctx.drawImage(source, 0, 0, cw, ch);
      source = step;
    }
    ctx.drawImage(source, 0, 0, tw, th);
    bitmap.close();
    var options = { type: data.mime };
    if (data.mime === 'image/jpeg' || data.mime === 'image/webp') {
      options.quality = data.quality;
    }
    var blob = await canvas.convertToBlob(options);
    self.postMessage({ id: id, blob: blob, width: tw, height: th });
  } catch (err) {
    self.postMessage({ id: id, error: String((err && err.message) || err) });
  }
};
</script>

<!-- ================= APP LOGIC ================= -->
<script>
(function () {
  'use strict';

  /* ---------- DOM refs ---------- */
  var $ = function (id) { return document.getElementById(id); };
  var els = {
    themeToggle: $('themeToggle'), iconSun: $('iconSun'), iconMoon: $('iconMoon'),
    fileInput: $('fileInput'), dropzone: $('dropzone'), fileCard: $('fileCard'),
    fileThumb: $('fileThumb'), fileName: $('fileName'), fileMeta: $('fileMeta'),
    replaceBtn: $('replaceBtn'), removeBtn: $('removeBtn'), controls: $('controls'),
    widthInput: $('widthInput'), heightInput: $('heightInput'), lockBtn: $('lockBtn'),
    iconLink: $('iconLink'), iconUnlink: $('iconUnlink'),
    pctRange: $('pctRange'), pctLabel: $('pctLabel'),
    formatSelect: $('formatSelect'), svgHint: $('svgHint'),
    qualityRange: $('qualityRange'), qualityLabel: $('qualityLabel'), qualityNote: $('qualityNote'),
    downloadBtn: $('downloadBtn'), dlSize: $('dlSize'),
    previewWrap: $('previewWrap'), emptyState: $('emptyState'), previewImg: $('previewImg'),
    sizeBadge: $('sizeBadge'), sizeBadgeText: $('sizeBadgeText'),
    dimsBadge: $('dimsBadge'), fmtBadge: $('fmtBadge'), processing: $('processing'),
    origMeta: $('origMeta'), outMeta: $('outMeta'), deltaChip: $('deltaChip'),
    toast: $('toast'), toastMsg: $('toastMsg'), statusLive: $('statusLive')
  };

  var inIframe = false;
  try { inIframe = window.self !== window.top; } catch (e) { inIframe = true; }

  /* ---------- State ---------- */
  var MAX_DIM = 20000;
  var MAX_AREA = 52428800;
  function freshState() {
    return {
      file: null, name: 'image', mime: 'image/png', isSVG: false,
      url: null, imgEl: null, origW: 0, origH: 0, size: 0,
      targetW: 0, targetH: 0, locked: true,
      format: 'original', quality: 90,
      resultBlob: null, resultURL: null
    };
  }
  var state = freshState();
  var jobId = 0;
  var debounceTimer = null;
  var workerReady = true;
  var rasterImg = null;
  var pendingJobs = new Map();
  var procWatchdog = null;

  /* ---------- Worker bootstrap ---------- */
  var resizeWorker = null;
  try {
    if (typeof Worker === 'undefined') throw new Error('no workers');
    var workerSrc = $('resize-worker-src').textContent;
    var workerBlobUrl = URL.createObjectURL(new Blob([workerSrc], { type: 'text/javascript' }));
    resizeWorker = new Worker(workerBlobUrl);
    resizeWorker.onmessage = onWorkerMessage;
    resizeWorker.onerror = function () { disableWorker(); };
  } catch (err) {
    resizeWorker = null;
    workerReady = false;
  }

  function disableWorker() {
    workerReady = false;
    pendingJobs.forEach(function (job) { clearTimeout(job.timer); job.resolve(null); });
    pendingJobs.clear();
  }

  function onWorkerMessage(event) {
    var msg = event.data;
    var job = pendingJobs.get(msg.id);
    if (!job) return;
    pendingJobs.delete(msg.id);
    clearTimeout(job.timer);
    if (msg.fallback) { job.resolve(null); return; }
    if (msg.error) { job.resolve(null); return; }
    job.resolve(msg.blob);
  }

  function workerResize(file, w, h, mime, quality, id) {
    return new Promise(function (resolve) {
      var timer = setTimeout(function () {
        if (pendingJobs.has(id)) { pendingJobs.delete(id); disableWorker(); resolve(null); }
      }, 8000);
      pendingJobs.set(id, { resolve: resolve, timer: timer });
      try {
        resizeWorker.postMessage({ id: id, file: file, width: w, height: h, mime: mime, quality: quality });
      } catch (err) {
        clearTimeout(timer);
        pendingJobs.delete(id);
        disableWorker();
        resolve(null);
      }
    });
  }

  /* ---------- Utilities ---------- */
  function clampDim(value) {
    return Math.min(MAX_DIM, Math.max(1, Math.round(value)));
  }
  function formatBytes(bytes) {
    if (!isFinite(bytes) || bytes < 0) return '—';
    if (bytes < 1024) return bytes + ' B';
    if (bytes < 1048576) return (bytes / 1024).toFixed(bytes < 10240 ? 1 : 0) + ' KB';
    return (bytes / 1048576).toFixed(2) + ' MB';
  }
  function updateRangeFill(rangeEl) {
    var min = parseFloat(rangeEl.min), max = parseFloat(rangeEl.max), val = parseFloat(rangeEl.value);
    var pct = ((val - min) / (max - min)) * 100;
    rangeEl.style.setProperty('--fill', pct + '%');
  }
  function resolveMime() {
    switch (state.format) {
      case 'jpeg': return 'image/jpeg';
      case 'png': return 'image/png';
      case 'webp': return 'image/webp';
      default: return state.isSVG ? 'image/png' : state.mime;
    }
  }
  function extForType(type) {
    if (type === 'image/jpeg') return 'jpg';
    if (type === 'image/webp') return 'webp';
    if (type === 'image/svg+xml') return 'svg';
    return 'png';
  }
  function currentFilename() {
    var type = (state.resultBlob && state.resultBlob.type) || resolveMime();
    return state.name + '-' + state.targetW + 'x' + state.targetH + '.' + extForType(type);
  }

  var toastTimer = null;
  function toast(message, kind) {
    els.toastMsg.textContent = message;
    els.toast.classList.remove('error', 'success');
    if (kind) els.toast.classList.add(kind);
    els.toast.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(function () { els.toast.classList.remove('show'); }, 3800);
  }

  /* ---------- Download link management ---------- */
  function setDownloadLink() {
    var btn = els.downloadBtn;
    if (!state.resultBlob || !state.resultURL) {
      btn.classList.add('is-disabled');
      btn.setAttribute('aria-disabled', 'true');
      btn.removeAttribute('href');
      btn.removeAttribute('download');
      els.dlSize.classList.add('hidden');
      return;
    }
    btn.href = state.resultURL;
    btn.download = currentFilename();
    btn.classList.remove('is-disabled');
    btn.removeAttribute('aria-disabled');
  }

  /* ---------- Processing overlay (with watchdog safety net) ---------- */
  function setProcessing(on) {
    els.processing.classList.toggle('hidden', !on);
    els.previewWrap.setAttribute('aria-busy', String(on));
    clearTimeout(procWatchdog);
    if (on) {
      procWatchdog = setTimeout(function () {
        els.processing.classList.add('hidden');
        els.previewWrap.setAttribute('aria-busy', 'false');
        toast('Processing is taking unusually long — try smaller dimensions.', 'error');
      }, 20000);
    }
  }

  /* ---------- Theme ---------- */
  function applyTheme(theme) {
    document.documentElement.setAttribute('data-theme', theme);
    try { localStorage.setItem('resizr-theme', theme); } catch (e) {}
    els.themeToggle.setAttribute('aria-pressed', String(theme === 'light'));
    els.iconSun.classList.toggle('hidden', theme !== 'dark');
    els.iconMoon.classList.toggle('hidden', theme === 'dark');
  }
  els.themeToggle.addEventListener('click', function () {
    applyTheme(document.documentElement.getAttribute('data-theme') === 'light' ? 'dark' : 'light');
  });
  applyTheme(document.documentElement.getAttribute('data-theme') === 'light' ? 'light' : 'dark');

  /* ---------- Loading files ---------- */
  function imageDimensions(url) {
    return new Promise(function (resolve, reject) {
      var probe = new Image();
      probe.onload = function () { resolve({ w: probe.naturalWidth, h: probe.naturalHeight }); };
      probe.onerror = function () { reject(new Error('Unsupported or corrupted image')); };
      probe.src = url;
    });
  }

  function loadSvg(file, url) {
    return new Promise(function (resolve, reject) {
      var img = new Image();
      img.onload = function () {
        var w = img.naturalWidth, h = img.naturalHeight;
        if (!w || !h) {
          var ratio = 1;
          var finish = function () {
            w = 1600; h = Math.max(1, Math.round(1600 / ratio));
            img.width = w; img.height = h;
            resolve({ img: img, w: w, h: h });
          };
          file.text().then(function (text) {
            var match = text.match(/viewBox\s*=\s*["']?\s*([-\d.eE+]+)[\s,]+([-\d.eE+]+)[\s,]+([-\d.eE+]+)[\s,]+([-\d.eE+]+)/i);
            if (match) {
              var vw = parseFloat(match[3]) - parseFloat(match[1]);
              var vh = parseFloat(match[4]) - parseFloat(match[2]);
              if (vw > 0 && vh > 0) ratio = vw / vh;
            }
            finish();
          }).catch(finish);
        } else {
          resolve({ img: img, w: w, h: h });
        }
      };
      img.onerror = function () { reject(new Error('Could not parse SVG file')); };
      img.src = url;
    });
  }

  function loadFile(file) {
    var validType = ['image/jpeg', 'image/png', 'image/webp', 'image/svg+xml'].indexOf(file.type) !== -1;
    var validExt = /\.(jpe?g|png|webp|svg)$/i.test(file.name || '');
    if (!validType && !validExt) { toast('Unsupported format — use JPEG, PNG, WebP or SVG.', 'error'); return; }
    if (file.size > 80 * 1024 * 1024) { toast('File too large — 80 MB maximum.', 'error'); return; }

    resetCurrent();

    state.file = file;
    state.size = file.size;
    state.name = (file.name || 'image').replace(/\.[^.]+$/, '') || 'image';
    state.isSVG = file.type === 'image/svg+xml' || /\.svg$/i.test(file.name);
    state.mime = file.type || 'image/png';
    state.url = URL.createObjectURL(file);

    var ready = state.isSVG
      ? loadSvg(file, state.url).then(function (res) {
          state.imgEl = res.img; state.origW = res.w; state.origH = res.h;
        })
      : imageDimensions(state.url).then(function (dim) {
          if (!dim.w || !dim.h) throw new Error('Unsupported or corrupted image');
          state.origW = dim.w; state.origH = dim.h;
        });

    ready.then(function () {
      state.targetW = state.origW;
      state.targetH = state.origH;
      state.resultBlob = state.file;
      state.resultURL = state.url;
      updateUIAfterLoad();
      setPreview(state.url);
      updateStats();
      setDownloadLink();
      els.statusLive.textContent = 'Image loaded: ' + state.origW + ' by ' + state.origH + ' pixels.';
    }).catch(function (err) {
      toast(err.message || 'Could not read that image.', 'error');
      resetCurrent();
    });
  }

  function updateUIAfterLoad() {
    els.dropzone.classList.add('hidden');
    els.fileCard.classList.remove('hidden');
    els.fileThumb.src = state.url;
    els.fileName.textContent = state.file.name || 'image';
    els.fileMeta.textContent = state.origW + ' × ' + state.origH + ' px · ' + formatBytes(state.size);
    els.controls.classList.remove('controls-locked');
    els.controls.setAttribute('aria-disabled', 'false');
    els.emptyState.classList.add('hidden');
    els.sizeBadge.classList.remove('hidden');
    els.dimsBadge.classList.remove('hidden');
    els.fmtBadge.classList.remove('hidden');
    syncDimUI();
    updateOutputUI();
  }

  function resetCurrent() {
    jobId++;
    clearTimeout(debounceTimer);
    pendingJobs.forEach(function (job) { clearTimeout(job.timer); });
    pendingJobs.clear();
    var resultUrl = state.resultURL, sourceUrl = state.url;
    if (resultUrl && resultUrl !== sourceUrl) URL.revokeObjectURL(resultUrl);
    if (sourceUrl) URL.revokeObjectURL(sourceUrl);
    state = freshState();
    rasterImg = null;

    els.dropzone.classList.remove('hidden');
    els.fileCard.classList.add('hidden');
    els.controls.classList.add('controls-locked');
    els.controls.setAttribute('aria-disabled', 'true');
    els.emptyState.classList.remove('hidden');
    els.previewImg.classList.add('hidden');
    els.previewImg.removeAttribute('src');
    els.sizeBadge.classList.add('hidden');
    els.dimsBadge.classList.add('hidden');
    els.fmtBadge.classList.add('hidden');
    els.widthInput.value = '';
    els.heightInput.value = '';
    els.pctRange.value = 100;
    els.pctLabel.textContent = '100%';
    updateRangeFill(els.pctRange);
    els.origMeta.textContent = '—';
    els.outMeta.textContent = '—';
    els.deltaChip.classList.add('hidden');
    setDownloadLink();
    setProcessing(false);
  }

  /* ---------- UI sync ---------- */
  function syncDimUI() {
    els.widthInput.value = state.targetW;
    els.heightInput.value = state.targetH;
    if (state.origW) {
      var pct = Math.min(200, Math.max(10, Math.round((state.targetW / state.origW) * 100)));
      els.pctRange.value = pct;
      els.pctLabel.textContent = pct + '%';
      updateRangeFill(els.pctRange);
    }
  }

  function updateOutputUI() {
    var mime = resolveMime();
    var lossy = mime === 'image/jpeg' || mime === 'image/webp';
    els.qualityRange.disabled = !lossy;
    els.qualityNote.textContent = lossy
      ? 'Lower quality = smaller file (JPEG & WebP only).'
      : 'PNG is lossless — quality does not apply.';
    els.qualityLabel.textContent = state.quality + '%';
    els.svgHint.classList.toggle('hidden', !(state.isSVG && state.format === 'original'));
  }

  function setPreview(url) {
    els.previewImg.classList.remove('hidden');
    els.previewImg.classList.remove('anim-pop');
    void els.previewImg.offsetWidth;
    els.previewImg.src = url;
    els.previewImg.classList.add('anim-pop');
  }

  function updateStats() {
    var blob = state.resultBlob;
    var outSize = blob ? blob.size : 0;
    els.origMeta.textContent = state.origW + ' × ' + state.origH + ' · ' + formatBytes(state.size);
    els.outMeta.textContent = state.targetW + ' × ' + state.targetH + ' · ' + formatBytes(outSize);
    els.sizeBadgeText.textContent = formatBytes(state.size) + '  →  ' + formatBytes(outSize);
    els.dimsBadge.textContent = state.targetW + ' × ' + state.targetH + ' px';

    var outType = (blob && blob.type) ? blob.type : resolveMime();
    var lossy = outType === 'image/jpeg' || outType === 'image/webp';
    els.fmtBadge.textContent = extForType(outType).toUpperCase() + (lossy ? ' · q' + state.quality : '');

    var chip = els.deltaChip;
    chip.classList.remove('hidden');
    var diff = outSize - state.size;
    if (Math.abs(diff) < 1024) {
      chip.textContent = 'unchanged'; chip.className = 'delta delta-flat';
    } else if (diff < 0) {
      chip.textContent = '↓ ' + Math.abs(Math.round((diff / state.size) * 100)) + '% smaller';
      chip.className = 'delta delta-down';
    } else {
      chip.textContent = '↑ ' + Math.round((diff / state.size) * 100) + '% larger';
      chip.className = 'delta delta-up';
    }
    els.dlSize.textContent = formatBytes(outSize);
    els.dlSize.classList.remove('hidden');
  }

  /* ---------- Resize pipeline ---------- */
  function scheduleResize() {
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(runResize, 320);
  }

  function applyResult(blob) {
    var previous = state.resultURL;
    state.resultBlob = blob;
    state.resultURL = URL.createObjectURL(blob);
    if (previous && previous !== state.url) URL.revokeObjectURL(previous);
    setPreview(state.resultURL);
    updateStats();
    setDownloadLink();
    els.statusLive.textContent = 'Resized to ' + state.targetW + ' by ' + state.targetH + ' pixels, ' + formatBytes(blob.size) + '.';
  }

  async function runResize() {
    if (!state.file || state.targetW < 1 || state.targetH < 1) return;
    var tw = state.targetW, th = state.targetH;
    if (tw * th > MAX_AREA) {
      toast('Output too large — please reduce the dimensions.', 'error');
      return;
    }
    var id = ++jobId;
    setProcessing(true);
    var mime = resolveMime();
    var quality = state.quality / 100;

    try {
      if (tw === state.origW && th === state.origH && state.format === 'original' && !state.isSVG) {
        if (id === jobId) {
          var previous = state.resultURL;
          state.resultBlob = state.file;
          state.resultURL = state.url;
          if (previous && previous !== state.url) URL.revokeObjectURL(previous);
          setPreview(state.url);
          updateStats();
          setDownloadLink();
        }
      } else if (state.isSVG) {
        var svgBlob = await mainThreadResize(state.imgEl, state.origW, state.origH, tw, th, mime, quality);
        if (id === jobId) applyResult(svgBlob);
      } else {
        var blob = null;
        if (workerReady && resizeWorker) {
          blob = await workerResize(state.file, tw, th, mime, quality, id);
          if (id !== jobId) return;
        }
        if (!blob) {
          var fallbackImg = await ensureRasterSource();
          if (id !== jobId) return;
          blob = await mainThreadResize(fallbackImg, state.origW, state.origH, tw, th, mime, quality);
        }
        if (id === jobId) applyResult(blob);
      }
    } catch (err) {
      if (id === jobId) {
        toast(err && err.message ? err.message : 'Processing failed.', 'error');
      }
    } finally {
      if (id === jobId) setProcessing(false);
    }
  }

  function ensureRasterSource() {
    return new Promise(function (resolve, reject) {
      if (rasterImg && rasterImg.src === state.url && rasterImg.complete && rasterImg.naturalWidth) {
        resolve(rasterImg);
        return;
      }
      rasterImg = new Image();
      rasterImg.onload = function () { resolve(rasterImg); };
      rasterImg.onerror = function () { reject(new Error('Could not decode image')); };
      rasterImg.src = state.url;
    });
  }

  function mainThreadResize(source, sw, sh, tw, th, mime, quality) {
    return new Promise(function (resolve, reject) {
      try {
        var canvas = document.createElement('canvas');
        canvas.width = tw; canvas.height = th;
        var ctx = canvas.getContext('2d');
        ctx.imageSmoothingEnabled = true;
        ctx.imageSmoothingQuality = 'high';
        if (mime === 'image/jpeg') { ctx.fillStyle = '#ffffff'; ctx.fillRect(0, 0, tw, th); }

        var cw = sw, ch = sh, current = source;
        while (cw > tw * 2 && ch > th * 2) {
          cw = Math.max(tw, Math.round(cw / 2));
          ch = Math.max(th, Math.round(ch / 2));
          var step = document.createElement('canvas');
          step.width = cw; step.height = ch;
          var sctx = step.getContext('2d');
          sctx.imageSmoothingEnabled = true;
          sctx.imageSmoothingQuality = 'high';
          sctx.drawImage(current, 0, 0, cw, ch);
          current = step;
        }
        ctx.drawImage(current, 0, 0, tw, th);

        canvas.toBlob(function (blob) {
          if (blob) resolve(blob);
          else reject(new Error('Image encoding failed'));
        }, mime, (mime === 'image/jpeg' || mime === 'image/webp') ? quality : undefined);
      } catch (err) {
        reject(err && err.message && /tainted|security/i.test(err.message)
          ? new Error('This image contains external references and cannot be exported.')
          : err);
      }
    });
  }

  /* ---------- Events: download ---------- */
  els.downloadBtn.addEventListener('click', function (event) {
    if (!state.resultBlob || !state.resultURL) { event.preventDefault(); return; }

    var filename = currentFilename();
    var isTouch = ('ontouchstart' in window) || navigator.maxTouchPoints > 0;

    if (isTouch && navigator.canShare && typeof File !== 'undefined') {
      try {
        var shareFile = new File([state.resultBlob], filename, { type: state.resultBlob.type || 'image/png' });
        if (navigator.canShare({ files: [shareFile] })) {
          event.preventDefault();
          navigator.share({ files: [shareFile], title: 'Resized image' })
            .then(function () { toast('Image saved via share sheet', 'success'); })
            .catch(function (shareErr) {
              if (shareErr && shareErr.name === 'AbortError') return;
              anchorDownloadFallback(filename);
            });
          return;
        }
      } catch (e) { /* fall through to normal download */ }
    }

    toast('Download started — ' + filename, 'success');
  });

  function anchorDownloadFallback(filename) {
    var anchor = document.createElement('a');
    anchor.href = state.resultURL;
    anchor.download = filename;
    anchor.rel = 'noopener';
    if (inIframe) anchor.target = '_blank';
    document.body.appendChild(anchor);
    anchor.click();
    anchor.remove();
  }

  /* in embedded previews, open in a new tab so the save still works (no visible message) */
  if (inIframe) {
    els.downloadBtn.setAttribute('target', '_blank');
  }

  /* ---------- Events: dimensions ---------- */
  function onDimInput(which) {
    if (!state.file) return;
    var el = which === 'w' ? els.widthInput : els.heightInput;
    var value = parseInt(el.value, 10);
    if (!isFinite(value) || value < 1) return;

    if (which === 'w') {
      var ratioH = state.targetH / state.targetW;
      state.targetW = value;
      if (state.locked) state.targetH = clampDim(value * ratioH);
    } else {
      var ratioW = state.targetW / state.targetH;
      state.targetH = value;
      if (state.locked) state.targetW = clampDim(value * ratioW);
    }
    syncDimUI();
    scheduleResize();
  }

  function onDimCommit(which) {
    if (!state.file) return;
    var el = which === 'w' ? els.widthInput : els.heightInput;
    var value = parseInt(el.value, 10);
    if (!isFinite(value)) {
      el.value = which === 'w' ? state.targetW : state.targetH;
      return;
    }
    if (which === 'w') {
      var ratioH = state.targetH / state.targetW;
      state.targetW = clampDim(value);
      if (state.locked) state.targetH = clampDim(state.targetW * ratioH);
    } else {
      var ratioW = state.targetW / state.targetH;
      state.targetH = clampDim(value);
      if (state.locked) state.targetW = clampDim(state.targetH * ratioW);
    }
    syncDimUI();
    scheduleResize();
  }

  els.widthInput.addEventListener('input', function () { onDimInput('w'); });
  els.heightInput.addEventListener('input', function () { onDimInput('h'); });
  els.widthInput.addEventListener('change', function () { onDimCommit('w'); });
  els.heightInput.addEventListener('change', function () { onDimCommit('h'); });

  els.lockBtn.addEventListener('click', function () {
    state.locked = !state.locked;
    els.lockBtn.setAttribute('aria-pressed', String(state.locked));
    els.iconLink.classList.toggle('hidden', !state.locked);
    els.iconUnlink.classList.toggle('hidden', state.locked);
    els.statusLive.textContent = state.locked ? 'Aspect ratio locked.' : 'Aspect ratio unlocked.';
  });

  els.pctRange.addEventListener('input', function () {
    var pct = parseInt(els.pctRange.value, 10);
    els.pctLabel.textContent = pct + '%';
    updateRangeFill(els.pctRange);
    if (!state.file) return;
    state.targetW = clampDim(state.origW * pct / 100);
    state.targetH = clampDim(state.origH * pct / 100);
    els.widthInput.value = state.targetW;
    els.heightInput.value = state.targetH;
    scheduleResize();
  });

  /* ---------- Events: presets ---------- */
  document.querySelectorAll('.preset').forEach(function (btn) {
    btn.addEventListener('click', function () {
      if (!state.file) return;
      var w = parseInt(btn.dataset.w, 10) || 0;
      var h = parseInt(btn.dataset.h, 10) || 0;
      state.targetW = w || state.origW;
      state.targetH = h || state.origH;
      syncDimUI();
      scheduleResize();
      btn.classList.remove('chip-flash');
      void btn.offsetWidth;
      btn.classList.add('chip-flash');
    });
  });

  /* ---------- Events: output ---------- */
  els.formatSelect.addEventListener('change', function () {
    state.format = els.formatSelect.value;
    updateOutputUI();
    scheduleResize();
  });

  els.qualityRange.addEventListener('input', function () {
    state.quality = parseInt(els.qualityRange.value, 10);
    els.qualityLabel.textContent = state.quality + '%';
    updateRangeFill(els.qualityRange);
    scheduleResize();
  });

  /* ---------- Events: upload ---------- */
  els.fileInput.addEventListener('change', function (event) {
    if (event.target.files && event.target.files[0]) loadFile(event.target.files[0]);
    event.target.value = '';
  });
  els.dropzone.addEventListener('click', function () { els.fileInput.click(); });
  els.dropzone.addEventListener('keydown', function (event) {
    if (event.key === 'Enter' || event.key === ' ') { event.preventDefault(); els.fileInput.click(); }
  });
  els.replaceBtn.addEventListener('click', function () { els.fileInput.click(); });
  els.removeBtn.addEventListener('click', function () {
    resetCurrent();
    els.statusLive.textContent = 'Image removed.';
  });

  function makeDropTarget(el) {
    el.addEventListener('dragenter', function (event) { event.preventDefault(); el.classList.add('dragover'); });
    el.addEventListener('dragover', function (event) { event.preventDefault(); el.classList.add('dragover'); });
    el.addEventListener('dragleave', function (event) {
      if (!el.contains(event.relatedTarget)) el.classList.remove('dragover');
    });
    el.addEventListener('drop', function (event) {
      event.preventDefault();
      el.classList.remove('dragover');
      var dropped = event.dataTransfer && event.dataTransfer.files && event.dataTransfer.files[0];
      if (dropped) loadFile(dropped);
    });
  }
  makeDropTarget(els.dropzone);
  makeDropTarget(els.previewWrap);
  ['dragover', 'drop'].forEach(function (eventName) {
    window.addEventListener(eventName, function (event) { event.preventDefault(); });
  });

  window.addEventListener('paste', function (event) {
    var items = event.clipboardData && event.clipboardData.items;
    if (!items) return;
    for (var i = 0; i < items.length; i++) {
      if (items[i].type.indexOf('image/') === 0) {
        var pasted = items[i].getAsFile();
        if (pasted) { event.preventDefault(); loadFile(pasted); break; }
      }
    }
  });

  /* ---------- Init ---------- */
  updateRangeFill(els.pctRange);
  updateRangeFill(els.qualityRange);
})();
</script>
</body>
</html>
