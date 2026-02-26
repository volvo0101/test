<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Crew Management System</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js"></script>

<style>
body { font-family: Arial; margin:40px; background:#f4f6f8; }
.card { background:white; padding:20px; margin-bottom:20px; border-radius:8px; }
button { padding:8px 14px; cursor:pointer; margin-top:5px; }
input, select, textarea { padding:6px; margin:5px 0; width:100%; }
table { width:100%; border-collapse: collapse; margin-top:10px; }
th, td { border:1px solid #ddd; padding:8px; vertical-align: top; }
th { background:#eee; }
a { text-decoration:none; color:blue; }
.dropdown { position:absolute; background:white; border:1px solid #ccc; max-height:200px; overflow-y:auto; display:none; z-index:1000; width:100%; }
</style>
</head>

<body>

<h2>⚓ Crew Management System</h2>

<!-- Add Seafarer -->
<div class="card">
<h3>Add Seafarer</h3>
<input id="internal_id" placeholder="Internal ID">
<input id="name" placeholder="Full Name">

<label>Position / Rank</label>
<select id="rank">
  <option value="">Select position</option>
  <option value="Master">Master</option>
  <option value="C/O">C/O</option>
  <option value="2/O">2/O</option>
  <option value="3/O">3/O</option>
  <option value="J/O">J/O</option>
  <option value="D/C">D/C</option>
  <option value="C/E">C/E</option>
  <option value="2/E">2/E</option>
  <option value="3/E">3/E</option>
  <option value="4/E">4/E</option>
  <option value="J/E">J/E</option>
  <option value="E/C">E/C</option>
  <option value="ETO">ETO</option>
  <option value="ETO assistance">ETO assistance</option>
  <option value="Pumpman">Pumpman</option>
  <option value="Bosun">Bosun</option>
  <option value="AB">AB</option>
  <option value="OS">OS</option>
  <option value="Oiler">Oiler</option>
  <option value="Wiper">Wiper</option>
  <option value="C/Cook">C/Cook</option>
  <option value="Messman">Messman</option>
  <option value="Fitter">Fitter</option>
  <option value="Painter">Painter</option>
</select>
<button onclick="addSeafarer()">Add</button>
</div>

<!-- Add Interview -->
<div class="card">
<h3>Add Interview</h3>
<div style="position:relative;">
  <input type="text" id="intSearch" placeholder="Type name or rank..." autocomplete="off">
  <div id="intDropdown" class="dropdown"></div>
</div>
<input type="hidden" id="intSeafarer">
<input type="date" id="intDate">
<select id="intResult">
  <option value="">Select decision</option>
  <option value="Approved">Approved</option>
  <option value="Standby">Standby</option>
  <option value="Rejected">Rejected</option>
</select>
<textarea id="intComments" rows="4" placeholder="Interview comments..."></textarea>
<button onclick="addInterview()">Add Interview</button>
</div>

<!-- Upload PDF/ZIP -->
<div class="card">
<h3>Upload Document</h3>
<input type="text" id="docSeafarerSearch" placeholder="Type name or rank..." autocomplete="off">
<input type="hidden" id="docSeafarer">
<div id="docSeafarerDropdown" class="dropdown"></div>

<select id="docType">
<option value="Certificate">Certificate</option>
<option value="Appraisal">Appraisal</option>
<option value="Training Book">Training Book</option>
<option value="Promotion">Promotion</option>
</select>
<input type="file" id="fileInput" accept=".pdf,.zip" multiple>
<button onclick="uploadDocument()">Upload</button>
</div>

<!-- Add Vessel -->
<div class="card">
<h3>Add Vessel</h3>
<input id="vesselName" placeholder="Vessel Name">
<input id="vesselAbbr" placeholder="Abbreviation">
<button onclick="addVessel()">Add Vessel</button>
</div>

<!-- Assign to Vessel -->
<div class="card">
<h3>Assign to Vessel</h3>
<input type="text" id="assignSeafarerSearch" placeholder="Type name or rank...">
<input type="hidden" id="assignSeafarer">
<div id="assignSeafarerDropdown" class="dropdown"></div>

<input type="text" id="assignVesselSearch" placeholder="Type vessel name...">
<input type="hidden" id="assignVessel">
<div id="assignVesselDropdown" class="dropdown"></div>

<label>Embarkation Date</label>
<input type="date" id="embarkDate">
<label>Disembarkation Date</label>
<input type="date" id="disembarkDate">
<button onclick="assignToVessel()">Save</button>
</div>

<!-- Crew List -->
<div class="card">
<h3>Crew List</h3>
<input type="text" id="crewSearchInput" placeholder="Search by name or rank..." style="width:300px;margin-bottom:10px;">
<button onclick="loadAll()">Refresh</button>
<table>
<thead>
<tr>
<th>Name</th>
<th>Rank</th>
<th>Status</th>
<th>Contract History</th>
<th>Interviews</th>
<th>Documents</th>
</tr>
</thead>
<tbody id="crewTable"></tbody>
</table>
</div>

<script>
const { createClient } = supabase
const client = createClient("чччч","чччч")
let allSeafarers = []

// ---------------- Helpers ----------------
function setupDropdown(inputId, hiddenId, dropdownId, data, fields){
  const input = document.getElementById(inputId)
  const hidden = hiddenId ? document.getElementById(hiddenId) : null
  const dropdown = dropdownId ? document.getElementById(dropdownId) : null

  input.addEventListener("input", ()=>{
    const val = input.value.toLowerCase()
    if(!dropdown) return
    dropdown.innerHTML = ""
    if(!val){ dropdown.style.display="none"; return }

    const filtered = data.filter(d=>fields.some(f=>d[f]?.toLowerCase().includes(val)))
    filtered.forEach(d=>{
      const item = document.createElement("div")
      item.style.padding="6px"; 
      item.style.cursor="pointer"; 
      item.style.borderBottom = "1px solid #eee"
      item.innerHTML = fields.map(f=>d[f]).join(" — ")
      item.onclick = ()=>{
        input.value = d.name || d[fields[0]]
        if(hidden) hidden.value = d.id
        dropdown.style.display="none"
      }
      dropdown.appendChild(item)
    })
    dropdown.style.display = filtered.length ? "block" : "none"
  })
}

// Подсчет стажа по должностям
function calculateServiceDays(allPositions) {
  const experience = {}

  allPositions.forEach(pos => {
    const position = pos.position || "Unknown"
    const start = new Date(pos.embarkation_date)
    const end = pos.disembarkation_date ? new Date(pos.disembarkation_date) : new Date()
    let totalDays = Math.floor((end - start) / (1000 * 60 * 60 * 24)) + 1

    const years = Math.floor(totalDays / 360)
    totalDays -= years * 360
    const months = Math.floor(totalDays / 30)
    totalDays -= months * 30
    const days = totalDays

    if(!experience[position]) experience[position] = {years:0, months:0, days:0}

    experience[position].years += years
    experience[position].months += months
    experience[position].days += days

    if(experience[position].days >= 30){
      experience[position].months += Math.floor(experience[position].days / 30)
      experience[position].days = experience[position].days % 30
    }
    if(experience[position].months >= 12){
      experience[position].years += Math.floor(experience[position].months / 12)
      experience[position].months = experience[position].months % 12
    }
  })

  return experience
}

function formatExperience(exp){
  return `${exp.years}y ${exp.months}m ${exp.days}d`
}

// Закрыть dropdown при клике вне его
document.addEventListener("click", e => {
  ["docSeafarerDropdown","intDropdown","assignSeafarerDropdown","assignVesselDropdown"].forEach(id=>{
    const dd = document.getElementById(id)
    if(dd && !e.target.closest(`#${id.replace("Dropdown","Search")}`)) dd.style.display="none"
  })
})

// ---------------- Seafarers ----------------
async function addSeafarer(){
  const name = document.getElementById("name").value
  const rank = document.getElementById("rank").value
  const internal_id = document.getElementById("internal_id").value
  if(!name || !rank) return alert("Fill all fields")
  const { error } = await client.from("seafarers").insert([{ name, rank, internal_id }])
  if(error) return alert(error.message)
  document.getElementById("name").value = ""
  document.getElementById("rank").value = ""
  document.getElementById("internal_id").value = ""
  loadAll()
}

function editRank(id, currentRank){
  const cell = document.getElementById("rank_text_" + id).parentElement
  const ranks = ["Master","C/O","2/O","3/O","J/O","D/C","C/E","2/E","3/E","4/E","J/E","E/C","ETO","ETO assistance","Pumpman","Bosun","AB","OS","Oiler","Wiper","C/Cook","Messman","Fitter","Painter"]
  let optionsHTML = ranks.map(r => `<option value="${r}" ${r===currentRank?'selected':''}>${r}</option>`).join("")
  cell.innerHTML = `<select id="rank_edit_${id}">${optionsHTML}</select>
    <button onclick="updateRank('${id}')">💾</button>`
}

// ---------------- Update Rank ----------------
async function updateRank(id){
  const input = document.getElementById("rank_edit_" + id)
  const newRank = input.value.trim()
  if(!newRank) return alert("Rank cannot be empty")

  const { data: seaServiceData } = await client.from("sea_service").select("*").eq("seafarer_id", id)
  const activeService = seaServiceData.find(s => !s.disembarkation_date || s.disembarkation_date === "")

  const today = new Date().toISOString().split("T")[0]

  if(activeService && activeService.position !== newRank){
    await client.from("sea_service").update({ disembarkation_date: today })
      .eq("id", activeService.id)

    await client.from("sea_service").insert([{
      seafarer_id: id,
      vessel_id: activeService.vessel_id || null,
      position: newRank,
      embarkation_date: today,
      disembarkation_date: null
    }])
  }

  await client.from("seafarers").update({ rank: newRank }).eq("id", id)
  loadAll()
}

// ---------------- Interviews ----------------
async function addInterview(){
  const seafarer_id = document.getElementById("intSeafarer").value
  const interview_date = document.getElementById("intDate").value
  const decision = document.getElementById("intResult").value
  const comment = document.getElementById("intComments").value
  if(!seafarer_id) return alert("Select seafarer")
  if(!interview_date) return alert("Select interview date")
  await client.from("interviews").insert([{ seafarer_id, interview_date, decision, comment }])
  document.getElementById("intComments").value = ""
  document.getElementById("intDate").value = ""
  document.getElementById("intSearch").value = ""
  document.getElementById("intSeafarer").value = ""
  loadAll()
}

// ---------------- Upload Documents ----------------
async function uploadDocument(){
  const seafarer_id = document.getElementById("docSeafarer").value
  const files = document.getElementById("fileInput").files
  const doc_type = document.getElementById("docType").value
  if(!seafarer_id || files.length === 0) return alert("Select seafarer and file")
  for(let file of files){
    const filePath = `${seafarer_id}/${Date.now()}_${file.name}`
    await client.storage.from("crew-documents").upload(filePath, file)
    const { data } = client.storage.from("crew-documents").getPublicUrl(filePath)
    await client.from("documents").insert([{ seafarer_id, file_name: file.name, file_url: data.publicUrl, doc_type }])
  }
  loadAll()
}

async function deleteDocument(id){
  if(!confirm("Delete document?")) return
  const { data: doc } = await client.from("documents").select("*").eq("id", id).single()
  if(doc?.file_url){
    const filePath = doc.file_url.split("/crew-documents/")[1]
    await client.storage.from("crew-documents").remove([filePath])
  }
  await client.from("documents").delete().eq("id", id)
  loadAll()
}

// ---------------- Vessels ----------------
async function addVessel(){
  const name = document.getElementById("vesselName").value
  const abbreviation = document.getElementById("vesselAbbr").value
  if(!name || !abbreviation) return alert("Fill all fields")
  await client.from("vessels").insert([{ name, abbreviation }])
  document.getElementById("vesselName").value = ""
  document.getElementById("vesselAbbr").value = ""
  loadAll()
}

async function assignToVessel(){
  const seafarer_id = document.getElementById("assignSeafarer").value
  const vessel_id = document.getElementById("assignVessel").value
  const embarkation_date = document.getElementById("embarkDate").value
  const disembarkation_date = document.getElementById("disembarkDate").value || null
  if(!seafarer_id || !vessel_id || !embarkation_date) return alert("All fields required")
  await client.from("sea_service").insert([{ seafarer_id, vessel_id, embarkation_date, disembarkation_date }])
  loadAll()
}

async function signOff(serviceId){
  const date = prompt("Enter sign off date (YYYY-MM-DD):")
  if(!date) return
  await client.from("sea_service").update({ disembarkation_date: date }).eq("id", serviceId)
  loadAll()
}

// ---------------- Load All ----------------
async function loadAll(){
  const { data: seafarers } = await client.from("seafarers").select("*")
  const { data: interviews } = await client.from("interviews").select("*")
  const { data: documents } = await client.from("documents").select("*")
  const { data: vessels } = await client.from("vessels").select("*")
  const { data: seaService } = await client.from("sea_service").select("*")

  allSeafarers = seafarers || []

  const searchValue = document.getElementById("crewSearchInput")?.value?.toLowerCase() || ""
  const table = document.getElementById("crewTable")
  table.innerHTML = ""

  for (let s of allSeafarers.filter(s => s.name.toLowerCase().includes(searchValue) || s.rank.toLowerCase().includes(searchValue))) {

    const allPositions = seaService?.filter(ss => ss.seafarer_id === s.id) || []
    const activeService = allPositions.find(ss => !ss.disembarkation_date || ss.disembarkation_date === "")
    const positionExperience = calculateServiceDays(allPositions)

    const currentRank = allPositions
      .filter(ss => ss.position && ss.position !== "Unknown")
      .sort((a,b) => new Date(b.embarkation_date) - new Date(a.embarkation_date))[0]?.position || "Unknown"

    const historyList = allPositions
      .sort((a,b) => new Date(b.embarkation_date) - new Date(a.embarkation_date))
      .map(ss => {
        const vessel = vessels?.find(v => v.id === ss.vessel_id)
        const signOffDate = ss.disembarkation_date ? ss.disembarkation_date : "Present"
        const posName = ss.position && ss.position !== "Unknown" ? ss.position : currentRank
        const expKey = (ss.position || posName).toLowerCase()
        const exp = positionExperience[expKey] ? formatExperience(positionExperience[expKey]) : "-"
        return `<div style="font-size:12px;background:#f1f3f6;padding:6px;margin-bottom:4px;border-radius:6px;">
          <b>${posName}</b> (${exp})<br>
          ${vessel?.name || "Unknown"}<br>
          ${ss.embarkation_date} → ${signOffDate}
        </div>`
      }).join("") || "-"

    const statusHTML = activeService 
      ? `<div style="color:green;font-weight:bold;">
          ON BOARD (${vessels?.find(v=>v.id===activeService.vessel_id)?.name || "Unknown"})
        </div>`
      : `<div style="color:red;font-weight:bold;">OFF SIGN</div>`

    const interviewsHTML = interviews?.filter(i => i.seafarer_id === s.id)
      .map(i => `<div style="font-size:12px;margin-bottom:2px;">
        ${i.interview_date} - ${i.decision} ${i.comment?`(${i.comment})`:''}
      </div>`).join("") || "-"

    const documentsHTML = documents?.filter(d => d.seafarer_id === s.id)
      .map(d => `<div style="font-size:12px;margin-bottom:2px;">
        <a href="${d.file_url}" target="_blank">${d.file_name}</a> [${d.doc_type}]
        <button onclick="deleteDocument(${d.id})" style="margin-left:5px;">🗑</button>
      </div>`).join("") || "-"

    const row = document.createElement("tr")
    row.innerHTML = `<td>${s.name}</td>
      <td><span id="rank_text_${s.id}">${activeService ? activeService.position : s.rank}</span>
          <button onclick="editRank('${s.id}','${activeService ? activeService.position : s.rank}')">✏</button></td>
      <td>${statusHTML}</td>
      <td>${historyList}</td>
      <td>${interviewsHTML}</td>
      <td>${documentsHTML}</td>`
    table.appendChild(row)
  }
}

loadAll()
</script>
</body>
</html>
