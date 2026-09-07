<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Control de Cambio de Guardia</title>

<!-- Firebase SDKs -->
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-storage-compat.js"></script>

<style>
:root{--bg:#f3f5f7;--card:#fff;--text:#17212b;--muted:#697586;--line:#dfe5eb;--accent:#1f5eff;--good:#168a55;--warn:#b87500;--bad:#c23838}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif}
.container{max-width:1250px;margin:auto;padding:24px}
header{display:flex;justify-content:space-between;gap:20px;align-items:center;margin-bottom:20px;flex-wrap:wrap}h1{margin:0 0 5px;font-size:28px}.sub{color:var(--muted)}
.tabs{display:flex;gap:8px;margin-bottom:18px}.tab{border:1px solid var(--line);background:#fff;padding:11px 16px;border-radius:10px;cursor:pointer;font-weight:700}.tab.active{background:var(--accent);color:white;border-color:var(--accent)}
.card{background:var(--card);border:1px solid var(--line);border-radius:15px;padding:20px;box-shadow:0 7px 25px rgba(20,30,40,.06);margin-bottom:16px}
h2{font-size:19px;margin:0 0 15px}
.section-title{font-size:14px;font-weight:700;color:var(--muted);margin:16px 0 10px;text-transform:uppercase;letter-spacing:0.5px}
.grid{display:grid;grid-template-columns:repeat(3,1fr);gap:14px}
.grid-2{display:grid;grid-template-columns:repeat(2,1fr);gap:14px}
.field{display:flex;flex-direction:column;gap:6px}.field label{font-size:13px;font-weight:700;color:var(--muted)}input,select,textarea{width:100%;border:1px solid var(--line);border-radius:9px;padding:11px;background:#fff;color:var(--text);font:inherit}textarea{min-height:80px;resize:vertical}
.unit{border:1px solid var(--line);border-radius:13px;padding:16px;margin-top:14px;background:#fff}
.unit-header{display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid var(--line);padding-bottom:10px;margin-bottom:12px;flex-wrap:wrap;gap:10px}
.unit-header h3{margin:0;color:var(--accent)}
.checkgrid{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.check{display:grid;grid-template-columns:1fr 145px;gap:10px;align-items:center;padding:10px;background:#fafbfc;border-radius:9px;border:1px solid #edf0f3}
.check span{font-size:14px;font-weight:600}
.actions{display:flex;gap:10px;flex-wrap:wrap}.btn{border:0;border-radius:10px;padding:12px 17px;font-weight:800;cursor:pointer}.primary{background:var(--accent);color:white}.secondary{background:#e9edf2;color:var(--text)}.danger{background:#fdeaea;color:var(--bad)}
.btn-sm{padding:6px 12px;font-size:12px;border-radius:7px}
.dashboard{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}.metric{padding:17px;border:1px solid var(--line);border-radius:12px;background:#fff}.metric small{display:block;color:var(--muted);font-weight:700}.metric strong{font-size:26px;display:block;margin-top:5px}.good{color:var(--good)}.warn{color:var(--warn)}.bad{color:var(--bad)}
.empty{padding:35px;text-align:center;color:var(--muted)}
.record{border:1px solid var(--line);border-radius:12px;padding:15px;margin-bottom:10px;background:#fff}.recordtop{display:flex;justify-content:space-between;gap:10px;align-items:center}.pill{padding:5px 9px;border-radius:99px;background:#eaf8f1;color:var(--good);font-size:12px;font-weight:800}
.items-status-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:6px;margin-top:6px;background:#fff;padding:8px 12px;border-radius:8px;border:1px solid #edf2f7}
.user-bar{display:flex;align-items:center;gap:12px;font-size:14px;font-weight:600}
.photo-preview-container{display:flex;gap:8px;flex-wrap:wrap;margin-top:8px}
.photo-thumb{width:70px;height:70px;object-fit:cover;border-radius:8px;border:1px solid var(--line);cursor:pointer}
#auth-card{max-width:400px;margin:60px auto}
@media(max-width:800px){.grid,.grid-2,.dashboard{grid-template-columns:1fr 1fr}.checkgrid{grid-template-columns:1fr}.check{grid-template-columns:1fr 130px}}
@media(max-width:520px){.container{padding:13px}.grid,.grid-2,.dashboard,.items-status-grid{grid-template-columns:1fr}header{display:block}}
</style>
</head>
<body>
<div class="container">

<header>
  <div>
    <h1>Control de Cambio de Guardia</h1>
    <div class="sub">Gestión de unidades y reportes en la nube</div>
  </div>
  <div id="user-info" class="user-bar" style="display:none">
    <span id="user-email"></span>
    <button class="btn secondary btn-sm" onclick="logout()">Cerrar sesión</button>
  </div>
</header>

<!-- PANTALLA DE LOGIN -->
<div id="auth-card" class="card">
  <h2>Iniciar Sesión</h2>
  <div class="field" style="margin-bottom:12px">
    <label>Correo Electrónico</label>
    <input type="email" id="auth-email" placeholder="chofer@empresa.com">
  </div>
  <div class="field" style="margin-bottom:18px">
    <label>Contraseña</label>
    <input type="password" id="auth-pass" placeholder="******">
  </div>
  <div class="actions">
    <button class="btn primary" onclick="login()" style="width:100%">Ingresar</button>
    <button class="btn secondary" onclick="register()" style="width:100%;margin-top:5px">Registrarse</button>
  </div>
</div>

<!-- CONTENIDO PRINCIPAL -->
<div id="main-app" style="display:none">
  <div class="tabs">
    <button class="tab active" onclick="showTab('cargar',this)">Cargar guardia</button>
    <button class="tab" onclick="showTab('historial',this)">Mis Guardias</button>
  </div>

  <section id="cargar">
  <div class="card">
  <h2>1. Datos de la guardia</h2>
  <div class="grid">
    <div class="field"><label>Fecha</label><input id="fecha" type="date"></div>
    <div class="field" style="grid-column: span 2"><label>Chofer</label><input id="chofer" placeholder="Nombre y apellido del chofer"></div>
  </div>
  </div>

  <div class="card">
  <h2>2. Estado, Verificaciones, Fotos y Observaciones por Unidad</h2>
  <div id="units"></div>
  </div>

  <div class="actions">
  <button id="btn-guardar" class="btn primary" onclick="guardar()">Guardar cambio de guardia</button>
  <button class="btn secondary" onclick="limpiar()">Limpiar formulario</button>
  </div>
  </section>

  <section id="historial" style="display:none">
  <div class="card">
  <h2>Mi Resumen</h2>
  <div class="dashboard">
  <div class="metric"><small>Guardias registradas</small><strong id="total">0</strong></div>
  <div class="metric"><small>Unidades revisadas</small><strong id="unidades">0</strong></div>
  <div class="metric"><small>Con observaciones</small><strong class="warn" id="conObs">0</strong></div>
  <div class="metric"><small>Móviles No Operativos</small><strong class="bad" id="noOperativos">0</strong></div>
  </div>
  </div>
  <div id="records"></div>
  </section>
</div>

</div>

<script>
// Configuración de tu proyecto traslados-b4628
const firebaseConfig = {
  apiKey: "AIzaSyAW7vN0udDac3JKf20UEQuGDs9c0b_eJ-k",
  authDomain: "traslados-b4628.firebaseapp.com",
@@ -144,7 +143,6 @@
 return n.includes("oxígeno") || n.includes("oxigeno") || n.includes("tubo");
}

// Control de Sesión
auth.onAuthStateChanged(user => {
  if (user) {
    document.getElementById("auth-card").style.display = "none";
@@ -208,7 +206,7 @@
   </div>

   <div class="section-title">Verificaciones del Móvil</div>
   <div class="grid-2">
   <div class="grid">
     <div class="field">
       <label>¿Se encontró residuos patogénicos?</label>
       <select id="u${ui}_residuos">
@@ -225,6 +223,14 @@
         <option value="No">No</option>
       </select>
     </div>
     <div class="field">
       <label>¿Cuenta con acople de tubo?</label>
       <select id="u${ui}_acople">
         <option value="">Seleccionar</option>
         <option value="Sí">Sí</option>
         <option value="No">No</option>
       </select>
     </div>
   </div>

   <div class="field" style="margin-top:14px">
@@ -281,7 +287,8 @@
       estados: items.map((it,ii)=>document.getElementById(`u${ui}i${ii}`).value),
       verificaciones: {
         residuos: document.getElementById(`u${ui}_residuos`).value,
         llave: document.getElementById(`u${ui}_llave`).value
         llave: document.getElementById(`u${ui}_llave`).value,
         acople: document.getElementById(`u${ui}_acople`).value
       },
       fotos: fotosUrls,
       obs: document.getElementById(`uo${ui}`).value.trim()
@@ -368,32 +375,28 @@
       if(u.verificaciones){
         if(u.verificaciones.residuos) verificacionesTxt.push(`Residuos patogénicos: ${u.verificaciones.residuos}`);
         if(u.verificaciones.llave) verificacionesTxt.push(`Llave fija y llave de tubo: ${u.verificaciones.llave}`);
       } else if(u.faltantes){
         if(u.faltantes.med) verificacionesTxt.push(`Medicamentos: ${u.faltantes.med}`);
         if(u.faltantes.ins) verificacionesTxt.push(`Insumos: ${u.faltantes.ins}`);
         if(u.faltantes.acople) verificacionesTxt.push(`Acople: ${u.faltantes.acople}`);
         if(u.faltantes.llave) verificacionesTxt.push(`Llave: ${u.faltantes.llave}`);
         if(u.verificaciones.acople) verificacionesTxt.push(`Acople de tubo: ${u.verificaciones.acople}`);
       }

       const estadosList = (u.estados||[]).map((est, idx) => {
         if(!est) return null;
         return `<div style="font-size:12px"><strong>${items[idx] || "Control"}:</strong> ${est}</div>`;
       }).filter(Boolean);

       const esOp = u.operativo !== "No Operativo";
       const badgeOp = esOp 
         ? `<span style="padding:3px 8px;border-radius:6px;background:#eaf8f1;color:var(--good);font-weight:700;font-size:11px">OPERATIVO</span>`
         : `<span style="padding:3px 8px;border-radius:6px;background:#fdeaea;color:var(--bad);font-weight:700;font-size:11px">NO OPERATIVO</span>`;

       const fotosHtml = (u.fotos && u.fotos.length) 
         ? `<div style="margin-top:8px">
              <strong style="font-size:12px;color:var(--muted)">FOTOS ADJUNTAS:</strong>
              <div class="photo-preview-container">
                ${u.fotos.map(f => `<a href="${f}" target="_blank"><img src="${f}" class="photo-thumb" alt="Foto unidad"></a>`).join("")}
              </div>
            </div>`
         : '';

       return `
       <div style="margin-top:8px;padding:10px;background:#f8fafc;border-radius:8px;border:1px solid #e2e8f0;font-size:13px">
         <div style="display:flex;justify-content:space-between;align-items:center">
         
