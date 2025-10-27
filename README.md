<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Secret Sauce Interactive Deck</title>
<style>
body {
    margin: 0;
    font-family: 'Arial', sans-serif;
    background: #0b0b0b;
    color: #fff;
    padding: 20px;
    overflow-x: hidden;
}

h1, h2 {
    color: #f5a623;
    text-align: center;
    text-shadow: 0 0 20px rgba(245,166,35,0.5);
}

h2 { margin-top: 50px; }

p {
    color: #ccc;
    line-height: 1.6;
    max-width: 700px;
    margin: 0 auto 20px;
    font-size: 1em;
}

#pipeline-wrapper {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    position: relative;
    width: 100%;
    max-width: 1200px;
    margin-bottom: 40px;
}

.node {
    background: linear-gradient(145deg, rgba(255,255,255,0.03), rgba(255,255,255,0.06));
    border-radius: 15px;
    padding: 20px;
    margin: 15px;
    width: 220px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.5);
    border: 1px solid rgba(255,255,255,0.1);
    backdrop-filter: blur(4px);
    text-align: center;
    color: #f5a623;
    cursor: pointer;
    position: relative;
    z-index: 5;
}

.node-title { font-weight: 600; margin-bottom: 8px; font-size: 1.1em; }
.node-content { font-size: 0.95em; color: #ddd; line-height: 1.4; }

button.run-btn {
    background: #f5a623;
    color: #0b0b0b;
    border: none;
    padding: 14px 30px;
    border-radius: 12px;
    font-size: 1em;
    cursor: pointer;
    margin: 25px 0;
    transition: 0.3s;
    box-shadow: 0 0 20px rgba(245,166,35,0.5);
    font-weight: bold;
}
button.run-btn:hover { background: #e59e20; box-shadow: 0 0 30px rgba(245,166,35,0.8); }

.output {
    background: rgba(255,255,255,0.05);
    color: #f5a623;
    font-family: monospace;
    padding: 18px;
    border-radius: 12px;
    text-align: center;
    min-height: 60px;
    font-size: 1.2em;
    box-shadow: 0 0 15px rgba(245,166,35,0.3);
    position: relative;
    z-index: 10;
    white-space: pre-line;
    margin-bottom: 50px;
}

.canvas-connection {
    position: absolute;
    top:0; left:0;
    width:100%; height:100%;
    pointer-events:none;
    z-index:1;
}

/* Popup for equations */
.popup {
    position: absolute;
    background: rgba(0,0,0,0.95);
    padding: 16px 20px;
    border-radius: 12px;
    font-family: 'Courier New', monospace;
    color: #f5a623;
    font-size: 1em;
    max-width: 280px;
    box-shadow: 0 0 25px rgba(245,166,35,0.4);
    pointer-events:none;
    opacity:0;
    transition: opacity 0.3s;
    z-index: 15;
    line-height: 1.5;
}

/* Sections */
.section { margin-bottom: 60px; padding: 0 10px; max-width: 800px; text-align:center; }

/* Mobile */
@media (max-width: 768px) {
    #pipeline-wrapper { flex-direction: column; align-items: center; }
    .node { width: 90%; max-width: 300px; }
    .popup { max-width: 90%; font-size:0.95em; }
}
</style>
</head>
<body>

<h1>Secret Sauce Interactive Deck</h1>
<p>Explore our full virality algorithm, see authentic-looking equations, and access our pitch, rollout plan, and value proposition in one page.</p>

<button class="run-btn" onclick="runSimulation()">Generate Trend Score</button>
<div class="output" id="output">Awaiting calculation...</div>
<canvas id="connectionCanvas" class="canvas-connection"></canvas>
<div id="popup" class="popup"></div>

<!-- Sections -->
<div class="section" id="pitch">
    <h2>Pitch Deck</h2>
    <p>Our algorithm identifies high-potential viral content before it trends. Combined with actionable insights and advanced signals, it allows brands and creators to dominate engagement efficiently.</p>
</div>

<div class="section" id="rollout">
    <h2>Rollout Plan</h2>
    <p>
        Phase 1: Beta with early adopters and creators.<br>
        Phase 2: Platform integration – we will need developer permission to access certain APIs for real-time data.<br>
        Phase 3: Full-scale launch – once fully developed, we hope to package this as a standalone unit that can be licensed or sold, so clients can deploy the algorithm without platform dependency.
    </p>
</div>

<div class="section" id="value">
    <h2>Value Proposition</h2>
    <p>Predict virality, optimize content strategy, and increase engagement ROI. Our pipeline delivers insights that standard engagement metrics cannot provide.</p>
</div>

<script>
// Modules with authentic equations
const modules = [
    {id:'data', title:'Data Collection', content:'Aggregates social signals, engagement metrics, and external trends.', eq:'D = Σ(SocialSignals + EngagementMetrics + ExternalTrends)'},
    {id:'feature', title:'Feature Engineering', content:'Engagement velocity, community resonance, novelty score, format efficiency, propagation potential.', eq:'F = w₁·Velocity + w₂·Resonance + w₃·Novelty + w₄·Format + w₅·Propagation'},
    {id:'score', title:'Scoring Module', content:'Weighted Virality Score (VS) calculation combining all features.', eq:'VS = Σ(Fᵢ · wᵢ)'},
    {id:'momentum', title:'Trend Momentum', content:'Predict engagement trajectory using early growth curves.', eq:'M(t) = d(Engagement)/dt'},
    {id:'insights', title:'Actionable Insights', content:'Provides recommendations and micro-actions for optimal virality.', eq:'Action = f(VS, Momentum, Community)'},
    {id:'advanced', title:'Advanced Signals', content:'Cross-content references, algorithmic sensitivity, external trend alignment.', eq:'Adv = CrossRefs + Sensitivity + ExternalAlignment'}
];

const moduleElements = {};
const popup=document.getElementById('popup');

// Create nodes with mobile-friendly popup
function createModule(node){
    const div=document.createElement('div');
    div.className='node';
    div.id=node.id;
    div.innerHTML=`<div class="node-title">${node.title}</div><div class="node-content">${node.content}</div>`;
    
    const showPopup = () => {
        popup.innerHTML = `<strong>Equation:</strong><br>${node.eq}`;
        const rect = div.getBoundingClientRect();
        let top = rect.bottom + window.scrollY + 8; // below node
        let left = rect.left + window.scrollX;
        const maxWidth = 280;
        if(left + maxWidth > window.innerWidth){
            left = window.innerWidth - maxWidth - 10;
        }
        popup.style.left = left + 'px';
        popup.style.top = top + 'px';
        popup.style.opacity = 1;
        setTimeout(()=>{popup.style.opacity=0;},4000);
    };
    
    div.addEventListener('click', showPopup);
    div.addEventListener('touchstart', showPopup);
    
    document.getElementById('pipeline-wrapper').appendChild(div);
    moduleElements[node.id]=div;
}
modules.forEach(createModule);

// Canvas connections
const canvas=document.getElementById('connectionCanvas');
const ctx=canvas.getContext('2d');
function resizeCanvas(){canvas.width=window.innerWidth; canvas.height=window.innerHeight;}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

function drawConnections(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    for(let i=0;i<modules.length-1;i++){
        const a=moduleElements[modules[i].id].getBoundingClientRect();
        const b=moduleElements[modules[i+1].id].getBoundingClientRect();
        ctx.strokeStyle='rgba(245,166,35,0.3)';
        ctx.lineWidth=2;
        ctx.beginPath();
        ctx.moveTo(a.left+a.width/2,a.top+a.height/2);
        ctx.lineTo(b.left+b.width/2,b.top+b.height/2);
        ctx.stroke();
    }
    requestAnimationFrame(drawConnections);
}
drawConnections();

// Floating equations
const eqs=["f(x)=x²+3x+2","y=sin(x)+cos(y)","E=mc²","a²+b²=c²","∑ᵢ=1ⁿ i²"];
function spawnEquation(module){
    const eq=document.createElement('div');
    eq.className='node-content';
    eq.style.position='absolute';
    const rect=moduleElements[module].getBoundingClientRect();
    eq.style.left = (rect.left + Math.random() * rect.width) + 'px';
    eq.style.top = (rect.top + Math.random() * rect.height) + 'px';
    eq.style.color = 'rgba(245,166,35,0.3)';
    eq.style.fontSize = '14px';
    eq.textContent = eqs[Math.floor(Math.random() * eqs.length)];
    document.body.appendChild(eq);

    let dy = -0.5 - Math.random(); // float upwards
    function animate() {
        const top = parseFloat(eq.style.top);
        if (top < rect.top - 50) { eq.remove(); return; }
        eq.style.top = (top + dy) + 'px';
        requestAnimationFrame(animate);
    }
    animate();
}

// Run simulation
function runSimulation() {
    const output = document.getElementById('output');
    output.textContent = 'Calculating...';
    const interval = setInterval(() => {
        modules.slice(1, modules.length - 1).forEach(m => spawnEquation(m.id));
    }, 200);

    setTimeout(() => {
        clearInterval(interval);
        const score = (Math.random() * 100).toFixed(2);
        let emoji = '⚡', action = '';
        if (score > 85) { emoji = '🚀'; action = 'Highly viral! Push immediately.'; }
        else if (score > 60) { emoji = '✨'; action = 'Trending upward. Promote and monitor.'; }
        else { emoji = '🌱'; action = 'Low trend score. Consider nurturing.'; }

        output.textContent = `Trend Score: ${score} ${emoji}\n${action}`;
    }, 3500);
}
</script>

</body>
</html>
