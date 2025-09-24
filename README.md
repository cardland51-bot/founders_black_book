<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Rental Management — Luxe Revamp</title>

<!-- Fonts & Icons -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
<link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css" rel="stylesheet">

<!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
:root{
  --bg-1:#071226; --bg-2:#081827;
  --glass: rgba(255,255,255,0.04);
  --glass-2: rgba(255,255,255,0.02);
  --accent1: #7b61ff; --accent2: #3ad0ff;
  --muted: rgba(255,255,255,0.68);
  --radius:14px;
}
*{box-sizing:border-box}
html,body{height:100%; margin:0; font-family:"Inter",system-ui,Segoe UI,Roboto; background:linear-gradient(180deg,var(--bg-1),var(--bg-2)); color:#e8f0ff;}
.container-app{display:grid; grid-template-columns:240px 1fr; gap:20px; padding:28px; min-height:100vh;}

/* Sidebar */
.sidebar{
  position:sticky; top:28px; height:calc(100vh - 56px);
  background:linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));
  border-radius:var(--radius); padding:18px; border:1px solid rgba(255,255,255,0.03);
  box-shadow: 0 12px 40px rgba(2,6,23,0.6); backdrop-filter: blur(6px);
}
.brand{display:flex;gap:12px;align-items:center;margin-bottom:12px}
.brand .logo{width:46px;height:46px;border-radius:10px;background:linear-gradient(90deg,var(--accent1),var(--accent2));display:flex;align-items:center;justify-content:center;font-weight:800;color:white}
.brand h3{margin:0;font-size:16px}
.small-muted{color:var(--muted);font-size:13px}

/* nav */
.nav-list{margin-top:12px}
.nav-item{display:flex;align-items:center;gap:10px;padding:10px;border-radius:10px;color:var(--muted);cursor:pointer;margin-bottom:6px}
.nav-item.active{background:linear-gradient(90deg, rgba(123,97,255,0.12), rgba(58,208,255,0.03)); color:white; box-shadow: inset 0 0 20px rgba(0,0,0,0.2)}
.nav-item:hover{color:#fff}

/* Main */
.main{padding:6px 0}
.topbar{display:flex;justify-content:space-between;align-items:center;margin-bottom:16px}
.search{display:flex;align-items:center;gap:10px;background:var(--glass);padding:8px 12px;border-radius:10px;border:1px solid rgba(255,255,255,0.02)}
.search input{background:transparent;border:0;color:var(--muted);outline:none}
.actions .btn{margin-left:8px}

/* Grid layout */
.grid{display:grid;grid-template-columns:repeat(12,1fr);gap:14px}

/* KPI */
.kpi{grid-column:span 3;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:16px;border-radius:12px;border:1px solid rgba(255,255,255,0.03);box-shadow:0 10px 30px rgba(2,6,23,0.45)}
.kpi h6{margin:0;color:#dff3ff}
.kpi .val{font-weight:800;font-size:20px;margin-top:8px}
.kpi .sub{color:var(--muted);font-size:13px;margin-top:6px}

/* panels */
.panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border-radius:12px;padding:18px;border:1px solid rgba(255,255,255,0.03);box-shadow:0 12px 40px rgba(2,6,23,0.45)}
.panel h5{margin:0;color:#eaf6ff}

/* tables */
.table thead th{border-bottom:1px solid rgba(255,255,255,0.03);color:var(--muted)}
.table tbody tr td{color:#e9f4ff}
.link-tenant{color:#9ae6ff;cursor:pointer;text-decoration:none}
.badge-soft{background:rgba(255,255,255,0.03);padding:6px 10px;border-radius:10px;color:#e8f6ff}

/* modal luxe */
.modal-backdrop.show{backdrop-filter: blur(6px); background: rgba(3,6,23,0.6)}
.modal-content{background:linear-gradient(180deg, rgba(6,12,24,0.95), rgba(6,10,20,0.95)); color:#eaf4ff; border-radius:12px; border: none}

/* small */
.controls{display:flex;gap:8px;align-items:center}
.btn-soft{background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.03);color:#eaf6ff}
@media (max-width:1000px){.container-app{grid-template-columns:1fr;padding:14px}.sidebar{position:relative;height:auto}}
</style>
</head>
<body>

<div class="container-app">
  <!-- SIDEBAR -->
  <aside class="sidebar">
    <div class="brand">
      <div class="logo">RM</div>
      <div>
        <h3>Cardland Rentals</h3>
        <div class="small-muted">Property Management</div>
      </div>
    </div>

    <div class="nav-list">
      <div class="nav-item active" data-section="overview"><i class="bi bi-speedometer2"></i> Overview</div>
      <div class="nav-item" data-section="properties"><i class="bi bi-building"></i> Properties</div>
      <div class="nav-item" data-section="tenants"><i class="bi bi-people"></i> Tenants</div>
      <div class="nav-item" id="openMaintenance"><i class="bi bi-wrench"></i> Maintenance</div>
      <div class="nav-item" data-section="payments"><i class="bi bi-currency-dollar"></i> Payments</div>
    </div>

    <hr style="border-color:rgba(255,255,255,0.03)">

    <div class="small-muted">Quick Actions</div>
    <div style="margin-top:10px; display:flex; flex-direction:column; gap:8px;">
      <button class="btn btn-soft btn-sm" id="btnAddProperty"><i class="bi bi-plus-lg"></i> Add Property</button>
      <button class="btn btn-soft btn-sm" id="btnExportAll"><i class="bi bi-download"></i> Export CSV</button>
    </div>
  </aside>

  <!-- MAIN -->
  <main class="main">
    <div class="topbar">
      <div style="display:flex;align-items:center;gap:12px">
        <div class="search"><i class="bi bi-search" style="opacity:.7"></i><input id="globalSearch" placeholder="Search tenants, properties, IDs..." /></div>
        <div class="small-muted">Welcome back, <strong>Admin</strong></div>
      </div>
      <div class="actions">
        <button class="btn btn-soft btn-sm"><i class="bi bi-bell"></i></button>
        <button class="btn btn-soft btn-sm"><i class="bi bi-gear"></i></button>
      </div>
    </div>

    <div class="grid">
      <!-- KPIs -->
      <div class="kpi">
        <h6>Total Properties</h6>
        <div class="val" id="kTotal">0</div>
        <div class="sub">Active listings in portfolio</div>
      </div>

      <div class="kpi">
        <h6>Occupied</h6>
        <div class="val" id="kOccupied">0</div>
        <div class="sub">Currently leased</div>
      </div>

      <div class="kpi">
        <h6>Vacant</h6>
        <div class="val" id="kVacant">0</div>
        <div class="sub">Available units</div>
      </div>

      <div class="kpi">
        <h6>Late Payments</h6>
        <div class="val" id="kLate">0</div>
        <div class="sub">Overdue tenants</div>
      </div>

      <!-- Rent chart -->
      <div class="panel" style="grid-column: span 7">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
          <h5>Monthly Rent Collection</h5>
          <div class="small-muted">Last 12 months</div>
        </div>
        <div style="height:320px"><canvas id="rentChart"></canvas></div>
      </div>

      <!-- Maintenance donut -->
      <div class="panel" style="grid-column: span 5; display:flex; flex-direction:column; justify-content:space-between;">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <h5>Maintenance Status</h5>
          <div class="badge-soft" id="maintSummary">—</div>
        </div>
        <div style="display:flex;align-items:center;justify-content:center; height:240px">
          <canvas id="maintChart" style="width:220px;height:220px"></canvas>
        </div>
        <div class="small-muted">Manage requests from the Maintenance tab</div>
      </div>

      <!-- Properties Table -->
      <div class="panel" style="grid-column: span 7">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px">
          <h5>Properties</h5>
          <div style="display:flex;gap:8px;align-items:center">
            <input id="searchProperties" class="form-control form-control-sm" placeholder="Search properties..." style="width:220px">
            <button class="btn btn-soft btn-sm" onclick="exportCSV('properties')"><i class="bi bi-download"></i></button>
          </div>
        </div>
        <div style="max-height:420px; overflow:auto">
          <table class="table table-borderless">
            <thead><tr class="small-muted"><th>Address</th><th>Type</th><th>Beds</th><th>Baths</th><th>Status</th><th>Tenant</th><th>Rent</th><th></th></tr></thead>
            <tbody id="tblProperties"></tbody>
          </table>
        </div>
      </div>

      <!-- Tenants Table -->
      <div class="panel" style="grid-column: span 5">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px">
          <h5>Tenants</h5>
          <div style="display:flex;gap:8px;align-items:center">
            <input id="searchTenants" class="form-control form-control-sm" placeholder="Search tenants..." style="width:180px">
            <button class="btn btn-soft btn-sm" onclick="exportCSV('tenants')"><i class="bi bi-download"></i></button>
          </div>
        </div>
        <div style="max-height:420px; overflow:auto">
          <table class="table table-borderless">
            <thead><tr class="small-muted"><th>Name</th><th>Property</th><th>Lease End</th><th>Rent</th><th>Status</th></tr></thead>
            <tbody id="tblTenants"></tbody>
          </table>
        </div>
      </div>

    </div>
  </main>
</div>

<!-- Maintenance Modal -->
<div class="modal fade" id="maintenanceModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-fullscreen">
    <div class="modal-content">
      <div class="modal-header" style="border-bottom:1px solid rgba(255,255,255,0.03)">
        <h5 class="modal-title">Maintenance Requests</h5>
        <div class="small-muted ms-2" id="maintCount">0 requests</div>
        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px">
          <input id="searchMaintenance" class="form-control" placeholder="Search issues, tenants, addresses..." style="width:480px">
          <div style="display:flex;gap:8px">
            <button class="btn btn-soft btn-sm" onclick="filterMaintenance('All')">All</button>
            <button class="btn btn-sm btn-warning" onclick="filterMaintenance('Pending')">Pending</button>
            <button class="btn btn-sm btn-info" onclick="filterMaintenance('In Progress')">In Progress</button>
            <button class="btn btn-sm btn-success" onclick="filterMaintenance('Completed')">Completed</button>
          </div>
        </div>

        <div style="max-height:76vh;overflow:auto">
          <table class="table table-striped">
            <thead class="table-dark"><tr><th>ID</th><th>Property</th><th>Tenant</th><th>Description</th><th>Status</th><th>Date</th><th>Actions</th></tr></thead>
            <tbody id="tblMaintenance"></tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- Tenant Modal -->
<div class="modal fade" id="tenantModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-lg modal-dialog-centered">
    <div class="modal-content p-3">
      <div class="modal-header border-0">
        <h5 class="modal-title">Tenant Details</h5>
        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body" id="tenantModalBody"></div>
    </div>
  </div>
</div>

<!-- Scripts -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
<script>
/* ---------- local persistence helpers ---------- */
const STORAGE_KEY = 'cardland_dashboard_v1';
function saveState(state){ try{ localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); }catch(e){} }
function loadState(){ try{ return JSON.parse(localStorage.getItem(STORAGE_KEY)); }catch(e){ return null } }

/* ---------- seed demo data (used on first load) ---------- */
const demo = {
  properties:[
    {address:'123 Main St', type:'Apartment', bedrooms:2, bathrooms:1, status:'Occupied', tenant:'John Doe', rent:2200, pendingApplications:[]},
    {address:'456 Oak Ave', type:'House', bedrooms:3, bathrooms:2, status:'Vacant', tenant:'', rent:0, pendingApplications:[{name:'Alice Johnson', leaseStart:'2025-10-01', leaseEnd:'2026-09-30', monthlyRent:2400}]},
    {address:'789 Pine Ln', type:'Condo', bedrooms:2, bathrooms:2, status:'Occupied', tenant:'Jane Smith', rent:2500, pendingApplications:[]},
  ],
  tenants:[
    {name:'John Doe', property:'123 Main St', leaseStart:'2025-01-01', leaseEnd:'2026-01-01', monthlyRent:2200, status:'Active'},
    {name:'Jane Smith', property:'789 Pine Ln', leaseStart:'2024-09-01', leaseEnd:'2025-09-01', monthlyRent:2500, status:'Active'}
  ],
  maintenance:[
    {id:101, property:'123 Main St', tenant:'John Doe', description:'Leaky faucet under kitchen sink', status:'Pending', date:'2025-09-20'},
    {id:102, property:'789 Pine Ln', tenant:'Jane Smith', description:'AC blowing warm on second floor', status:'In Progress', date:'2025-09-18'}
  ]
};

/* ---------- application state ---------- */
let state = loadState() || demo;

/* save on unload */
window.addEventListener('beforeunload', ()=> saveState(state));

/* ---------- helper to reinitialize after modifications ---------- */
function refreshAll(){
  renderKPIs();
  renderTables();
  refreshCharts();
  saveState(state);
}

/* ---------- KPI & tables rendering ---------- */
function renderKPIs(){
  const total = state.properties.length;
  const occupied = state.properties.filter(p=>p.status==='Occupied').length;
  const vacant = total - occupied;
  const late = state.tenants.filter(t=>t.status==='Late').length;
  document.getElementById('kTotal').textContent = total;
  document.getElementById('kOccupied').textContent = occupied;
  document.getElementById('kVacant').textContent = vacant;
  document.getElementById('kLate').textContent = late;
}

function renderTables(){
  // Properties
  const pbody = document.getElementById('tblProperties'); pbody.innerHTML = '';
  state.properties.forEach(p=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${p.address}</td>
      <td>${p.type}</td>
      <td>${p.bedrooms}</td>
      <td>${p.bathrooms}</td>
      <td><span class="badge-soft">${p.status}</span></td>
      <td>${p.tenant ? `<a class="link-tenant" onclick="openTenant('${p.tenant}')">${p.tenant}</a>` : '-'}</td>
      <td>$${p.rent.toLocaleString()}</td>
      <td>${p.pendingApplications && p.pendingApplications.length ? `<button class="btn btn-sm btn-warning" onclick="openPending('${p.address}')">${p.pendingApplications.length} Pending</button>` : '-'}</td>
    `;
    pbody.appendChild(tr);
  });

  // Tenants
  const tbody = document.getElementById('tblTenants'); tbody.innerHTML = '';
  state.tenants.forEach(t=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><a class="link-tenant" onclick="openTenant('${t.name}')">${t.name}</a></td>
      <td>${t.property}</td>
      <td>${t.leaseEnd}</td>
      <td>$${t.monthlyRent.toLocaleString()}</td>
      <td><span class="badge-soft">${t.status}</span></td>
    `;
    tbody.appendChild(tr);
  });

  // Maintenance table for modal
  const mbody = document.getElementById('tblMaintenance'); if(mbody) mbody.innerHTML = '';
  state.maintenance.forEach((m, idx)=>{
    if(!mbody) return;
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${m.id}</td>
      <td>${m.property}</td>
      <td>${m.tenant}</td>
      <td style="max-width:420px">${m.description}</td>
      <td>${m.status}</td>
      <td>${m.date}</td>
      <td>
        ${m.status !== 'In Progress' ? `<button class="btn btn-sm btn-warning me-1" onclick="updateMaintenance(${idx},'In Progress')">In Progress</button>` : ''}
        ${m.status !== 'Completed' ? `<button class="btn btn-sm btn-success" onclick="updateMaintenance(${idx},'Completed')">Completed</button>` : ''}
      </td>
    `;
    mbody.appendChild(tr);
  });

  document.getElementById('maintCount').textContent = `${state.maintenance.length} requests`;
  document.getElementById('maintSummary').textContent = `${state.maintenance.filter(r=>r.status==='Pending').length} Pending • ${state.maintenance.filter(r=>r.status==='In Progress').length} In Progress • ${state.maintenance.filter(r=>r.status==='Completed').length} Done`;
}

/* ---------- Charts ---------- */
const rentCanvas = document.getElementById('rentChart').getContext('2d');
const gradient = rentCanvas.createLinearGradient(0,0,0,300);
gradient.addColorStop(0, 'rgba(123,97,255,0.28)');
gradient.addColorStop(1, 'rgba(58,208,255,0.03)');
const rentChart = new Chart(rentCanvas, {
  type:'line',
  data:{ labels:['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'], datasets:[{label:'Rent', data:Array(12).fill(0), backgroundColor: gradient, borderColor:'rgba(123,97,255,0.98)', tension:0.36, fill:true, pointRadius:3}]},
  options:{ responsive:true, plugins:{legend:{display:false}}, scales:{ x:{grid:{display:false}, ticks:{color:'rgba(255,255,255,0.6)'} }, y:{grid:{color:'rgba(255,255,255,0.03)'}, ticks:{color:'rgba(255,255,255,0.6)'}}}}
});

const maintCanvas = document.getElementById('maintChart').getContext('2d');
const maintChart = new Chart(maintCanvas, {
  type:'doughnut',
  data:{ labels:['Pending','In Progress','Completed'], datasets:[{data:[0,0,0], backgroundColor:['#f39c12','#3498db','#2ecc71'], hoverOffset:6}] },
  options:{ responsive:true, plugins:{legend:{position:'bottom', labels:{color:'rgba(255,255,255,0.9)'}}} }
});

function refreshCharts(){
  // simple illustrative logic: current month sum of tenant monthly rent
  const tenants = state.tenants;
  const monthIndex = new Date().getMonth();
  const monthly = Array(12).fill(0);
  monthly[monthIndex] = tenants.reduce((s,t) => s + (t.status !== 'Late' ? t.monthlyRent : 0), 0);
  rentChart.data.datasets[0].data = monthly;
  rentChart.update();

  maintChart.data.datasets[0].data = [
    state.maintenance.filter(m=>m.status==='Pending').length,
    state.maintenance.filter(m=>m.status==='In Progress').length,
    state.maintenance.filter(m=>m.status==='Completed').length
  ];
  maintChart.update();
}

/* ---------- actions ---------- */
function updateMaintenance(idx, status){
  state.maintenance[idx].status = status;
  refreshAll();
}

function openMaintenanceModal(){
  const modal = new bootstrap.Modal(document.getElementById('maintenanceModal'));
  modal.show();
  renderTables();
}

document.getElementById('openMaintenance').addEventListener('click', openMaintenanceModal);

function openTenant(name){
  const t = state.tenants.find(x => x.name === name);
  if(!t) return;
  const body = document.getElementById('tenantModalBody');
  body.innerHTML = `
    <div style="display:flex;gap:18px">
      <div style="flex:1">
        <h5 style="margin:0">${t.name}</h5>
        <div class="small-muted" style="margin-bottom:8px">Contact</div>
        <div>Phone: (555) 123-4567<br>Email: ${t.name.toLowerCase().replace(' ','')}@example.com</div>
        <div class="small-muted" style="margin-top:12px">Lease</div>
        <div>${t.leaseStart} → <strong>${t.leaseEnd}</strong></div>
      </div>
      <div style="flex:1">
        <h6 style="margin-top:0">Financial</h6>
        <div>Monthly Rent: <strong>$${t.monthlyRent.toLocaleString()}</strong></div>
        <div style="margin-top:8px">Status: <span class="badge-soft">${t.status}</span></div>
        <h6 style="margin-top:14px">Maintenance</h6>
        <div>${state.maintenance.filter(m=>m.tenant===t.name).map(m=>`<div style="margin-bottom:8px"><strong>${m.description}</strong><div class="small-muted">${m.date} • ${m.status}</div></div>`).join('') || '<div class="small-muted">No requests</div>'}</div>
      </div>
    </div>
  `;
  new bootstrap.Modal(document.getElementById('tenantModal')).show();
}

/* pending apps modal via tenantModal for simplicity */
function openPending(address){
  const property = state.properties.find(p=>p.address===address);
  if(!property) return;
  const body = document.getElementById('tenantModalBody');
  body.innerHTML = `<h5>Pending Applications — ${address}</h5>`;
  if(!property.pendingApplications.length) body.innerHTML += '<p class="small-muted">No pending applications</p>';
  else {
    body.innerHTML += `<div class="list-group">` + property.pendingApplications.map((a,i)=>`
      <div class="list-group-item d-flex justify-content-between align-items-center">
        <div><strong>${a.name}</strong><div class="small-muted">Lease: ${a.leaseStart} → ${a.leaseEnd}</div></div>
        <div><button class="btn btn-sm btn-success me-1" onclick="approvePending('${address}',${i})">Approve</button><button class="btn btn-sm btn-danger" onclick="declinePending('${address}',${i})">Decline</button></div>
      </div>`).join('') + `</div>`;
  }
  new bootstrap.Modal(document.getElementById('tenantModal')).show();
}

function approvePending(address, idx){
  const prop = state.properties.find(p=>p.address===address);
  const app = prop.pendingApplications.splice(idx,1)[0];
  prop.status='Occupied'; prop.tenant = app.name; prop.rent = app.monthlyRent;
  state.tenants.push({name:app.name, property:address, leaseStart:app.leaseStart, leaseEnd:app.leaseEnd, monthlyRent:app.monthlyRent, status:'Active'});
  refreshAll();
  openPending(address);
}
function declinePending(address, idx){
  const prop = state.properties.find(p=>p.address===address);
  prop.pendingApplications.splice(idx,1);
  refreshAll();
  openPending(address);
}

/* ---------- searching & export ---------- */
document.getElementById('searchProperties').addEventListener('input', e=>{
  const q = e.target.value.toLowerCase();
  document.querySelectorAll('#tblProperties tr').forEach(tr=>{
    tr.style.display = Array.from(tr.cells).some(td=>td.textContent.toLowerCase().includes(q)) ? '' : 'none';
  });
});
document.getElementById('searchTenants').addEventListener('input', e=>{
  const q = e.target.value.toLowerCase();
  document.querySelectorAll('#tblTenants tr').forEach(tr=>{
    tr.style.display = Array.from(tr.cells).some(td=>td.textContent.toLowerCase().includes(q)) ? '' : 'none';
  });
});
document.getElementById('globalSearch').addEventListener('keydown', e=>{
  if(e.key === 'Enter'){
    const q = e.target.value.toLowerCase();
    const t = state.tenants.find(x=>x.name.toLowerCase().includes(q));
    if(t){ openTenant(t.name); return; }
    const p = state.properties.find(x=>x.address.toLowerCase().includes(q));
    if(p){ openPending(p.address); return; }
    alert('No quick match found.');
  }
});
function exportCSV(type){
  let rows = [];
  if(type === 'properties'){
    rows = [['Address','Type','Beds','Baths','Status','Tenant','Rent']];
    state.properties.forEach(p=> rows.push([p.address,p.type,p.bedrooms,p.bathrooms,p.status,p.tenant || '-', p.rent]));
  } else if(type === 'tenants'){
    rows = [['Name','Property','LeaseStart','LeaseEnd','MonthlyRent','Status']];
    state.tenants.forEach(t=> rows.push([t.name,t.property,t.leaseStart,t.leaseEnd,t.monthlyRent,t.status]));
  } else {
    // export both
    exportCSV('properties'); exportCSV('tenants'); return;
  }
  const csv = rows.map(r=>r.map(c => `"${String(c).replace(/"/g,'""')}"`).join(',')).join('\n');
  const blob = new Blob([csv], {type:'text/csv'});
  const link = document.createElement('a'); link.href = URL.createObjectURL(blob); link.download = `${type}.csv`; link.click();
}

/* filter maintenance in modal */
function filterMaintenance(status){
  document.querySelectorAll('#tblMaintenance tr').forEach(tr=>{
    if(status === 'All') tr.style.display = '';
    else tr.style.display = (tr.cells[4].textContent === status) ? '' : 'none';
  });
}

/* ---------- attach top buttons ---------- */
document.getElementById('btnExportAll').addEventListener('click', ()=> exportCSV('both'));
document.getElementById('btnAddProperty').addEventListener('click', ()=>{
  const addr = prompt('Address for new property (e.g. 555 Park Ave)');
  if(!addr) return;
  state.properties.push({address:addr,type:'Apartment',bedrooms:1,bathrooms:1,status:'Vacant',tenant:'',rent:0,pendingApplications:[]});
  refreshAll();
});

/* ---------- init ---------- */
function init(){
  renderKPIs(); renderTables(); refreshCharts();
}
init();

</script>
</body>
</html>
