<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Red Mansion Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box;}
body{font-family:'Inter',sans-serif;background:linear-gradient(135deg,#4b0000,#220000);color:#fff;display:flex;height:100vh;}
.dashboard{display:flex;width:100%;}

/* Sidebar */
.sidebar{width:220px;background:rgba(255,255,255,0.05);backdrop-filter:blur(12px);padding:2rem 1rem;display:flex;flex-direction:column;border-right:1px solid rgba(255,255,255,0.1);}
.logo{font-size:1.5rem;font-weight:700;margin-bottom:2rem;text-align:center;}
.sidebar nav a{display:block;color:#ddd;padding:0.8rem 1rem;margin:0.3rem 0;border-radius:12px;text-decoration:none;transition:0.3s;cursor:pointer;}
.sidebar nav a.active,.sidebar nav a:hover{background:rgba(255,255,255,0.2);color:#fff;}

/* Main */
.main{flex:1;padding:2rem;overflow-y:auto;}
.header{margin-bottom:2rem;}
.header h1{font-size:2rem;font-weight:700;}
.header p{color:#ccc;}

/* KPI Cards */
.kpis{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:1.5rem;margin-bottom:2rem;}
.card{padding:1.5rem;border-radius:20px;text-align:center;cursor:pointer;transition:0.3s;}
.card:hover{transform:translateY(-5px);}
.glass{background:rgba(255,255,255,0.25);backdrop-filter:blur(14px);box-shadow:0 8px 32px rgba(0,0,0,0.2);border:1px solid rgba(255,255,255,0.3);}
.card h3,.card .value{color:#fff;}

/* Charts */
.charts{display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:1.5rem;margin-bottom:2rem;}
.chart-card{padding:1.5rem;border-radius:20px;}
.chart-card h3{margin-bottom:1rem;font-weight:600;}

/* Lists */
.lists{display:grid;grid-template-columns:repeat(auto-fit,minmax(300px,1fr));gap:1.5rem;margin-bottom:2rem;}
.lists ul{list-style:none;max-height:200px;overflow-y:auto;}

/* Modals */
.modal-overlay{position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.6);display:flex;justify-content:center;align-items:center;z-index:1000;}
.modal-overlay.hidden{display:none;}
.modal{background:rgba(255,255,255,0.15);backdrop-filter:blur(14px);padding:2rem;border-radius:20px;width:90%;max-width:900px;max-height:85%;overflow-y:auto;position:relative;}
.close-modal{position:absolute;top:1rem;right:1rem;font-size:1.5rem;background:none;border:none;color:#fff;cursor:pointer;}
</style>
</head>
<body>
<div class="dashboard">
  <aside class="sidebar">
    <h2 class="logo">🏰 Red Mansion</h2>
    <nav>
      <a class="active" id="overview-tab">Overview</a>
      <a id="properties-tab">Properties</a>
      <a id="tenants-tab">Tenants</a>
      <a id="maintenance-tab">Maintenance</a>
      <a id="applications-tab">Applications</a>
    </nav>
  </aside>
  <main class="main">
    <header class="header">
      <h1>Dashboard</h1>
      <p>Red Mansion Luxury Property Management</p>
    </header>
    <section class="kpis">
      <div id="total-properties" class="card glass"> <h3>Total Properties</h3> <p class="value">0</p> </div>
      <div id="occupied-properties" class="card glass"> <h3>Occupied</h3> <p class="value">0</p> </div>
      <div id="vacant-properties" class="card glass"> <h3>Vacant</h3> <p class="value">0</p> </div>
      <div id="monthly-rent" class="card glass"> <h3>Monthly Rent Collected</h3> <p class="value">$0</p> </div>
    </section>
    <section class="charts">
      <div class="chart-card glass"> <h3>Rent Collection</h3> <canvas id="rentChart"></canvas> </div>
      <div class="chart-card glass"> <h3>Expenses</h3> <canvas id="expenseChart"></canvas> </div>
      <div class="chart-card glass"> <h3>Occupancy Rate</h3> <canvas id="occupancyChart"></canvas> </div>
    </section>
    <section class="lists">
      <div id="maintenance-list" class="card glass"> <h3>Maintenance Requests</h3> <ul id="maintenance-items"></ul> </div>
      <div id="tenant-list" class="card glass"> <h3>Tenants</h3> <ul id="tenant-items"></ul> </div>
      <div id="applications-list" class="card glass"> <h3>Pending Applications</h3> <ul id="application-items"></ul> </div>
    </section>
  </main>
</div>

<div id="modal-overlay" class="modal-overlay hidden">
  <div class="modal glass">
    <button class="close-modal">&times;</button>
    <div id="modal-content"></div>
  </div>
</div>

<script>
// Sample Data
const properties = [
  {name:"Red Mansion 1", status:"Occupied", rent:5000},
  {name:"Red Mansion 2", status:"Vacant", rent:4500},
  {name:"Red Mansion 3", status:"Occupied", rent:6000}
];
const tenants = [
  {name:"Alice Green", property:"Red Mansion 1"},
  {name:"Bob White", property:"Red Mansion 3"}
];
const maintenanceRequests = [
  {title:"Pool Heater Issue", property:"Red Mansion 1", status:"Pending"},
  {title:"Roof Leak", property:"Red Mansion 3", status:"In Progress"}
];
const applications = [
  {name:"Charlie Black", property:"Red Mansion 2", status:"Pending"}
];

// Populate KPI
document.getElementById("total-properties").querySelector(".value").innerText=properties.length;
document.getElementById("occupied-properties").querySelector(".value").innerText=properties.filter(p=>p.status==="Occupied").length;
document.getElementById("vacant-properties").querySelector(".value").innerText=properties.filter(p=>p.status==="Vacant").length;
document.getElementById("monthly-rent").querySelector(".value").innerText="$"+properties.reduce((sum,p)=>sum+(p.status==="Occupied"?p.rent:0),0);

// Function to create clickable list items
function createListItem(text, htmlContent){
  const li=document.createElement("li"); li.innerText=text; li.style.cursor="pointer";
  li.addEventListener("click",()=>openModal(htmlContent));
  return li;
}

// Populate Lists
maintenanceRequests.forEach(m=>{
  document.getElementById("maintenance-items").appendChild(createListItem(
    `${m.title} - ${m.property} - ${m.status}`,
    `<h2>${m.title}</h2><p>Property: ${m.property}</p><p>Status: ${m.status}</p>`
  ));
});
tenants.forEach(t=>{
  document.getElementById("tenant-items").appendChild(createListItem(
    `${t.name} - ${t.property}`,
    `<h2>${t.name}</h2><p>Property: ${t.property}</p>`
  ));
});
applications.forEach(a=>{
  document.getElementById("application-items").appendChild(createListItem(
    `${a.name} - ${a.property} (${a.status})`,
    `<h2>${a.name}</h2><p>Property: ${a.property}</p><p>Status: ${a.status}</p>`
  ));
});

// Modal Functions
const modalOverlay=document.getElementById("modal-overlay");
const modalContent=document.getElementById("modal-content");
function openModal(html){ modalContent.innerHTML=html; modalOverlay.classList.remove("hidden"); }
document.querySelector(".close-modal").addEventListener("click",()=>modalOverlay.classList.add("hidden"));

// Charts
new Chart(document.getElementById('rentChart'),{
  type:'line', data:{labels:['Jan','Feb','Mar','Apr','May'],datasets:[{label:'Rent Collected',data:[15000,16000,15500,16200,15800],borderColor:'#FF5733',backgroundColor:'rgba(255,87,51,0.2)',fill:true,tension:0.4}]}, options:{responsive:true,maintainAspectRatio:false}
});
new Chart(document.getElementById('expenseChart'),{
  type:'bar', data:{labels:['Repairs','Utilities','Staff','Misc'],datasets:[{label:'Expenses',data:[5000,2000,1500,800],backgroundColor:['#FFC300','#DAF7A6','#FF5733','#C70039']}]}, options:{responsive:true,maintainAspectRatio:false}
});
new Chart(document.getElementById('occupancyChart'),{
  type:'doughnut', data:{labels:['Occupied','Vacant'],datasets:[{data:[tenants.length, properties.length-tenants.length],backgroundColor:['#4CAF50','#F44336']}]}, options:{responsive:true,maintainAspectRatio:false}
});

// Sidebar tab clicks to show modals
document.getElementById("properties-tab").addEventListener("click",()=>openModal(`<h2>Properties</h2>${properties.map(p=>`<p>${p.name} - ${p.status} - $${p.rent}</p>`).join('')}`));
document.getElementById("tenants-tab").addEventListener("click",()=>openModal(`<h2>Tenants</h2>${tenants.map(t=>`<p>${t.name} - ${t.property}</p>`).join('')}`));
document.getElementById("maintenance-tab").addEventListener("click",()=>openModal(`<h2>Maintenance Requests</h2>${maintenanceRequests.map(m=>`<p>${m.title} - ${m.property} - ${m.status}</p>`).join('')}`));
document.getElementById("applications-tab").addEventListener("click",()=>openModal(`<h2>Pending Applications</h2>${applications.map(a.map(a=>`<p>${a.name} - ${a.property} - ${a.status}</p>`).join('')}`));
</script>
</body>
</html>
