<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Rental Management — Luxe Dashboard</title>

<!-- Fonts + Icons -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
<link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css" rel="stylesheet">

<!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
:root{
  --bg-1: #0f1724;       /* deep navy */
  --bg-2: #0b1320;       /* darker */
  --glass: rgba(255,255,255,0.06);
  --glass-2: rgba(255,255,255,0.04);
  --accent: linear-gradient(90deg,#7b61ff,#3ad0ff);
  --muted: rgba(255,255,255,0.65);
  --card-radius: 14px;
  --shadow-1: 0 8px 30px rgba(2,6,23,0.6);
}

*{box-sizing:border-box}
html,body{height:100%}
body{
  margin:0;
  min-height:100%;
  background: radial-gradient(1200px 400px at 10% 10%, rgba(59,45,126,0.18), transparent 10%),
              radial-gradient(1000px 300px at 90% 90%, rgba(58,208,255,0.06), transparent 10%),
              linear-gradient(180deg,var(--bg-1),var(--bg-2));
  color: #e6eef8;
  font-family: "Inter", system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  -webkit-font-smoothing:antialiased;
  -moz-osx-font-smoothing:grayscale;
  padding: 24px;
}

/* Layout */
.app {
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 20px;
  align-items:start;
}

.sidebar {
  position:sticky;
  top:24px;
  height: calc(100vh - 48px);
  background: linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));
  border-radius: var(--card-radius);
  padding:18px;
  box-shadow: var(--shadow-1);
  backdrop-filter: blur(6px);
  border: 1px solid rgba(255,255,255,0.04);
}

.brand {
  display:flex; gap:12px; align-items:center; margin-bottom:16px;
}
.brand .mark{
  width:44px;height:44px;border-radius:10px;
  background:var(--accent); box-shadow:0 6px 20px rgba(123,97,255,0.18);
  display:flex; align-items:center; justify-content:center; font-weight:800;
  color:white;
}
.brand h3{font-size:16px;margin:0;color:#fff;letter-spacing:0.2px}

/* Sidebar items */
.nav-item{
  display:flex;align-items:center;gap:12px;padding:10px;border-radius:10px;margin-bottom:6px;color:var(--muted);cursor:pointer;
}
.nav-item:hover{background:rgba(255,255,255,0.02);color:#fff}
.nav-item.active{background:linear-gradient(90deg, rgba(123,97,255,0.12), rgba(58,208,255,0.03)); color:#fff; box-shadow: inset 0 0 20px rgba(0,0,0,0.25)}

/* Main content area */
.content {
  position:relative;
}

/* Topbar */
.topbar {
  display:flex; justify-content:space-between; align-items:center; margin-bottom:18px;
}
.topbar .search {
  background: var(--glass);
  border-radius: 10px;
  padding:8px 12px;
  display:flex; align-items:center; gap:8px; border:1px solid rgba(255,255,255,0.03);
}
.topbar input{ background:transparent; border:0; outline:none; color:var(--muted); min-width:220px }
.topbar .actions button{ margin-left:10px }

/* Dashboard grid */
.grid {
  display:grid;
  grid-template-columns: repeat(12, 1fr);
  gap:16px;
}

/* KPI card */
.kpi {
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  border-radius:12px; padding:16px; box-shadow:0 6px 30px rgba(2,6,23,0.45);
  border: 1px solid rgba(255,255,255,0.035);
}
.kpi h6{ margin:0; font-weight:600; color:rgba(255,255,255,0.85) }
.kpi .value { font-size:22px; font-weight:800; margin-top:6px }
.kpi .sub { color:var(--muted); font-size:12px; margin-top:8px }

/* Panels */
.panel{
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  border-radius:var(--card-radius);
  padding:18px;
  box-shadow:0 10px 40px rgba(2,6,23,0.45);
  border: 1px solid rgba(255,255,255,0.04);
}

/* table styling */
.table thead th{ background:transparent; border-bottom: 1px solid rgba(255,255,255,0.04); color:var(--muted) }
.table tbody tr td{ color:#e9f0fb }
.small-muted{ font-size:12px; color:var(--muted) }

/* Modal overlays — extra luxe */
.modal-backdrop.show {
  background-color: rgba(3,6,23,0.6);
  backdrop-filter: blur(6px);
}
.modal-content{
  background: linear-gradient(180deg, rgba(10,14,30,0.9), rgba(6,10,20,0.85));
  border: none;
  color:#dfeeff;
  border-radius:14px;
}

/* Buttons */
.btn-soft{
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.04);
  color: #eaf5ff;
}

/* small utilities */
.badge-soft {
  background: rgba(255,255,255,0.04);
  color: #dff4ff;
  border-radius:10px;
  padding:6px 10px;
  font-weight:600;
}

/* responsive */
@media (max-width: 1000px){
  .app { grid-template-columns: 1fr; }
  .sidebar { position:relative; height:auto; }
  .main { margin-left:0 }
}
</style>
</head>
<body>

<div class="app container-fluid px-0">
  <!-- Sidebar -->
  <aside class="sidebar">
    <div class="brand">
      <div class="mark">RM</div>
      <div>
        <h3>Cardland Rentals</h3>
        <div class="small-muted">Management Dashboard</div>
      </div>
    </div>

    <div class="mb-3">
      <div class="nav-item active" data-section="sectionOverview"><i class="bi bi-speedometer2"></i> Overview</div>
      <div class="nav-item" data-section="sectionProperties"><i class="bi bi-building"></i> Properties</div>
      <div class="nav-item" data-section="sectionTenants"><i class="bi bi-people"></i> Tenants</div>
      <div class="nav-item" id="maintenanceTrigger"><i class="bi bi-wrench"></i> Maintenance</div>
      <div class="nav-item" data-section="sectionPayments"><i class="bi bi-currency-dollar"></i> Payments</div>
    </div>

    <hr style="border-color: rgba(255,255,255,0.03)">
    <div class="small-muted">Quick actions</div>
    <div class="mt-2 d-grid gap-2">
      <button class="btn btn-soft btn-sm"><i class="bi bi-plus-lg"></i> New Property</button>
      <button class="btn btn-soft btn-sm"><i class="bi bi-file-earmark-arrow-down"></i> Export Data</button>
    </div>
  </aside>

  <!-- Main content -->
  <main class="content">
    <div class="topbar">
      <div style="display:flex;align-items:center;gap:12px">
        <div class="search">
          <i class="bi bi-search" style="opacity:.6"></i>
          <input id="globalSearch" placeholder="Search tenants, properties..." />
        </div>
        <div class="small-muted">Welcome back — <strong>Admin</strong></div>
      </div>
      <div class="actions">
        <button class="btn btn-sm btn-soft"><i class="bi bi-bell"></i></button>
        <button class="btn btn-sm btn-soft"><i class="bi bi-gear"></i></button>
      </div>
    </div>

    <!-- Grid -->
    <div class="grid">
      <!-- KPIs row -->
      <div class="kpi panel" style="grid-column: span 3">
        <h6>Total Properties</h6>
        <div class="value" id="kpiTotal">0</div>
        <div class="sub">Active listings across portfolio</div>
      </div>

      <div class="kpi panel" style="grid-column: span 3">
        <h6>Occupied</h6>
        <div class="value" id="kpiOccupied">0</div>
        <div class="sub">Properties currently leased</div>
      </div>

      <div class="kpi panel" style="grid-column: span 3">
        <h6>Vacant</h6>
        <div class="value" id="kpiVacant">0</div>
        <div class="sub">Available properties</div>
      </div>

      <div class="kpi panel" style="grid-column: span 3">
        <h6>Late Payments</h6>
        <div class="value" id="kpiLate">0</div>
        <div class="sub">Tenants with overdue balances</div>
      </div>

      <!-- Charts -->
      <div class="panel" style="grid-column: span 7">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
          <h5 style="margin:0">Monthly Rent Collection</h5>
          <div class="small-muted">Last 12 months</div>
        </div>
        <div style="height:300px">
          <canvas id="chartRent" style="width:100%;height:100%"></canvas>
        </div>
      </div>

      <div class="panel" style="grid-column: span 5">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
          <h5 style="margin:0">Maintenance Status</h5>
          <div class="badge-soft">Live</div>
        </div>
        <div style="height:300px;display:flex;align-items:center;justify-content:center">
          <canvas id="chartMaintenance" style="width:240px;height:240px"></canvas>
        </div>
      </div>

      <!-- Tables -->
      <div class="panel" style="grid-column: span 7">
        <div class="d-flex justify-content-between align-items-center mb-2">
          <h5 style="margin:0">Properties</h5>
          <div>
            <input id="searchProperties" class="form-control form-control-sm" placeholder="Search properties..." style="width:240px;display:inline-block" />
          </div>
        </div>
        <div style="max-height:420px;overflow:auto">
          <table class="table table-borderless table-hover">
            <thead>
              <tr class="small-muted">
                <th>Address</th><th>Type</th><th>Beds</th><th>Baths</th><th>Status</th><th>Tenant</th><th>Rent</th><th></th>
              </tr>
            </thead>
            <tbody id="tblProperties"></tbody>
          </table>
        </div>
      </div>

      <div class="panel" style="grid-column: span 5">
        <div class="d-flex justify-content-between align-items-center mb-2">
          <h5 style="margin:0">Tenants</h5>
          <div>
            <input id="searchTenants" class="form-control form-control-sm" placeholder="Search tenants..." style="width:200px;display:inline-block" />
          </div>
        </div>
        <div style="max-height:420px;overflow:auto">
          <table class="table table-borderless table-hover">
            <thead>
              <tr class="small-muted">
                <th>Name</th><th>Property</th><th>Lease Ends</th><th>Rent</th><th>Status</th>
              </tr>
            </thead>
            <tbody id="tblTenants"></tbody>
          </table>
        </div>
      </div>

    </div> <!-- grid end -->

  </main>
</div>

<!-- Maintenance Modal (full-screen style) -->
<div class="modal fade" id="maintenanceModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-fullscreen">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Maintenance Requests</h5>
        <div class="small-muted ms-2" id="maintenanceCount"></div>
        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px">
          <div>
            <input id="searchMaintenance" class="form-control" placeholder="Search issues, tenants, addresses..." style="width:420px" />
          </div>
          <div>
            <button class="btn btn-sm btn-soft" onclick="filterMaintenance('All')">All</button>
            <button class="btn btn-sm btn-warning" onclick="filterMaintenance('Pending')">Pending</button>
            <button class="btn btn-sm btn-info" onclick="filterMaintenance('In Progress')">In Progress</button>
            <button class="btn btn-sm btn-success" onclick="filterMaintenance('Completed')">Completed</button>
          </div>
        </div>

        <div style="max-height:75vh;overflow:auto">
          <table class="table table-striped">
            <thead class="table-dark">
              <tr><th>ID</th><th>Property</th><th>Tenant</th><th>Description</th><th>Status</th><th>Date</th><th>Actions</th></tr>
            </thead>
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
        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body" id="tenantModalBody"></div>
    </div>
  </div>
</div>

<script>
/* ---------------- sample data (replace with real) ---------------- */
let properties = [
  {address:'123 Main St', type:'Apartment', bedrooms:2, bathrooms:1, status:'Occupied', tenant:'John Doe', rent:2200, pendingApplications:[]},
  {address:'456 Oak Ave', type:'House', bedrooms:3, bathrooms:2, status:'Vacant', tenant:'', rent:0, pendingApplications:[]},
  {address:'789 Pine Ln', type:'Condo', bedrooms:2, bathrooms:2, status:'Occupied', tenant:'Jane Smith', rent:2500, pendingApplications:[]},
];

let tenants = [
  {name:'John Doe', property:'123 Main St', leaseStart:'2025-01-01', leaseEnd:'2026-01-01', monthlyRent:2200, status:'Active'},
  {name:'Jane Smith', property:'789 Pine Ln', leaseStart:'2024-09-01', leaseEnd:'2025-09-01', monthlyRent:2500, status:'Active'}
];

let maintenanceRequests = [
  {id:101, property:'123 Main St', tenant:'John Doe', description:'Leaky kitchen faucet - slow drip from pipe under sink', status:'Pending', date:'2025-09-20'},
  {id:102, property:'789 Pine Ln', tenant:'Jane Smith', description:'AC not cooling on 2nd floor', status:'In Progress', date:'2025-09-18'},
];

/* ---------------- DOM helpers ---------------- */
const $ = (sel) => document.querySelector(sel);
const $$ = (sel) => Array.from(document.querySelectorAll(sel));

/* ---------------- Chart setup ---------------- */
const rentCtx = document.getElementById('chartRent').getContext('2d');
const gradientFill = rentCtx.createLinearGradient(0,0,0,300);
gradientFill.addColorStop(0, 'rgba(123,97,255,0.28)');
gradientFill.addColorStop(1, 'rgba(58,208,255,0.03)');

const chartRent = new Chart(rentCtx, {
  type: 'line',
  data: {
    labels: ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'],
    datasets: [{
      label: 'Rent Collected',
      data: Array(12).fill(0),
      fill: true,
      backgroundColor: gradientFill,
      borderColor: '#7b61ff',
      borderWidth: 2.6,
      pointRadius: 3.5,
      tension: 0.36,
    }]
  },
  options: {
    plugins: { legend:{display:false} },
    scales: {
      x: { grid: { display:false }, ticks:{ color:'rgba(255,255,255,0.6)'} },
      y: { grid: { color:'rgba(255,255,255,0.03)' }, ticks:{ color:'rgba(255,255,255,0.6)' } }
    },
    responsive:true, maintainAspectRatio:false, interaction:{mode:'index', intersect:false}
  }
});

const maintenanceCtx = document.getElementById('chartMaintenance').getContext('2d');
const chartMaint = new Chart(maintenanceCtx, {
  type:'doughnut',
  data:{
    labels:['Pending','In Progress','Completed'],
    datasets:[{data:[0,0,0], backgroundColor:['#f39c12','#3498db','#2ecc71'], hoverOffset:6}]
  },
  options:{ plugins:{legend:{labels:{color:'rgba(255,255,255,0.8)'}}}, responsive:true, maintainAspectRatio:false }
});

/* ---------------- render functions ---------------- */
function renderKPIs(){
  $('#kpiTotal').textContent = properties.length;
  $('#kpiOccupied').textContent = properties.filter(p=>p.status==='Occupied').length;
  $('#kpiVacant').textContent = properties.filter(p=>p.status==='Vacant').length;
  $('#kpiLate').textContent = tenants.filter(t=>t.status==='Late').length || 0;
}

function renderTables(){
  const tbodyP = $('#tblProperties'); tbodyP.innerHTML='';
  properties.forEach(p=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td style="min-width:220px">${p.address}</td>
      <td>${p.type}</td>
      <td>${p.bedrooms}</td>
      <td>${p.bathrooms}</td>
      <td><span class="badge-soft">${p.status}</span></td>
      <td>${p.tenant ? `<a href="#" class="text-decoration-none text-info" onclick="openTenant('${p.tenant}')">${p.tenant}</a>` : '-'}</td>
      <td>$${p.rent.toLocaleString()}</td>
      <td>${p.pendingApplications.length>0 ? `<button class="btn btn-sm btn-warning" onclick="openPending('${p.address}')">${p.pendingApplications.length} Pending</button>` : '-'}</td>
    `;
    tbodyP.appendChild(tr);
  });

  const tbodyT = $('#tblTenants'); tbodyT.innerHTML='';
  tenants.forEach(t=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><a href="#" class="text-decoration-none text-info" onclick="openTenant('${t.name}')">${t.name}</a></td>
      <td>${t.property}</td>
      <td>${t.leaseEnd}</td>
      <td>$${t.monthlyRent.toLocaleString()}</td>
      <td><span class="badge-soft">${t.status}</span></td>
    `;
    tbodyT.appendChild(tr);
  });

  // maintenance table for modal
  const tbodyM = $('#tblMaintenance'); tbodyM.innerHTML='';
  maintenanceRequests.forEach((r, i)=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${r.id}</td><td>${r.property}</td><td>${r.tenant}</td>
      <td style="max-width:420px">${r.description}</td>
      <td>${r.status}</td><td>${r.date}</td>
      <td>
        ${r.status!=='In Progress'?`<button class="btn btn-sm btn-warning me-1" onclick="updateMaint(${i}, 'In Progress')">In Progress</button>`:''}
        ${r.status!=='Completed'?`<button class="btn btn-sm btn-success" onclick="updateMaint(${i}, 'Completed')">Completed</button>`:''}
      </td>
    `;
    tbodyM.appendChild(tr);
  });

  $('#maintenanceCount').textContent = `${maintenanceRequests.length} requests`;
}

/* ---------------- charts update ---------------- */
function refreshCharts(){
  // rent chart: naive example — sum monthly rents to current month
  const monthly = Array(12).fill(0);
  const monthNow = new Date().getMonth();
  // distribute each tenant's rent to current month only for demo
  monthly[monthNow] = tenants.reduce((s,t)=>s + (t.status!=='Late' ? t.monthlyRent : 0), 0);
  chartRent.data.datasets[0].data = monthly;
  chartRent.update();

  // maintenance
  const pending = maintenanceRequests.filter(r=>r.status==='Pending').length;
  const inprogress = maintenanceRequests.filter(r=>r.status==='In Progress').length;
  const complete = maintenanceRequests.filter(r=>r.status==='Completed').length;
  chartMaint.data.datasets[0].data = [pending, inprogress, complete];
  chartMaint.update();
}

/* ---------------- modal helpers ---------------- */
function openMaintenanceModal(){
  const modal = new bootstrap.Modal($('#maintenanceModal'));
  modal.show();
  // render already done by renderTables
}
function openTenant(name){
  const tenant = tenants.find(t=>t.name===name);
  if(!tenant) return;
  const body = $('#tenantModalBody');
  body.innerHTML = `
    <div class="row gx-4">
      <div class="col-md-6">
        <h5 style="margin-top:0">${tenant.name}</h5>
        <div class="small-muted">Contact</div>
        <p>Phone: (555) 123-4567<br>Email: ${tenant.name.toLowerCase().replace(' ','.')}@example.com</p>
        <div class="small-muted">Lease</div>
        <p>Start: ${tenant.leaseStart}<br>End: ${tenant.leaseEnd}</p>
      </div>
      <div class="col-md-6">
        <h6>Financial</h6>
        <p>Monthly Rent: <strong>$${tenant.monthlyRent.toLocaleString()}</strong></p>
        <p>Status: <span class="badge-soft">${tenant.status}</span></p>
        <h6 class="mt-3">Maintenance</h6>
        <div>${maintenanceRequests.filter(r=>r.tenant===tenant.name).map(r=>`<div style="margin-bottom:6px"><strong>${r.description}</strong><div class="small-muted">${r.date} • ${r.status}</div></div>`).join('') || '<div class="small-muted">No requests</div>'}</div>
      </div>
    </div>
  `;
  new bootstrap.Modal($('#tenantModal')).show();
}

function openPending(address){
  // show a small modal or route — here we use tenant modal to keep simple
  const property = properties.find(p=>p.address===address);
  if(!property) return;
  const body = $('#tenantModalBody');
  body.innerHTML = `<h5>Pending Applications - ${address}</h5>`;
  if(!property.pendingApplications.length) body.innerHTML += '<p class="small-muted">No pending applications</p>';
  else{
    let html = '<div class="list-group">';
    property.pendingApplications.forEach((a,i)=>{
      html += `<div class="list-group-item d-flex justify-content-between align-items-center">
        <div><strong>${a.name}</strong><div class="small-muted">Lease: ${a.leaseStart} — ${a.leaseEnd}</div></div>
        <div><button class="btn btn-sm btn-success me-1" onclick="approvePending('${address}',${i})">Approve</button><button class="btn btn-sm btn-danger" onclick="declinePending('${address}',${i})">Decline</button></div>
      </div>`;
    });
    html += `</div>`;
    body.innerHTML += html;
  }
  new bootstrap.Modal($('#tenantModal')).show();
}

/* ---------------- actions ---------------- */
function updateMaint(i, status){
  maintenanceRequests[i].status = status;
  renderTables(); refreshCharts();
}

function approvePending(address, idx){
  const prop = properties.find(p=>p.address===address);
  const app = prop.pendingApplications.splice(idx,1)[0];
  prop.status='Occupied'; prop.tenant = app.name; prop.rent = app.monthlyRent;
  tenants.push({ name: app.name, property: address, leaseStart: app.leaseStart||new Date().toISOString().slice(0,10), leaseEnd: app.leaseEnd||'', monthlyRent: app.monthlyRent, status:'Active' });
  renderTables(); refreshCharts();
}

function declinePending(address, idx){
  const prop = properties.find(p=>p.address===address);
  prop.pendingApplications.splice(idx,1);
  renderTables();
}

/* ---------------- searching + filtering ---------------- */
$('#searchProperties').addEventListener('input', e=>{
  const q = e.target.value.toLowerCase();
  $$('#tblProperties tr').forEach(tr=>{
    tr.style.display = Array.from(tr.cells).some(td=>td.textContent.toLowerCase().includes(q)) ? '' : 'none';
  });
});
$('#searchTenants').addEventListener('input', e=>{
  const q = e.target.value.toLowerCase();
  $$('#tblTenants tr').forEach(tr=>{
    tr.style.display = Array.from(tr.cells).some(td=>td.textContent.toLowerCase().includes(q)) ? '' : 'none';
  });
});
$('#searchMaintenance')?.addEventListener('input', e=>{
  const q = e.target.value.toLowerCase();
  $$('#tblMaintenance tr').forEach(tr=>{
    tr.style.display = Array.from(tr.cells).some(td=>td.textContent.toLowerCase().includes(q)) ? '' : 'none';
  });
});

/* filter for maintenance modal */
function filterMaintenance(status){
  $$('#tblMaintenance tr').forEach(tr=>{
    if(status==='All') tr.style.display='';
    else tr.style.display = tr.cells[4].textContent === status ? '' : 'none';
  });
}

/* global search: quick focus on matched section */
$('#globalSearch').addEventListener('keydown', (e)=>{
  if(e.key === 'Enter'){
    const q = e.target.value.toLowerCase();
    // quick heuristic: search tenants first, then properties
    const t = tenants.find(t=>t.name.toLowerCase().includes(q));
    if(t){ openTenant(t.name); return; }
    const p = properties.find(p=>p.address.toLowerCase().includes(q));
    if(p){ openPending(p.address); return; }
    alert('No quick matches found.');
  }
});

/* sidebar interactions */
$$('.nav-item').forEach(item=>{
  item.addEventListener('click', ()=>{
    $$('.nav-item').forEach(n=>n.classList.remove('active'));
    item.classList.add('active');
    const section = item.getAttribute('data-section');
    if(section){
      // show section areas — here: map custom to original smaller app sections
      showSection(section);
    } else if(item.id === 'maintenanceTrigger'){
      openMaintenanceModal();
    }
  });
});

function showSection(id){
  // in this condensed UI we only hide/show content in main area — simple approach
  // sections: sectionOverview, sectionProperties, sectionTenants, sectionPayments
  if(id === 'sectionOverview'){
    // scroll to top (overview is the default area in our layout)
    window.scrollTo({top:0,behavior:'smooth'});
  } else if(id === 'sectionProperties'){
    // focus properties
    $('#searchProperties').focus();
  } else if(id === 'sectionTenants'){
    $('#searchTenants').focus();
  } else if(id === 'sectionPayments'){
    // optional: focus chart
    document.getElementById('chartRent').scrollIntoView({behavior:'smooth'});
  }
}

/* ---------------- init ---------------- */
function init(){
  renderKPIs();
  renderTables();
  refreshCharts();
}
init();

</script>
</body>
</html>
