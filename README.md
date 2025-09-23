
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Rental Management Dashboard</title>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f4f6f9; }
.sidebar { height: 100vh; background: #2c3e50; color: white; padding-top: 20px; width: 180px; transition: width 0.3s; position: fixed; }
.sidebar.collapsed { width: 50px; }
.sidebar h2 { text-align: center; font-size: 1.3rem; margin-bottom: 20px; }
.sidebar a { color: white; display: block; padding: 10px 15px; text-decoration: none; margin-bottom: 5px; border-radius: 5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.sidebar a:hover { background: #34495e; }
.sidebar-link.active { background-color: #34495e; font-weight: bold; }
.main { margin-left: 180px; transition: margin-left 0.3s; padding: 20px; }
.main.expanded { margin-left: 50px; }
.kpi-card { padding: 15px; height: 120px; display: flex; flex-direction: column; justify-content: center; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); color: white; }
.kpi-card h5 { font-size: 1rem; margin-bottom: 5px; }
.kpi-card h2 { font-size: 1.5rem; margin: 0; }
.card { border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
table th, table td { vertical-align: middle !important; }
.chart-container { height: 300px; }
.dashboard-section { display: none; }
input.search-box { margin-bottom: 10px; }
.table-striped tbody tr.overdue { background-color: #f8d7da; } /* overdue rent */
.table-striped tbody tr.expiring { background-color: #fff3cd; } /* lease expiring soon */
</style>
</head>
<body>

<!-- Sidebar -->
<div class="sidebar" id="sidebar">
  <h2>Rental Dashboard</h2>
  <a href="#" class="sidebar-link" data-section="sectionOverview">Overview</a>
  <a href="#" class="sidebar-link" data-section="sectionProperties">Properties</a>
  <a href="#" class="sidebar-link" data-section="sectionTenants">Tenants</a>
  <a href="#" class="sidebar-link" data-section="sectionPayments">Payments</a>
  <a href="#" class="sidebar-link" data-section="sectionMaintenance">Maintenance</a>
  <button class="btn btn-sm btn-light mt-3 ms-3" id="toggleSidebar">Toggle</button>
</div>

<!-- Main Content -->
<div class="main" id="mainContent">
  <h1 class="mb-4">Rental Management Dashboard</h1>

  <!-- Overview Section -->
  <div id="sectionOverview" class="dashboard-section">
    <div class="row mb-4 g-3">
      <div class="col-md-3"><div class="kpi-card bg-primary text-center"><h5>Total Properties</h5><h2 id="totalProperties">0</h2></div></div>
      <div class="col-md-3"><div class="kpi-card bg-success text-center"><h5>Occupied</h5><h2 id="occupiedProperties">0</h2></div></div>
      <div class="col-md-3"><div class="kpi-card bg-danger text-center"><h5>Vacant</h5><h2 id="vacantProperties">0</h2></div></div>
      <div class="col-md-3"><div class="kpi-card bg-warning text-center"><h5>Late Payments</h5><h2 id="latePayments">0</h2></div></div>
    </div>
    <div class="row mb-4 g-3">
      <div class="col-md-6">
        <div class="card p-3"><h5>Monthly Rent Collection ($)</h5><div class="chart-container"><canvas id="rentChart"></canvas></div></div>
      </div>
      <div class="col-md-6">
        <div class="card p-3"><h5>Maintenance Requests Status</h5><div class="chart-container"><canvas id="maintenanceChart"></canvas></div></div>
      </div>
    </div>
  </div>

  <!-- Properties Section -->
  <div id="sectionProperties" class="dashboard-section">
    <input type="text" id="searchProperties" class="form-control search-box" placeholder="Search Properties...">
    <div class="card p-3 mb-4">
      <h5>Properties Overview</h5>
      <table class="table table-striped table-hover mt-3" id="propertiesTable">
        <thead class="table-dark">
          <tr>
            <th data-sort="address">Property</th>
            <th data-sort="type">Type</th>
            <th data-sort="bedrooms">Bedrooms</th>
            <th data-sort="bathrooms">Bathrooms</th>
            <th data-sort="status">Status</th>
            <th>Tenant</th>
            <th data-sort="rent">Rent ($)</th>
            <th>Pending Applications</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
      <button class="btn btn-sm btn-outline-primary mt-2" onclick="exportTableToCSV('properties.csv')">Export CSV</button>
    </div>
  </div>

  <!-- Tenants Section -->
  <div id="sectionTenants" class="dashboard-section">
    <input type="text" id="searchTenants" class="form-control search-box" placeholder="Search Tenants...">
    <div class="card p-3 mb-4">
      <h5>Tenants Overview</h5>
      <table class="table table-striped table-hover mt-3" id="tenantsTable">
        <thead class="table-dark">
          <tr>
            <th data-sort="name">Name</th>
            <th data-sort="property">Property</th>
            <th data-sort="leaseStart">Lease Start</th>
            <th data-sort="leaseEnd">Lease End</th>
            <th data-sort="monthlyRent">Monthly Rent</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
      <button class="btn btn-sm btn-outline-primary mt-2" onclick="exportTableToCSV('tenants.csv')">Export CSV</button>
    </div>
  </div>

  <!-- Payments Section -->
  <div id="sectionPayments" class="dashboard-section">
    <div class="card p-3 mb-4">
      <h5>Payments Overview</h5>
      <div class="chart-container">
        <canvas id="paymentsChart"></canvas>
      </div>
    </div>
  </div>

  <!-- Maintenance Section -->
  <div id="sectionMaintenance" class="dashboard-section">
    <div class="card p-3 mb-4">
      <h5>Maintenance Overview</h5>
      <div class="chart-container">
        <canvas id="maintenanceOverviewChart"></canvas>
      </div>
    </div>
  </div>

</div>

<!-- Pending Applications Modal -->
<div class="modal fade" id="pendingModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-lg">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Pending Applications</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body" id="pendingModalBody"></div>
    </div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>

// ---------------- Sidebar & Sections ----------------
const sidebar = document.getElementById('sidebar');
const mainContent = document.getElementById('mainContent');
document.getElementById('toggleSidebar').addEventListener('click', ()=>{
  sidebar.classList.toggle('collapsed');
  mainContent.classList.toggle('expanded');
});

const sections = document.querySelectorAll('.dashboard-section');
const sidebarLinks = document.querySelectorAll('.sidebar-link');

function showSection(sectionId){
  sections.forEach(sec => sec.style.display='none');
  document.getElementById(sectionId).style.display='block';
  sidebarLinks.forEach(link=>link.classList.remove('active'));
  document.querySelector(`.sidebar-link[data-section="${sectionId}"]`).classList.add('active');
}

sidebarLinks.forEach(link=>{
  link.addEventListener('click', e=>{
    e.preventDefault();
    showSection(link.getAttribute('data-section'));
  });
});

showSection('sectionProperties');

// ---------------- Charts ----------------
const rentCtx = document.getElementById('rentChart').getContext('2d');
const rentChart = new Chart(rentCtx, {type:'line', data:{labels:['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'], datasets:[{label:'Rent Collected', data:Array(12).fill(0), backgroundColor:'rgba(39,174,96,0.2)', borderColor:'rgba(39,174,96,1)', borderWidth:2, fill:true, tension:0.4, pointRadius:5, pointBackgroundColor:''rgba(39,174,96,1)'}]}, options:{responsive:true, maintainAspectRatio:false}});
const maintenanceCtx = document.getElementById('maintenanceChart').getContext('2d');
const maintenanceChart = new Chart(maintenanceCtx, {type:'doughnut', data:{labels:['Pending','In Progress','Completed'], datasets:[{label:'Maintenance Status', data:[0,0,0], backgroundColor:['#e74c3c','#f1c40f','#2ecc71']}]}, options:{responsive:true, maintainAspectRatio:false}});

// ---------------- Sample Data ----------------
let properties = [
  {address:"123 Main St",type:"Apartment",bedrooms:2,bathrooms:1,status:"Occupied",tenant:"John Doe",rent:2200,pendingApplications:[{name:"Alice Johnson",leaseStart:"2025-10-01",leaseEnd:"2026-09-30",monthlyRent:2400}]},
  {address:"456 Oak Ave",type:"House",bedrooms:3,bathrooms:2,status:"Vacant",tenant:"",rent:0,pendingApplications:[]},
  {address:"789 Pine Ln",type:"Condo",bedrooms:2,bathrooms:2,status:"Occupied",tenant:"Jane Smith",rent:2500,pendingApplications:[]}
];
let tenants = [];

// ---------------- Dashboard Update ----------------
function updateDashboard(){
  document.getElementById('totalProperties').innerText = properties.length;
  const occupiedCount = properties.filter(p=>p.status==='Occupied').length;
  document.getElementById('occupiedProperties').innerText = occupiedCount;
  document.getElementById('vacantProperties').innerText = properties.length - occupiedCount;
  document.getElementById('latePayments').innerText = 0; // Add overdue logic later

  renderPropertiesTable();
  renderTenantsTable();
  updateCharts();
}

// ---------------- Render Properties Table ----------------
function renderPropertiesTable(){
  const tbody = document.querySelector('#propertiesTable tbody');
  tbody.innerHTML = '';
  properties.forEach(p=>{
    const pendingBtn = p.pendingApplications.length>0 ? `<button class="btn btn-sm btn-info" onclick="showPendingApps('${p.address}')">View Pending (${p.pendingApplications.length})</button>` : '-';
    tbody.innerHTML += `<tr>
      <td>${p.address}</td>
      <td>${p.type}</td>
      <td>${p.bedrooms}</td>
      <td>${p.bathrooms}</td>
      <td><span class="badge bg-${p.status==='Occupied'?'success':'danger'}">${p.status}</span></td>
      <td>${p.tenant || '-'}</td>
      <td>${p.rent}</td>
      <td>${pendingBtn}</td>
    </tr>`;
  });
}

// ---------------- Render Tenants Table ----------------
function renderTenantsTable(){
  const tbody = document.querySelector('#tenantsTable tbody');
  tbody.innerHTML = '';
  tenants = [];
  properties.forEach(p=>{
    if(p.status==='Occupied') tenants.push({name:p.tenant, property:p.address, leaseStart:'-', leaseEnd:'-', monthlyRent:p.rent, status:'Active'});
  });
  tenants.forEach(t=>{
    tbody.innerHTML += `<tr>
      <td>${t.name}</td>
      <td>${t.property}</td>
      <td>${t.leaseStart}</td>
      <td>${t.leaseEnd}</td>
      <td>${t.monthlyRent}</td>
      <td><span class="badge bg-success">${t.status}</span></td>
    </tr>`;
  });
}

// ---------------- Update Charts ----------------
function updateCharts(){
  const month = new Date().getMonth();
  rentChart.data.datasets[0].data[month] = tenants.reduce((sum,t)=>sum+t.monthlyRent,0);
  rentChart.update();

  maintenanceChart.data.datasets[0].data = [0,0,0]; // Placeholder
  maintenanceChart.update();
}

// ---------------- Pending Applications Modal ----------------
function showPendingApps(address){
  const property = properties.find(p=>p.address===address);
  const modalBody = document.getElementById('pendingModalBody');
  modalBody.innerHTML = '';
  if(!property || property.pendingApplications.length===0){
    modalBody.innerHTML = '<p>No pending applications.</p>';
  } else {
    property.pendingApplications.forEach((app,i)=>{
      const div = document.createElement('div');
      div.classList.add('d-flex','justify-content-between','align-items-center','mb-2');
      div.innerHTML = `<span>${app.name} | Lease: ${app.leaseStart} - ${app.leaseEnd} | Rent: $${app.monthlyRent}</span>
      <div>
        <button class="btn btn-sm btn-success me-2" onclick="approveApp(${i}, '${property.address}')">Approve</button>
        <button class="btn btn-sm btn-danger" onclick="declineApp(${i}, '${property.address}')">Decline</button>
      </div>`;
      modalBody.appendChild(div);
    });
  }
  new bootstrap.Modal(document.getElementById('pendingModal')).show();
}

// ---------------- Approve / Decline Functions ----------------
function approveApp(index, address){
  const property = properties.find(p=>p.address===address);
  const app = property.pendingApplications.splice(index,1)[0];
  property.tenant = app.name;
  property.status = 'Occupied';
  property.rent = app.monthlyRent;
  updateDashboard();
  showPendingApps(address);
}

function declineApp(index, address){
  const property = properties.find(p=>p.address===address);
  property.pendingApplications.splice(index,1);
  showPendingApps(address);
  updateDashboard();
}

// ---------------- Search Functionality ----------------
document.getElementById('searchProperties').addEventListener('input', e=>{
  const term = e.target.value.toLowerCase();
  document.querySelectorAll('#propertiesTable tbody tr').forEach(tr=>{
    tr.style.display = Array.from(tr.children).some(td=>td.textContent.toLowerCase().includes(term)) ? '' : 'none';
  });
});
document.getElementById('searchTenants').addEventListener('input', e=>{
  const term = e.target.value.toLowerCase();
  document.querySelectorAll('#tenantsTable tbody tr').forEach(tr=>{
    tr.style.display = Array.from(tr.children).some(td=>td.textContent.toLowerCase().includes(term)) ? '' : 'none';
  });
});

// ---------------- CSV Export ----------------
function exportTableToCSV(filename){
  let csv = '';
  const table = document.querySelector('#propertiesTable') || document.querySelector('#tenantsTable');
  const rows = table.querySelectorAll('tr');
  rows.forEach(row=>{
    const cols = row.querySelectorAll('td, th');
    csv += Array.from(cols).map(td=>td.innerText.replace(/,/g,'')).join(',') + '\n';
  });
  const blob = new Blob([csv], {type:'text/csv'});
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = filename;
  link.click();
}

// ---------------- Initialize Dashboard ----------------
updateDashboard();

</script>
</body>
</html>
