# Rental-Management-dashboard
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Interactive Rental Dashboard</title>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f4f6f9; }
.sidebar { height: 100vh; background: #2c3e50; color: white; position: fixed; width: 250px; padding-top: 20px; }
.sidebar h2 { text-align: center; margin-bottom: 30px; font-size: 1.5rem; }
.sidebar a { color: white; display: block; padding: 10px 20px; text-decoration: none; margin-bottom: 5px; border-radius: 5px; }
.sidebar a:hover { background: #34495e; }
.main { margin-left: 250px; padding: 20px; }
.kpi-card { padding: 15px; height: 120px; display: flex; flex-direction: column; justify-content: center; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); color: white; }
.kpi-card h5 { font-size: 1rem; margin-bottom: 5px; }
.kpi-card h2 { font-size: 1.5rem; margin: 0; }
.card { border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
table th, table td { vertical-align: middle !important; }
.chart-container { height: 300px; }
</style>
</head>
<body>

<div class="sidebar">
  <h2>Rental Dashboard</h2>
  <a href="#">Overview</a>
  <a href="#">Properties</a>
  <a href="#">Tenants</a>
  <a href="#">Payments</a>
  <a href="#">Maintenance</a>
  <a href="#">Reports</a>
</div>

<div class="main">
  <h1 class="mb-4">Property Rental Management Dashboard</h1>

  <!-- KPI Cards -->
  <div class="row mb-4 g-3">
    <div class="col-md-3"><div class="kpi-card bg-primary text-center"><h5>Total Properties</h5><h2 id="totalProperties">0</h2></div></div>
    <div class="col-md-3"><div class="kpi-card bg-success text-center"><h5>Occupied</h5><h2 id="occupiedProperties">0</h2></div></div>
    <div class="col-md-3"><div class="kpi-card bg-danger text-center"><h5>Vacant</h5><h2 id="vacantProperties">0</h2></div></div>
    <div class="col-md-3"><div class="kpi-card bg-warning text-center"><h5>Late Payments</h5><h2 id="latePayments">0</h2></div></div>
  </div>

  <!-- Charts -->
  <div class="row mb-4 g-3">
    <div class="col-md-6">
      <div class="card p-3"><h5>Monthly Rent Collection ($)</h5><div class="chart-container"><canvas id="rentChart"></canvas></div></div>
    </div>
    <div class="col-md-6">
      <div class="card p-3"><h5>Maintenance Requests Status</h5><div class="chart-container"><canvas id="maintenanceChart"></canvas></div></div>
    </div>
  </div>

  <!-- Properties Table -->
  <div class="card p-3 mb-4">
    <h5>Properties Overview</h5>
    <table class="table table-striped table-hover mt-3" id="propertiesTable">
      <thead class="table-dark"><tr><th>Property</th><th>Type</th><th>Bedrooms</th><th>Bathrooms</th><th>Status</th><th>Tenant</th><th>Rent ($)</th><th>Pending Applications</th></tr></thead>
      <tbody></tbody>
    </table>
  </div>

  <!-- Tenants Table -->
  <div class="card p-3 mb-4">
    <h5>Tenants Overview</h5>
    <table class="table table-striped table-hover mt-3" id="tenantsTable">
      <thead class="table-dark"><tr><th>Name</th><th>Property</th><th>Lease Start</th><th>Lease End</th><th>Monthly Rent</th><th>Status</th></tr></thead>
      <tbody></tbody>
    </table>
  </div>
</div>

<!-- Modal for Pending Applications -->
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
/* Initialize Charts */
const rentCtx = document.getElementById('rentChart').getContext('2d');
const rentChart = new Chart(rentCtx, {type:'line', data:{labels:['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'], datasets:[{label:'Rent Collected', data:Array(12).fill(0), backgroundColor:'rgba(39,174,96,0.2)', borderColor:'rgba(39,174,96,1)', borderWidth:2, fill:true, tension:0.4, pointRadius:5, pointBackgroundColor:'rgba(39,174,96,1)'}]}, options:{responsive:true, maintainAspectRatio:false, plugins:{legend:{display:true}}, scales:{y:{beginAtZero:true, ticks:{stepSize:1000}}, x:{ticks:{maxRotation:0}}}}});

const maintenanceCtx = document.getElementById('maintenanceChart').getContext('2d');
const maintenanceChart = new Chart(maintenanceCtx, {type:'doughnut', data:{labels:['Pending','In Progress','Completed'], datasets:[{label:'Maintenance Status', data:[0,0,0], backgroundColor:['#e74c3c','#f1c40f','#2ecc71']}]}, options:{responsive:true, maintainAspectRatio:false}});

/* Dashboard State */
let properties = [];
let tenants = [];
let maintenanceCounts = {Pending:0, InProgress:0, Completed:0};

/* Add Rental Application */
function addRentalApplication(app) {
  let property = properties.find(p=>p.address===app.propertyAddress);
  if(!property){
    property={address:app.propertyAddress,type:app.propertyType,bedrooms:app.bedrooms,bathrooms:app.bathrooms,status:app.tenantName?'Occupied':'Vacant',tenant:app.tenantName||'-',rent:app.monthlyRent||0,pendingApplications:[]};
    properties.push(property);
  } else {
    if(property.status==='Occupied'){property.pendingApplications.push({name:app.tenantName,leaseStart:app.leaseStart,leaseEnd:app.leaseEnd,monthlyRent:app.monthlyRent});}
    else{property.tenant=app.tenantName; property.rent=app.monthlyRent; property.status='Occupied';}
  }
  updateDashboard();
}

/* Update Dashboard */
function updateDashboard(){
  document.getElementById('totalProperties').innerText = properties.length;
  const occupiedCount = properties.filter(p=>p.status==='Occupied').length;
  document.getElementById('occupiedProperties').innerText = occupiedCount;
  document.getElementById('vacantProperties').innerText = properties.length-occupiedCount;
  document.getElementById('latePayments').innerText = 0; 

  renderPropertiesTable();
  renderTenantsTable();
  updateCharts();
}

/* Render Properties Table */
function renderPropertiesTable(){
  const propTable=document.querySelector('#propertiesTable tbody');
  propTable.innerHTML='';
  properties.forEach(p=>{
    const pendingBtn=p.pendingApplications.length>0?`<button class="btn btn-sm btn-info" onclick="showPendingApps('${p.address}')">View Pending (${p.pendingApplications.length})</button>`:'-';
    propTable.innerHTML+=`<tr><td>${p.address}</td><td>${p.type}</td><td>${p.bedrooms}</td><td>${p.bathrooms}</td><td><span class="badge bg-${p.status==='Occupied'?'success':'danger'}">${p.status}</span></td><td>${p.tenant}</td><td>${p.rent}</td><td>${pendingBtn}</td></tr>`;
  });
}

/* Render Tenants Table */
function renderTenantsTable(){
  const tenantTable=document.querySelector('#tenantsTable tbody');
  tenantTable.innerHTML='';
  tenants=[];
  properties.forEach(p=>{if(p.status==='Occupied')tenants.push({name:p.tenant,property:p.address,leaseStart:'-',leaseEnd:'-',monthlyRent:p.rent,status:'Active'});});
  tenants.forEach(t=>{
    tenantTable.innerHTML+=`<tr><td>${t.name}</td><td>${t.property}</td><td>${t.leaseStart}</td><td>${t.leaseEnd}</td><td>${t.monthlyRent}</td><td><span class="badge bg-success">${t.status}</span></td></tr>`;
  });
}

/* Update Charts */
function updateCharts(){
  const month=new Date().getMonth();
  rentChart.data.datasets[0].data[month]=tenants.reduce((sum,t)=>sum+t.monthlyRent,0);
  rentChart.update();
  maintenanceChart.data.datasets[0].data=[maintenanceCounts.Pending,maintenanceCounts.InProgress,maintenanceCounts.Completed];
  maintenanceChart.update();
}

/* Show Pending Applications in Modal */
function showPendingApps(address){
  const property=properties.find(p=>p.address===address);
  const modalBody=document.getElementById('pendingModalBody');
  modalBody.innerHTML='';
  if(!property || property.pendingApplications.length===0){modalBody.innerHTML='<p>No pending applications.</p>';}
  else{
    property.pendingApplications.forEach((app,i)=>{
      const div=document.createElement('div');
      div.classList.add('d-flex','justify-content-between','align-items-center','mb-2');
      div.innerHTML=`<span>${app.name} | Lease: ${app.leaseStart} - ${app.leaseEnd} | Rent: $${app.monthlyRent}</span>
      <div>
        <button class="btn btn-sm btn-success me-1" onclick="approveApplication('${address}',${i})">Approve</button>
        <button class="btn btn-sm btn-danger" onclick="declineApplication('${address}',${i})">Decline</button>
      </div>`;
      modalBody.appendChild(div);
    });
  }
  new bootstrap.Modal(document.getElementById('pendingModal')).show();
}

/* Approve / Decline Functions */
function approveApplication(address,index){
  const property=properties.find(p=>p.address===address);
  const app=property.pendingApplications.splice(index,1)[0];
  property.tenant=app.name;
  property.rent=app.monthlyRent;
  property.status='Occupied';
  updateDashboard();
  showPendingApps(address); // Refresh modal
}

function declineApplication(address,index){
  const property=properties.find(p=>p.address===address);
  property.pendingApplications.splice(index,1);
  updateDashboard();
  showPendingApps(address); // Refresh modal
}

/* Example Applications */
addRentalApplication({propertyAddress:"123 Main St",propertyType:"Apartment",bedrooms:2,bathrooms:1,tenantName:"John Doe",leaseStart:"2025-01-01",leaseEnd:"2026-01-01",monthlyRent:2200});
addRentalApplication({propertyAddress:"456 Oak Ave",propertyType:"House",bedrooms:3,bathrooms:2});
addRentalApplication({propertyAddress:"123 Main St",propertyType:"Apartment",bedrooms:2,bathrooms:1,tenantName:"Alice Johnson",leaseStart:"2025-10-01",leaseEnd:"2026-09-30",monthlyRent:2400});
addRentalApplication({propertyAddress:"789 Pine Ln",propertyType:"Condo",bedrooms:2,bathrooms:2,tenantName:"Jane Smith",leaseStart:"2024-09-01",leaseEnd:"2025-09-01",monthlyRent:2500});
</script>
</body>
</html>
