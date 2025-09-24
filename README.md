<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Luxury Property Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
/* General */
body { margin:0; font-family:'Inter',sans-serif; background:#1f1c2c; color:#fff; display:flex; min-height:100vh; overflow:hidden; }
.dashboard { display:flex; flex:1; width:100%; overflow:hidden; }

/* Sidebar */
.sidebar { width:220px; background:rgba(0,0,0,0.6); backdrop-filter:blur(4px); padding:2rem 1rem; display:flex; flex-direction:column; border-right:1px solid rgba(255,255,255,0.1); }
.logo { font-size:1.5rem; font-weight:700; margin-bottom:2rem; text-align:center; color:#FFD700; }
.sidebar nav a { display:block; color:#ddd; padding:0.8rem 1rem; margin:0.3rem 0; border-radius:12px; text-decoration:none; transition:0.3s; cursor:pointer; }
.sidebar nav a.active, .sidebar nav a:hover { background:rgba(255,255,255,0.2); color:#fff; }

/* Main */
.main { flex:1; padding:1rem; overflow-y:auto; display:flex; flex-direction:column; }
.header { margin-bottom:1rem; }
.header h1 { font-size:2rem; font-weight:700; }
.header p { color:#ccc; }

/* KPI Cards */
.kpis { display:grid; grid-template-columns:repeat(auto-fit,minmax(180px,1fr)); gap:1rem; margin-bottom:1rem; }
.card { padding:1rem; border-radius:20px; text-align:center; display:flex; flex-direction:column; justify-content:center; transition:all 0.3s; background:rgba(255,255,255,0.1); backdrop-filter:blur(2px); box-shadow:0 2px 8px rgba(0,0,0,0.2); border:1px solid rgba(255,255,255,0.1); cursor:pointer; }
.card:hover { transform:translateY(-3px); background:rgba(255,255,255,0.2); }

/* Charts */
.charts { display:grid; grid-template-columns:repeat(auto-fit,minmax(280px,1fr)); gap:1rem; margin-bottom:1rem; }
.chart-card { padding:1rem; border-radius:20px; display:flex; flex-direction:column; justify-content:center; background:rgba(255,255,255,0.1); backdrop-filter:blur(2px); }

/* Lists */
.lists { display:grid; grid-template-columns:repeat(auto-fit,minmax(280px,1fr)); gap:1rem; margin-bottom:1rem; }
.list-box { background:rgba(255,255,255,0.1); backdrop-filter:blur(2px); padding:1rem; border-radius:20px; max-height:250px; overflow-y:auto; }
.list-box h3 { margin-top:0; display:flex; justify-content:space-between; align-items:center; }
.list-box ul { list-style:none; padding-left:0; margin:0; }
.list-box li { padding:0.5rem; border-bottom:1px solid rgba(255,255,255,0.05); cursor:pointer; }
.list-box li:hover { background:rgba(255,255,255,0.1); }

/* Forms */
.form-box { background:rgba(255,255,255,0.1); backdrop-filter:blur(2px); padding:1rem; border-radius:20px; margin-bottom:1rem; }
.form-box input, .form-box select, .form-box textarea, .form-box button { width:100%; margin-bottom:0.5rem; padding:0.5rem; border-radius:8px; border:none; }
.form-box button { background:#FFD700; color:#000; font-weight:600; cursor:pointer; transition:0.3s; }
.form-box button:hover { background:#FFC300; }
.message { color:#FFD700; margin-bottom:0.5rem; }

/* Modal */
.modal-overlay { position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.6); display:flex; justify-content:center; align-items:center; z-index:1000; }
.modal-overlay.hidden { display:none; }
.modal { background:rgba(255,255,255,0.1); backdrop-filter:blur(4px); padding:2rem; border-radius:20px; width:90%; max-width:900px; max-height:85%; overflow-y:auto; position:relative; }
.close-modal { position:absolute; top:1rem; right:1rem; font-size:1.5rem; background:none; border:none; color:#fff; cursor:pointer; }
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

<section class="kpis">
<div id="total-properties" class="card"><h3>Total Properties</h3><p class="value">0</p></div>
<div id="occupied-properties" class="card"><h3>Occupied</h3><p class="value">0</p></div>
<div id="vacant-properties" class="card"><h3>Vacant</h3><p class="value">0</p></div>
<div id="monthly-rent" class="card"><h3>Monthly Rent Collected</h3><p class="value">$0</p></div>
</section>

<section class="charts">
<div class="chart-card"><h3>Rent Collection</h3><canvas id="rentChart"></canvas></div>
<div class="chart-card"><h3>Expenses</h3><canvas id="expenseChart"></canvas></div>
<div class="chart-card"><h3>Occupancy Rate</h3><canvas id="occupancyChart"></canvas></div>
</section>

<section class="lists">
<div class="list-box">
<h3>Maintenance Requests <button onclick="exportCSV('maintenance')">Export CSV</button></h3>
<ul id="maintenance-items"></ul>
</div>
<div class="list-box">
<h3>Tenants <button onclick="exportCSV('tenants')">Export CSV</button></h3>
<ul id="tenant-items"></ul>
</div>
<div class="list-box">
<h3>Pending Applications <button onclick="exportCSV('applications')">Export CSV</button></h3>
<ul id="application-items"></ul>
</div>
</section>

<section class="form-box">
<h3>Add Property</h3>
<p class="message" id="property-message"></p>
<input type="text" id="new-property-name" placeholder="Property Name">
<select id="new-property-status">
<option value="Occupied">Occupied</option>
<option value="Vacant">Vacant</option>
</select>
<input type="number" id="new-property-rent" placeholder="Monthly Rent">
<button onclick="addProperty()">Add Property</button>
</section>

<section class="form-box">
<h3>Submit Maintenance Request</h3>
<p class="message" id="maintenance-message"></p>
<input type="text" id="form-tenant-name" placeholder="Your Name">
<input type="text" id="form-property" placeholder="Property Name">
<textarea id="form-issue" placeholder="Describe Issue"></textarea>
<button onclick="submitMaintenanceForm()">Submit Request</button>
</section>

<section class="form-box">
<h3>Submit Rental Application</h3>
<p class="message" id="application-message"></p>
<input type="text" id="applicant-name" placeholder="Your Name">
<input type="text" id="app-property" placeholder="Property Interested">
<button onclick="submitRentalApplication()">Submit Application</button>
</section>

</main>
</div>

<div id="modal-overlay" class="modal-overlay hidden">
<div class="modal">
<button class="close-modal">&times;</button>
<div id="modal-content"></div>
</div>
</div>

<script>
// Sample Data
let properties = [
{name:"Red Mansion 1",status:"Occupied",rent:5000},
{name:"Red Mansion 2",status:"Vacant",rent:4500},
{name:"Red Mansion 3",status:"Occupied",rent:6000},
{name:"Red Mansion 4",status:"Vacant",rent:4800},
{name:"Red Mansion 5",status:"Occupied",rent:5200},
{name:"Red Mansion 6",status:"Vacant",rent:4700},
{name:"Red Mansion 7",status:"Occupied",rent:5500},
{name:"Red Mansion 8",status:"Vacant",rent:4600}
];
let tenants = [
{name:"Alice Green",property:"Red Mansion 1"},
{name:"Bob White",property:"Red Mansion 3"},
{name:"Carol Blue",property:"Red Mansion 5"},
{name:"David Gray",property:"Red Mansion 7"}
];
let maintenanceRequests = [
{title:"Pool Heater Issue",property:"Red Mansion 1",status:"Pending"},
{title:"Roof Leak",property:"Red Mansion 3",status:"In Progress"},
{title:"AC Repair",property:"Red Mansion 5",status:"Completed"}
];
let applications = [
{name:"Charlie Black",property:"Red Mansion 2",status:"Pending"},
{name:"Eve Orange",property:"Red Mansion 4",status:"Pending"}
];
let submittedForms = [];

const MAX_DISPLAY = 10;

// Update KPI
function updateKPI(){
document.getElementById("total-properties").querySelector(".value").innerText = properties.length;
document.getElementById("occupied-properties").querySelector(".value").innerText = properties.filter(p=>p.status==="Occupied").length;
document.getElementById("vacant-properties").querySelector(".value").innerText = properties.filter(p=>p.status==="Vacant").length;
document.getElementById("monthly-rent").querySelector(".value").innerText = "$"+properties.reduce((sum,p)=>sum+(p.status==="Occupied"?p.rent:0),0);
}

// Create list item
function createListItem(text, htmlContent){
const li = document.createElement("li");
li.innerText = text;
li.addEventListener("click",()=>openModal(htmlContent));
return li;
}

// Update Lists
function updateLists(){
const maintenanceUl = document.getElementById("maintenance-items");
const tenantUl = document.getElementById("tenant-items");
const appUl = document.getElementById("application-items");
[maintenanceUl, tenantUl, appUl].forEach(ul=>ul.innerHTML="");

maintenanceRequests.slice(0,MAX_DISPLAY).forEach(m=>maintenanceUl.appendChild(
createListItem(`${m.title} - ${m.property} - ${m.status}`, `<h2>${m.title}</h2><p>Property: ${m.property}</p><p>Status: ${m.status}</p>`)
));

tenants.slice(0,MAX_DISPLAY).forEach(t=>tenantUl.appendChild(
createListItem(`${t.name} - ${t.property}`, `<h2>${t.name}</h2><p>Property: ${t.property}</p>`)
));

applications.slice(0,MAX_DISPLAY).forEach(a=>appUl.appendChild(
createListItem(`${a.name} - ${a.property} - ${a.status}`, `<h2>${a.name}</h2><p>Property: ${a.property}</p><p>Status: ${a.status}</p>`)
));
}

// Modal
const modalOverlay = document.getElementById("modal-overlay");
const modalContent = document.getElementById("modal-content");
function openModal(html){modalContent.innerHTML=html; modalOverlay.classList.remove("hidden");}
document.querySelector(".close-modal").addEventListener("click",()=>modalOverlay.classList.add("hidden"));

// CSV Export
function exportCSV(type){
let data = [];
if(type==="properties") data = properties;
if(type==="tenants") data = tenants;
if(type==="maintenance") data = maintenanceRequests;
if(type==="applications") data = applications;
if(!data.length) return alert("No data to export");
const csvContent = "data:text/csv;charset=utf-8,"+Object.keys(data[0]).join(",")+"\n"+data.map(e=>Object.values(e).join(",")).join("\n");
const encodedUri = encodeURI(csvContent);
const link = document.createElement("a");
link.setAttribute("href", encodedUri);
link.setAttribute("download", `${type}.csv`);
document.body.appendChild(link);
link.click();
document.body.removeChild(link);
}

// Add Property
function addProperty(){
const name = document.getElementById("new-property-name").value;
const status = document.getElementById("new-property-status").value;
const rent = parseFloat(document.getElementById("new-property-rent").value);
const message = document.getElementById("property-message");
if(name && !isNaN(rent)){
properties.push({name,status,rent});
updateKPI(); updateLists();
message.innerText = "Property added!";
setTimeout(()=>message.innerText="",2000);
document.getElementById("new-property-name").value="";
document.getElementById("new-property-rent").value="";
}
}

// Submit Maintenance Form
function submitMaintenanceForm(){
const name = document.getElementById("form-tenant-name").value;
const property = document.getElementById("form-property").value;
const issue = document.getElementById("form-issue").value;
const message = document.getElementById("maintenance-message");
if(name && property && issue){
maintenanceRequests.push({title:issue,property,status:"Pending"});
submittedForms.push({type:"Maintenance",name,property,issue});
updateLists();
message.innerText = "Maintenance request submitted!";
setTimeout(()=>message.innerText="",2000);
document.getElementById("form-tenant-name").value="";
document.getElementById("form-property").value="";
document.getElementById("form-issue").value="";
}
}

// Submit Rental Application
function submitRentalApplication(){
const name = document.getElementById("applicant-name").value;
const property = document.getElementById("app-property").value;
const message = document.getElementById("application-message");
if(name && property){
applications.push({name, property, status:"Pending"});
submittedForms.push({type:"Application",name,property});
updateLists();
message.innerText = "Rental application submitted!";
setTimeout(()=>message.innerText="",2000);
document.getElementById("applicant-name").value="";
document.getElementById("app-property").value="";
}

// Charts
let rentChart, expenseChart, occupancyChart;
function loadCharts(){
if(rentChart) return; // Already loaded
rentChart = new Chart(document.getElementById('rentChart'), { type:'line', data:{ labels:properties.map(p=>p.name), datasets:[{ label:'Rent Collected', data: properties.map(p=>p.status==="Occupied"?p.rent:0), borderColor:'#FFD700', backgroundColor:'rgba(255,215,0,0.2)', fill:true, tension:0.4 }] }, options:{responsive:true, maintainAspectRatio:false} });
expenseChart = new Chart(document.getElementById('expenseChart'), { type:'bar', data:{ labels:['Repairs','Utilities','Staff','Misc'], datasets:[{ label:'Expenses', data:[2000,1000,1500,500], backgroundColor:['#FFC300','#DAF7A6','#FF5733','#C70039'] }] }, options:{responsive:true, maintainAspectRatio:false} });
occupancyChart = new Chart(document.getElementById('occupancyChart'), { type:'doughnut', data:{ labels:['Occupied','Vacant'], datasets:[{ data:[properties.filter(p=>p.status==="Occupied").length, properties.filter(p=>p.status==="Vacant").length], backgroundColor:['#4CAF50','#F44336'] }] }, options:{responsive:true, maintainAspectRatio:false} });
}

// Tab Navigation
const tabs = document.querySelectorAll(".sidebar nav a");
tabs.forEach(tab=>{
tab.addEventListener("click",()=>{
tabs.forEach(t=>t.classList.remove("active"));
tab.classList.add("active");
if(tab.id==='overview-tab'){ loadCharts(); }
});
});

// Initialize
updateKPI();
updateLists();
</script>

</body>
</html>
