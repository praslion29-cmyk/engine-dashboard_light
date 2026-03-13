<title>Airline Engine Health Monitoring – Status Berwarna</title>
<style>
body { font-family: Arial, sans-serif; padding: 20px; background-color: #f7f7f7; }
h1, h2 { color: #2c3e50; }
.cycle-input { margin-bottom: 15px; padding:10px; background:#fff; border-radius:5px; }
table { border-collapse: collapse; width: 100%; margin-top: 20px; background:#fff; }
th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
th { background-color: #2c3e50; color: #fff; }
.expandable-row { display: none; background:#eef; }
.expand-btn { cursor:pointer; color:blue; text-decoration:underline; }
.status-normal { color: green; font-weight: bold; }
.status-monitor { color: orange; font-weight: bold; }
.status-alert { color: red; font-weight: bold; }
</style>
</head>
<body>

<h1>Airline Engine Health Monitoring Status
 (status will be coloured if reach limit)</h1>

<h2>Flight Info</h2>
Tanggal: <input type="date" id="flightDate"><br>
Aircraft Registration: <input type="text" id="aircraftReg" placeholder="e.g. PK-GAB"><br>
Engine Number: <input type="text" id="engineNo" placeholder="e.g. ENG001"><br><br>

<label for="numCycles">Jumlah flight cycles:</label>
<input type="number" id="numCycles" min="1" max="10" value="3">
<button onclick="generateInputs()">Generate Input</button>

<div id="inputsContainer"></div>
<button onclick="analyzeCycles()">Analyze</button>

<h2>Engine Health Summary</h2>
<table id="summaryTable">
  <thead>
    <tr>
      <th>Cycle</th>
      <th>EGT (°C)</th>
      <th>N1 RPM</th>
      <th>N2 RPM</th>
      <th>Oil Pressure</th>
      <th>Oil Temp</th>
      <th>Vibration</th>
      <th>Fuel Flow (FFPH)</th>
      <th>Status</th>
      <th>Alerts</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
const LIMITS = {
  EGT: 950, N1: 5382, N2: 15183,
  OilPressureMin: 13, OilPressureMax: 60,
  OilTemp: 140, Vibration: 4.0,
  FuelFlow: 5160
};

function generateInputs(){
  const num = document.getElementById("numCycles").value;
  const container = document.getElementById("inputsContainer");
  container.innerHTML="";
  for(let i=1;i<=num;i++){
    const div=document.createElement("div");
    div.className="cycle-input";
    div.innerHTML=`<h3>Cycle ${i}</h3>
      EGT (°C): <input type="number" id="egt${i}" value="700"><br>
      N1 RPM: <input type="number" id="n1${i}" value="5200"><br>
      N2 RPM: <input type="number" id="n2${i}" value="15000"><br>
      Oil Pressure: <input type="number" id="op${i}" value="45"><br>
      Oil Temp (°C): <input type="number" id="ot${i}" value="140"><br>
      Vibration: <input type="number" id="vib${i}" step="0.1" value="3.5"><br>
      Fuel Flow (FFPH): <input type="number" id="ff${i}" value="5000">`;
    container.appendChild(div);
  }
}

function analyzeCycles(){
  const flightDate=document.getElementById("flightDate").value;
  const aircraftReg=document.getElementById("aircraftReg").value;
  const engineNo=document.getElementById("engineNo").value;

  const num=document.getElementById("numCycles").value;
  const tbody=document.querySelector("#summaryTable tbody");
  tbody.innerHTML="";

  for(let i=1;i<=num;i++){
    const egt=parseFloat(document.getElementById(`egt${i}`).value);
    const n1=parseFloat(document.getElementById(`n1${i}`).value);
    const n2=parseFloat(document.getElementById(`n2${i}`).value);
    const op=parseFloat(document.getElementById(`op${i}`).value);
    const ot=parseFloat(document.getElementById(`ot${i}`).value);
    const vib=parseFloat(document.getElementById(`vib${i}`).value);
    const ff=parseFloat(document.getElementById(`ff${i}`).value);

    let score=0,alerts=[];
    if(egt>LIMITS.EGT){score+=2;alerts.push("⚠️ EGT OVER LIMIT");} else if(egt>725) score+=1;
    if(n1>LIMITS.N1){score+=1;alerts.push("⚠️ N1 OVER LIMIT");}
    if(n2>LIMITS.N2){score+=1;alerts.push("⚠️ N2 OVER LIMIT");}
    if(op<LIMITS.OilPressureMin||op>LIMITS.OilPressureMax){score+=2;alerts.push("⚠️ Oil Pressure ALERT");} else if(op<40||op>50) score+=1;
    if(ot>LIMITS.OilTemp){score+=2;alerts.push("⚠️ Oil Temp ALERT");}
    if(vib>LIMITS.Vibration){score+=2;alerts.push("⚠️ Vibration ALERT");}
    if(ff>LIMITS.FuelFlow){score+=2;alerts.push("⚠️ Fuel Flow OVER LIMIT");}

    let statusClass="", statusText="";
    if(score==0){statusClass="status-normal"; statusText="Normal";}
    else if(score<=2){statusClass="status-monitor"; statusText="Monitor";}
    else {statusClass="status-alert"; statusText="Maintenance Required";}

    const row=`<tr>
      <td>${i}</td>
      <td>${egt}</td>
      <td>${n1}</td>
      <td>${n2}</td>
      <td>${op}</td>
      <td>${ot}</td>
      <td>${vib}</td>
      <td>${ff}</td>
      <td class="${statusClass}">${statusText}</td>
      <td>${alerts.join(", ")}</td>
    </tr>`;
    tbody.innerHTML+=row;
  }
}
</script>
</body>
</html>
