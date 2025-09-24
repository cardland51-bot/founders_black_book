<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Red Mansion Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box;}
body{font-family:'Inter',sans-serif;background:linear-gradient(135deg,#4b0000,#220000);color:#fff;display:flex;min-height:100vh;}
.dashboard{display:flex;width:100%;flex:1;overflow:hidden;}

/* Sidebar */
.sidebar{width:220px;background:rgba(255,255,255,0.05);backdrop-filter:blur(12px);padding:2rem 1rem;display:flex;flex-direction:column;border-right:1px solid rgba(255,255,255,0.1);}
.logo{font-size:1.5rem;font-weight:700;margin-bottom:2rem;text-align:center;}
.sidebar nav a{display:block;color:#ddd;padding:0.8rem 1rem;margin:0.3rem 0;border-radius:12px;text-decoration:none;transition:0.3s;cursor:pointer;}
.sidebar nav a.active,.sidebar nav a:hover{background:rgba(255,255,255,0.2);color:#fff;}

/* Main */
.main{flex:1;padding:1.5rem;overflow-y:auto;display:flex;flex-direction:column;}
.header{margin-bottom:1rem;}
.header h1{font-size:2rem;font-weight:700;}
.header p{color:#ccc;}

/* KPI Cards */
.kpis{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:1rem;margin-bottom:1rem;max-height:220px;overflow-y:auto;}
.card{padding:1rem;border-radius:20px;text-align:center;cursor:pointer;display:flex;flex-direction:column;justify-content:center;transition:all 0.3s;}
.card:hover{transform:translateY(-5px);background:rgba(255,255,255,0.35);}
.glass{background:rgba(255,255,255,0.25);backdrop-filter:blur(14px);box-shadow:0 8px 32px rgba(0,0,0,0.2);border:1px solid rgba(255,255,255,0.3);}
.card h3,.card .value{color:#fff;}

/* Charts */
.charts{display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:1rem;margin-bottom:1rem;max-height:300px;overflow-y:auto;}
.chart-card{padding:1rem;border-radius:20px;display:flex;flex-direction:column;justify-content:center;}
.chart-card canvas{height:200px !important;}

/* Lists */
.lists{display:grid;grid-template-columns:repeat(auto-fit,minmax(300px,1fr));gap:1rem;max-height:300px;overflow-y:auto;}
.lists ul{list-style:none;max-height:200px;overflow-y:auto;padding-left:0;}
.lists li{padding:0.5rem;cursor:pointer;border-bottom:1px solid rgba(255,255,255,0.1);}
.lists li:hover{background:rgba(255,255,255,0.1);}

/* Forms */
.form-box{background:rgba(255,255,255,0.15);backdrop-filter:blur(12px);padding:1rem;border-radius:20px;margin-bottom:1rem;}
.form-box input, .form-box select, .form-box textarea, .form-box button{width:100%;margin-bottom:0.5rem;padding:0.5rem;border-radius:8px;border:none;}
.form-box button{background:#FF5733;color:#fff;font-weight:600;cursor:pointer;transition:0.3s;}
.form-box button:hover{background:#C70039;}

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
      <a id="forms-tab">Forms Inbox</a>
    </nav>
  </aside>
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
      <div id="maintenance-list" class="card glass"> <h3>Maintenance Requests</h3> <ul id="maintenance-items"></ul> <button onclick="exportCSV('maintenance')">Export CSV</button></div>
      <div id="tenant-list" class="card glass"> <h3>Tenants</h3> <ul id="tenant-items"></ul> <button onclick="exportCSV('tenants')">Export CSV</button></div>
      <div id="applications-list" class="card glass"> <h3>Pending Applications</h3> <ul id="application-items"></ul> <button onclick="exportCSV('applications')">Export CSV</button></div>
    </section>

    <!-- Submission Forms -->
    <section class="form-box">
      <h3>Submit Maintenance Request</h3>
      <input type="text" id="form-tenant-name" placeholder="Your Name">
      <input type="text" id="form-property" placeholder="Property Name">
      <textarea id="form-issue" placeholder="Describe Issue"></textarea>
      <button onclick="submitMaintenanceForm()">Submit Request</button>
    </section>
    <section class="form-box">
      <h3>Submit Rental Application</h3>
      <input type="text" id="applicant-name" placeholder="Your Name">
      <input type="text" id="app-property" placeholder="Property Interested">
      <button onclick="submitRentalApplication()">Submit Application</button>
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
// Data
let properties = [{name:"Red Mansion 1",status:"Occupied",rent:5000},{name:"Red Mansion 2",status:"Vacant",rent:4500}];
let tenants = [{name:"Alice Green",property:"Red Mansion 1"}];
let maintenanceRequests = [{title:"Pool Heater Issue",property:"Red Mansion 1",status:"Pending"}];
let applications = [{name:"Charlie Black",property:"Red Mansion 2",status:"Pending"}];
let submittedForms = [];

// Populate KPI
function updateKPI(){
  document.getElementById("total-properties").querySelector(".value").innerText = properties.length;
  document.getElementById("occupied-properties").querySelector(".value").innerText = properties.filter(p=>p.status==="Occupied").length;
  document.getElementById("vacant-properties").querySelector(".value").innerText = properties.filter(p=>p.status==="Vacant").length;
  document.getElementById("monthly-rent").querySelector(".value").innerText = "$"+properties.reduce((sum,p)=>sum+(p.status==="
  ==="Occupied"?p.rent:0),0);
}

// Populate Lists
function createListItem(text, htmlContent){
  const li = document.createElement("li");
  li.innerText = text;
  li.addEventListener("click",()=>openModal(htmlContent));
  return li;
}

function updateLists(){
  const MAX_DISPLAY = 10;
  const maintenanceUl = document.getElementById("maintenance-items");
  const tenantUl = document.getElementById("tenant-items");
  const appUl = document.getElementById("application-items");
  
  maintenanceUl.innerHTML = "";
  maintenanceRequests.slice(0,MAX_DISPLAY).forEach(m => maintenanceUl.appendChild(
    createListItem(`${m.title} - ${m.property} - ${m.status}`, `<h2>${m.title}</h2><p>Property: ${m.property}</p><p>Status: ${m.status}</p>`)
  ));
  
  tenantUl.innerHTML = "";
  tenants.slice(0,MAX_DISPLAY).forEach(t => tenantUl.appendChild(
    createListItem(`${t.name} - ${t.property}`, `<h2>${t.name}</h2><p>Property: ${t.property}</p>`)
  ));
  
  appUl.innerHTML = "";
  applications.slice(0,MAX_DISPLAY).forEach(a => appUl.appendChild(
    createListItem(`${a.name} - ${a.property} - ${a.status}`, `<h2>${a.name}</h2><p>Property: ${a.property}</p><p>Status: ${a.status}</p>`)
  ));
}

// Modal
const modalOverlay = document.getElementById("modal-overlay");
const modalContent = document.getElementById("modal-content");
function openModal(html){
  modalContent.innerHTML = html;
  modalOverlay.classList.remove("hidden");
}
document.querySelector(".close-modal").addEventListener("click",()=>modalOverlay.classList.add("hidden"));

// CSV Export
function exportCSV(type){
  let data = [];
  if(type==="properties") data = properties;
  if(type==="tenants") data = tenants;
  if(type==="maintenance") data = maintenanceRequests;
  if(type==="applications") data = applications;
  
  const csvContent = "data:text/csv;charset=utf-8," 
    + Object.keys(data[0]||{}).join(",") + "\n"
    + data.map(e => Object.values(e).join(",")).join("\n");
  
  const encodedUri = encodeURI(csvContent);
  const link = document.createElement("a");
  link.setAttribute("href", encodedUri);
  link.setAttribute("download", `${type}.csv`);
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}

// Submission Forms
function submitMaintenanceForm(){
  const name = document.getElementById("form-tenant-name").value;
  const property = document.getElementById("form-property").value;
  const issue = document.getElementById("form-issue").value;
  if(name && property && issue){
    maintenanceRequests.push({title:issue, property:property, status:"Pending"});
    submittedForms.push({type:"Maintenance", name, property, issue});
    updateLists();
    alert("Maintenance request submitted!");
  }
}

function submitRentalApplication(){
  const name = document.getElementById("applicant-name").value;
  const property = document.getElementById("app-property").value;
  if(name && property){
    applications.push({name, property, status:"Pending"});
    submittedForms.push({type:"Application", name, property});
    updateLists();
    alert("Rental application submitted!");
  }
}

// Charts
const rentChart = new Chart(document.getElementById('rentChart'), {
  type: 'line',
  data: {labels:['Jan','Feb','Mar','Apr','May'], datasets:[{label:'Rent Collected', data:[15000,16000,15500,16200,15800], borderColor:'#FF5733', backgroundColor:'rgba(255,87,51,0.2)', fill:true, tension:0.4}]},
  options:{responsive:true, maintainAspectRatio:false}
});
const expenseChart = new Chart(document.getElementById('expenseChart'), {
  type:'bar',
  data:{labels:['Repairs','Utilities','Staff','Misc'], datasets:[{label:'Expenses', data:[5000,2000,1500,800], backgroundColor:['#FFC300','#DAF7A6','#FF5733','#C70039']}]},
  options:{responsive:true, maintainAspectRatio:false}
});
const occupancyChart = new Chart(document.getElementById('occupancyChart'), {
  type:'doughnut',
  data:{labels:['Occupied','Vacant'], datasets:[{data:[tenants.length, properties.length-tenants.length], backgroundColor:['#4CAF50','#F44336']}]},
  options:{responsive:true, maintainAspectRatio:false}
});

// Sidebar tab modals
document.getElementById("properties-tab").addEventListener("click",()=>openModal(`<h2>Properties</h2>${properties.map(p=>`<p>${p.name} - ${p.status} - $${p.rent}</p>`).join('')}`));
document.getElementById("tenants-tab").addEventListener("click",()=>openModal(`<h2>Tenants</h2>${tenants.map(t=>`<p>${t.name} - ${t.property}</p>`).join('')}`));
document.getElementById("maintenance-tab").addEventListener("click",()=>openModal(`<h2>Maintenance Requests</h2>${maintenanceRequests.map(m=>`<p>${m.title} - ${m.property} - ${m.status}</p>`).join('')}`));
document.getElementById("applications-tab").addEventListener("click",()=>openModal(`<h2>Pending Applications</h2>${applications.map(a=>`<p>${a.name} - ${a.property} - ${a.status}</p>`).join('')}`));
document.getElementById("forms-tab").addEventListener("click",()=>openModal(`<h2>Forms Inbox</h2>${submittedForms.map(f=>`<p>${f.type}: ${f.name} - ${f.property}${f.issue?` - ${f.issue}`:''}</p>`).join('')}`));

// Initial population
updateKPI();
updateLists();
</script>
</body>
</html>
