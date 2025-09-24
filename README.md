<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Luxury Rental Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
/* ===== Base Styles ===== */
* {margin:0; padding:0; box-sizing:border-box;}
body {font-family: 'Inter', sans-serif; background: linear-gradient(135deg, #141e30, #243b55); color: #fff; display:flex; height:100vh;}
.dashboard {display:flex; width:100%;}

/* ===== Sidebar ===== */
.sidebar {width:220px; background: rgba(255,255,255,0.05); backdrop-filter: blur(12px); padding:2rem 1rem; display:flex; flex-direction:column; border-right:1px solid rgba(255,255,255,0.1);}
.logo {font-size:1.5rem; font-weight:700; margin-bottom:2rem; text-align:center;}
.sidebar nav a {display:block; color:#ddd; padding:0.8rem 1rem; margin:0.3rem 0; border-radius:12px; text-decoration:none; transition:0.3s;}
.sidebar nav a.active, .sidebar nav a:hover {background: rgba(255,255,255,0.2); color:#fff;}

/* ===== Main ===== */
.main {flex:1; padding:2rem; overflow-y:auto;}
.header {margin-bottom:2rem;}
.header h1 {font-size:2rem; font-weight:700;}
.header p {color:#ccc;}

/* ===== KPI Cards ===== */
.kpis {display:grid; grid-template-columns: repeat(auto-fit, minmax(220px,1fr)); gap:1.5rem; margin-bottom:2rem;}
.card {padding:1.5rem; border-radius:20px; text-align:center; cursor:pointer; transition:0.3s;}
.card:hover {transform: translateY(-5px);}
.glass {background: rgba(255,255,255,0.1); backdrop-filter: blur(14px); box-shadow: 0 8px 32px rgba(0,0,0,0.2);}
.card h3 {font-size:1rem; margin-bottom:0.5rem; font-weight:600;}
.card .value {font-size:1.8rem; font-weight:700;}

/* ===== Charts ===== */
.charts {display:grid; grid-template-columns: repeat(auto-fit, minmax(280px,1fr)); gap:1.5rem; margin-bottom:2rem;}
.chart-card {padding:1.5rem; border-radius:20px;}
.chart-card h3 {margin-bottom:1rem; font-weight:600;}

/* ===== Lists ===== */
.lists {display:grid; grid-template-columns: repeat(auto-fit, minmax(300px,1fr)); gap:1.5rem; margin-bottom:2rem;}
.lists ul {list-style:none; max-height:200px; overflow-y:auto;}

/* ===== Modals ===== */
.modal-overlay {position:fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.5); display:flex; justify-content:center; align-items:center; z-index:1000;}
.modal-overlay.hidden {display:none;}
.modal {background: rgba(255,255,255,0.15); backdrop-filter: blur(14px); padding:2rem; border-radius:20px; width:80%; max-width:800px; max-height:80%; overflow-y:auto; position:relative;}
.close-modal {position:absolute; top:1rem; right:1rem; font-size:1.5rem; background:none; border:none; color:#fff; cursor:pointer;}
</style>
</head>
<body>
<div class="dashboard">
  <!-- Sidebar -->
  <aside class="sidebar">
    <h2 class="logo">🏠 Rentals</h2>
    <nav>
      <a href="#overview" class="active">Overview</a>
      <a href="#properties" id="properties-tab">Properties</a>
      <a href="#tenants" id="tenants-tab">Tenants</a>
      <a href="#maintenance" id="maintenance-tab">Maintenance</a>
      <a href="#applications" id="applications-tab">Applications</a>
    </nav>
  </aside>

  <!-- Main -->
  <main class="main">
    <header class="header">
      <h1>Dashboard</h1>
      <p>Luxury Property Management Overview</p>
    </header>

    <!-- KPI Cards -->
    <section class="kpis">
      <div id="total-properties" class="card glass"> <h3>Total Properties</h3> <p class="value">0</p> </div>
      <div id="occupied-properties" class="card glass"> <h3>Occupied</h3> <p class="value">0</p> </div>
      <div id="vacant-properties" class="card glass"> <h3>Vacant</h3> <p class="value">0</p> </div>
      <div id="monthly-rent" class="card glass"> <h3>Monthly Rent Collected</h3> <p class="value">$0</p> </div>
    </section>

    <!-- Charts -->
    <section class="charts">
      <div class="chart-card glass"> <h3>Rent Collection</h3> <canvas id="rentChart"></canvas> </div>
      <div class="chart-card glass"> <h3>Expenses</h3> <canvas id="expenseChart"></canvas> </div>
      <div class="chart-card glass"> <h3>Occupancy Rate</h3> <canvas id="occupancyChart"></canvas> </div>
    </section>

    <!-- Lists -->
    <section class="lists">
      <div id="maintenance-list" class="card glass"> <h3>Maintenance Requests</h3> <ul id="maintenance-items"></ul> </div>
      <div id="tenant-list" class="card glass"> <h3>Tenants</h3> <ul id="tenant-items"></ul> </div>
      <div id="applications-list" class="card glass"> <h3>Pending Applications</h3> <ul id="application-items"></ul> </div>
    </section>

  </main>
</div>

<!-- Modal -->
<div id="modal-overlay" class="modal-overlay hidden">
  <div class="modal glass">
    <button class="close-modal">&times;</button>
    <div id="modal-content"></div>
  </div>
</div>

<script>
// Sample Data
const properties = [
  {name: "Property 1", status: "Occupied", rent: 1500},
  {name: "Property 2", status: "Vacant", rent: 1200},
  {name: "Property 3", status: "Occupied", rent: 1800}
];
const tenants = [
  {name: "John Doe", property: "Property 1"},
  {name: "Jane Smith", property: "Property 3"}
];
const maintenanceRequests = [
  {title: "Leaky Faucet", property: "Property 1", status: "Pending"},
  {title: "Broken Heater", property: "Property 3", status: "In Progress"}
];
const applications = [
  {name: "Alice Brown", property: "Property 2", status: "Pending"}
];

// Populate KPI
document.getElementById("total-properties").querySelector(".value").innerText = properties.length;
document.getElementById("occupied-properties").querySelector(".value").innerText = properties.filter(p=>p.status==="Occupied").length;
document.getElementById("vacant-properties").querySelector(".value").innerText = properties.filter(p=>p.status==="Vacant").length;
document.getElementById("monthly-rent").querySelector(".value").innerText = "$" + properties.reduce((sum,p)=>sum+(p.status==="Occupied"?p.rent:0),0);

// Populate Lists
const maintenanceUl = document.getElementById("maintenance-items");
maintenanceRequests.forEach(req=>{
  const li = document.createElement("li"); li.innerText = `${req.title} (${req.property}) - ${req.status}`; li.style.cursor="pointer";
  li.addEventListener("click",()=>openModal(`<h2>${req.title}</h2><p>Property: ${req.property}</p><p>Status: ${req.status}</p>`));
  maintenanceUl.appendChild(li);
});
const tenantsUl = document.getElementById("tenant-items");
tenants.forEach(t=>{
  const li = document.createElement("li"); li.innerText = `${t.name} - ${t.property}`; li.style.cursor="pointer";
  li.addEventListener("click",()=>openModal(`<h2>${t.name}</h2><p>Property: ${t.property}</p>`));
  tenantsUl.appendChild(li);
});
const appsUl = document.getElementById("application-items");
applications.forEach(a=>{
  const li = document.createElement("li"); li.innerText = `${a.name} - ${a.property} (${a.status})`; li.style.cursor="pointer";
  li.addEventListener("click",()=>openModal(`<h2>${a.name}</h2><<p>Property: ${a.property}</p><p>Status: ${a.status}</p>`));
  appsUl.appendChild(li);
});

// Modal functions
const modalOverlay = document.getElementById("modal-overlay");
const modalContent = document.getElementById("modal-content");
function openModal(htmlContent){
  modalContent.innerHTML = htmlContent;
  modalOverlay.classList.remove("hidden");
}
document.querySelector(".close-modal").addEventListener("click",()=>{
  modalOverlay.classList.add("hidden");
});

// Charts
new Chart(document.getElementById('rentChart'), {
  type: 'line',
  data: {
    labels: ['Jan','Feb','Mar','Apr','May'],
    datasets: [{
      label: 'Rent Collected',
      data: [4500,4800,5100,5300,5000],
      borderColor: '#4CAF50',
      backgroundColor: 'rgba(76,175,80,0.2)',
      fill: true,
      tension: 0.4
    }]
  },
  options: {responsive:true, maintainAspectRatio:false}
});

new Chart(document.getElementById('expenseChart'), {
  type: 'bar',
  data: {
    labels:['Utilities','Repairs','Taxes','Misc'],
    datasets:[{
      label:'Expenses',
      data:[1200,800,1500,400],
      backgroundColor:['#FF6384','#36A2EB','#FFCE56','#9C27B0']
    }]
  },
  options: {responsive:true, maintainAspectRatio:false}
});

new Chart(document.getElementById('occupancyChart'), {
  type:'doughnut',
  data: {
    labels:['Occupied','Vacant'],
    datasets:[{
      data:[tenants.length, properties.length - tenants.length],
      backgroundColor:['#4CAF50','#F44336']
    }]
  },
  options: {responsive:true, maintainAspectRatio:false}
});
</script>
</body>
</html>
