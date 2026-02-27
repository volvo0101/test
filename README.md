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
<label>Your Telegram</label>
<input type="text" id="interviewBy" placeholder="@username">
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
<select id="assignPosition">
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

<div class="card">
  <h3>Add Office Comments</h3>

  <div style="position:relative;">
    <input type="text" id="commentSeafarerSearch" placeholder="Type name or rank..." autocomplete="off">
    <input type="hidden" id="commentSeafarer">
    <div id="commentSeafarerDropdown" class="dropdown"></div>
  </div>

  <label>Your Telegram</label>
  <input type="text" id="commentBy" placeholder="@username">

  <label>QA Comment</label>
  <textarea id="qaComment" rows="2" placeholder="Comment for QA..."></textarea>

  <label>TSI Comment</label>
  <textarea id="tsiComment" rows="2" placeholder="Comment for TSI..."></textarea>

  <label>OPS Comment</label>
  <textarea id="opsComment" rows="2" placeholder="Comment for OPS..."></textarea>

  <button onclick="addOfficeComments()">Add Comments</button>
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
<th>
          Rank 
          <button onclick="toggleDropdown('rankFilterDropdown')">▼</button>
          <div id="rankFilterDropdown" class="dropdown" style="display:none;">
            <label><input type="checkbox" class="filterPosition" value="Master" checked onchange="loadAll()"> Master</label>
            <label><input type="checkbox" class="filterPosition" value="C/O" checked onchange="loadAll()"> C/O</label>
            <label><input type="checkbox" class="filterPosition" value="2/O" checked onchange="loadAll()"> 2/O</label>
            <label><input type="checkbox" class="filterPosition" value="3/O" checked onchange="loadAll()"> 3/O</label>
            <label><input type="checkbox" class="filterPosition" value="J/O" checked onchange="loadAll()"> J/O</label>
            <label><input type="checkbox" class="filterPosition" value="C/E" checked onchange="loadAll()"> C/E</label>
            <label><input type="checkbox" class="filterPosition" value="2/E" checked onchange="loadAll()"> 2/E</label>
            <label><input type="checkbox" class="filterPosition" value="3/E" checked onchange="loadAll()"> 3/E</label>
            <label><input type="checkbox" class="filterPosition" value="4/E" checked onchange="loadAll()"> 4/E</label>
            <label><input type="checkbox" class="filterPosition" value="J/E" checked onchange="loadAll()"> J/E</label>
            <label><input type="checkbox" class="filterPosition" value="E/C" checked onchange="loadAll()"> E/C</label>
            <label><input type="checkbox" class="filterPosition" value="ETO" checked onchange="loadAll()"> ETO</label>
            <label><input type="checkbox" class="filterPosition" value="ETO assistance" checked onchange="loadAll()"> ETO assistance</label>
            <label><input type="checkbox" class="filterPosition" value="Pumpman" checked onchange="loadAll()"> Pumpman</label>
            <label><input type="checkbox" class="filterPosition" value="Bosun" checked onchange="loadAll()"> Bosun</label>
            <label><input type="checkbox" class="filterPosition" value="AB" checked onchange="loadAll()"> AB</label>
            <label><input type="checkbox" class="filterPosition" value="OS" checked onchange="loadAll()"> OS</label>
            <label><input type="checkbox" class="filterPosition" value="Oiler" checked onchange="loadAll()"> Oiler</label>
            <label><input type="checkbox" class="filterPosition" value="Wiper" checked onchange="loadAll()"> Wiper</label>
            <label><input type="checkbox" class="filterPosition" value="C/Cook" checked onchange="loadAll()"> C/Cook</label>
            <label><input type="checkbox" class="filterPosition" value="Messman" checked onchange="loadAll()"> Messman</label>
            <label><input type="checkbox" class="filterPosition" value="Fitter" checked onchange="loadAll()"> Fitter</label>
            <label><input type="checkbox" class="filterPosition" value="Painter" checked onchange="loadAll()"> Painter</label>
            </div>
        </th>
        <th>
          Status
          <button onclick="toggleDropdown('statusFilterDropdown')">▼</button>
          <div id="statusFilterDropdown" class="dropdown" style="display:none;">
            <label><input type="checkbox" id="filterOnboard" checked onchange="loadAll()"> Onboard</label>
            <label><input type="checkbox" id="filterAshore" checked onchange="loadAll()"> Ashore</label>
          </div>
        </th>
<th>Contract History</th>
<th>Interviews</th>
<th>QA</th>
<th>TSI</th>
<th>OPS</th>
<th>Documents</th>
</tr>
</thead>
<tbody id="crewTable"></tbody>
</table>
</div>

<script>
const { createClient } = supabase
const client = createClient("https://kjtigzaevodgpdtndyqs.supabase.co",
"sb_publishable_qZEENkcQYkmw4oxJP3Lekw_pRerDtsE")
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
  const experience = {} // ключ = должность

  allPositions.forEach(pos => {
    const position = pos.position
    if(!position || position === "Unknown") return // пропускаем пустые

    const start = new Date(pos.embarkation_date)
    const end = pos.disembarkation_date ? new Date(pos.disembarkation_date) : new Date()
    let totalDays = Math.floor((end - start) / (1000*60*60*24)) + 1

    // инициализация перед суммированием
    function toggleDropdown(id){
  const dd = document.getElementById(id)
  dd.style.display = dd.style.display === "block" ? "none" : "block"
}

// закрываем dropdown при клике вне области
document.addEventListener("click", e=>{
  ["rankFilterDropdown","statusFilterDropdown"].forEach(id=>{
    const dd = document.getElementById(id)
    if(dd && !e.target.closest(`#${id}`) && !e.target.closest("button")) dd.style.display="none"
  })
})
    
    // инициализация перед суммированием
    if(!experience[position]) experience[position] = {years:0, months:0, days:0}

    experience[position].years += Math.floor(totalDays / 360)
    totalDays = totalDays % 360

    experience[position].months += Math.floor(totalDays / 30)
    totalDays = totalDays % 30

    experience[position].days += totalDays
  })

  // корректируем переполнение
  Object.keys(experience).forEach(pos => {
    if(experience[pos].days >= 30){
      experience[pos].months += Math.floor(experience[pos].days / 30)
      experience[pos].days = experience[pos].days % 30
    }
    if(experience[pos].months >= 12){
      experience[pos].years += Math.floor(experience[pos].months / 12)
      experience[pos].months = experience[pos].months % 12
    }
  })

  return experience
}

function formatExperience(exp){
  return `${exp.years}y ${exp.months}m ${exp.days}d`
}
document.addEventListener("click", e=>{
  ["docSeafarerDropdown","intDropdown","assignSeafarerDropdown","assignVesselDropdown"].forEach(id=>{
    const dd = document.getElementById(id)
    if(dd && !e.target.closest(`#${id.replace("Dropdown","Search")}`)) dd.style.display="none"
  })
})

// ---------------- Seafarers ----------------
function generateInternalID() {
  const now = new Date()
  const datePart = now.getFullYear().toString().slice(-2)  // последние 2 цифры года
                + String(now.getMonth()+1).padStart(2,'0') // месяц
                + String(now.getDate()).padStart(2,'0')   // день
  const randomPart = Math.floor(1000 + Math.random() * 9000) // случайное 4-значное число
  return `ID${datePart}${randomPart}` // пример: ID26021234
}
  
async function addSeafarer(){
  const name = document.getElementById("name").value
  const rank = document.getElementById("rank").value
  if(!name || !rank) return alert("Fill all fields")

  const internal_id = generateInternalID() // генерируем автоматически

  const { error } = await client.from("seafarers").insert([{ name, rank, internal_id }])
  if(error) return alert(error.message)

  document.getElementById("name").value = ""
  document.getElementById("rank").value = ""
  loadAll()
}

function editRank(id, currentRank){
  const cell = document.getElementById("rank_text_" + id).parentElement
  cell.innerHTML = `<select id="rank_edit_${id}">
      <option ${currentRank=="Master"?"selected":""}>Master</option>
      <option ${currentRank=="C/O"?"selected":""}>C/O</option>
      <option ${currentRank=="2/O"?"selected":""}>2/O</option>
      <option ${currentRank=="3/O"?"selected":""}>3/O</option>
      <option ${currentRank=="J/O"?"selected":""}>J/O</option>
      <option ${currentRank=="D/C"?"selected":""}>D/C</option>
      <option ${currentRank=="C/E"?"selected":""}>C/E</option>
      <option ${currentRank=="2/E"?"selected":""}>2/E</option>
      <option ${currentRank=="3/E"?"selected":""}>3/E</option>
      <option ${currentRank=="4/E"?"selected":""}>4/E</option>
      <option ${currentRank=="J/E"?"selected":""}>J/E</option>
      <option ${currentRank=="E/C"?"selected":""}>E/C</option>
      <option ${currentRank=="ETO"?"selected":""}>ETO</option>
      <option ${currentRank=="ETO assistance"?"selected":""}>ETO assistance</option>
      <option ${currentRank=="Pumpman"?"selected":""}>Pumpman</option>
      <option ${currentRank=="Bosun"?"selected":""}>Bosun</option>
      <option ${currentRank=="AB"?"selected":""}>AB</option>
      <option ${currentRank=="OS"?"selected":""}>OS</option>
      <option ${currentRank=="Oiler"?"selected":""}>Oiler</option>
      <option ${currentRank=="Wiper"?"selected":""}>Wiper</option>
      <option ${currentRank=="C/Cook"?"selected":""}>C/Cook</option>
      <option ${currentRank=="Messman"?"selected":""}>Messman</option>
      <option ${currentRank=="Fitter"?"selected":""}>Fitter</option>
      <option ${currentRank=="Painter"?"selected":""}>Painter</option>
    </select>
    <button onclick="updateRank('${id}')">💾</button>`
}

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

async function addOfficeComments(){
  const seafarer_id = document.getElementById("commentSeafarer").value
  const created_by = document.getElementById("commentBy").value.trim() || "Unknown"
  if(!seafarer_id) return alert("Select a seafarer")

  const qa = document.getElementById("qaComment").value.trim()
  const tsi = document.getElementById("tsiComment").value.trim()
  const ops = document.getElementById("opsComment").value.trim()

  const inserts = []

  if(qa) inserts.push({ seafarer_id, department: "QA", comment: qa, created_by })
  if(tsi) inserts.push({ seafarer_id, department: "TSI", comment: tsi, created_by })
  if(ops) inserts.push({ seafarer_id, department: "OPS", comment: ops, created_by })

  if(inserts.length === 0) return alert("Enter at least one comment")

  await client.from("office_comments").insert(inserts)

  // Очистка полей
  document.getElementById("qaComment").value = ""
  document.getElementById("tsiComment").value = ""
  document.getElementById("opsComment").value = ""
  document.getElementById("commentSeafarerSearch").value = ""
  document.getElementById("commentSeafarer").value = ""
  document.getElementById("commentBy").value = ""

  loadAll()
}

// ---------------- Interviews ----------------
async function addInterview(){
  const seafarer_id = document.getElementById("intSeafarer").value
  const interview_date = document.getElementById("intDate").value
  const decision = document.getElementById("intResult").value
  const comment = document.getElementById("intComments").value
  if(!seafarer_id || !interview_date) return alert("Select seafarer and date")
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
  if(!seafarer_id || files.length===0) return alert("Select seafarer and file")
  for(let file of files){
    const filePath = `${seafarer_id}/${Date.now()}_${file.name}`
    await client.storage.from("crew-documents").upload(filePath, file)
    const { data } = client.storage.from("crew-documents").getPublicUrl(filePath)
    await client.from("documents").insert([{ seafarer_id, file_name:file.name, file_url:data.publicUrl, doc_type }])
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
  const position = document.getElementById("assignPosition").value
  const vessel_id = document.getElementById("assignVessel").value
  const embarkation_date = document.getElementById("embarkDate").value
  const disembarkation_date = document.getElementById("disembarkDate").value || null

  if(!seafarer_id || !vessel_id || !embarkation_date)
    return alert("All fields required")

  // Получаем текущий ранг моряка
  const { data: seafarer } = await client
    .from("seafarers")
    .select("rank")
    .eq("id", seafarer_id)
    .single()

  await client.from("sea_service").insert([{
  seafarer_id,
  vessel_id,
  position,
  embarkation_date,
  disembarkation_date
}])

  loadAll()
}

async function signOff(serviceId){
  const date = prompt("Enter sign off date (YYYY-MM-DD):")
  if(!date) return
  await client.from("sea_service").update({ disembarkation_date: date }).eq("id", serviceId)
  loadAll()
}

// ---------------- Load All ----------------
async function loadAll() {

  const { data: seafarers } = await client.from("seafarers").select("*")
  const { data: interviews } = await client.from("interviews").select("*")
  const { data: documents } = await client.from("documents").select("*")
  const { data: vessels } = await client.from("vessels").select("*")
  const { data: seaService } = await client.from("sea_service").select("*")

  allSeafarers = seafarers || []

  const searchValue = document.getElementById("crewSearchInput")?.value?.toLowerCase() || ""
  const table = document.getElementById("crewTable")
  table.innerHTML = ""

  for (let s of filteredSeafarers) {

    const allPositions = seaService?.filter(ss => ss.seafarer_id === s.id) || []
    const activeService = allPositions.find(ss =>
      !ss.disembarkation_date || ss.disembarkation_date === ""
    )

    // 🔥 ВАЖНО — считаем стаж один раз
    const positionExperience = calculateServiceDays(allPositions)

    // текущий ранг
    const currentRank = activeService
      ? activeService.position
      : s.rank || "Unknown"

    // ---------------- HISTORY ----------------
    const historyList = allPositions
      .sort((a,b)=>new Date(b.embarkation_date)-new Date(a.embarkation_date))
      .map(ss => {

        const vessel = vessels?.find(v => v.id === ss.vessel_id)
        const signOffDate = ss.disembarkation_date || "Present"

        const exp = positionExperience[ss.position]
        const formattedExp = exp ? formatExperience(exp) : "0y 0m 0d"

        return `
        <div style="font-size:12px;background:#f1f3f6;padding:6px;margin-bottom:4px;border-radius:6px;">
          <b>${ss.position}</b> (${formattedExp})<br>
          ${vessel?.name || "Unknown"}<br>
          ${ss.embarkation_date} → ${signOffDate}
        </div>`
      }).join("") || "-"

    // ---------------- STATUS ----------------
    const statusHTML = activeService 
      ? `<div style="color:green;font-weight:bold;">
          ON BOARD (${vessels?.find(v=>v.id===activeService.vessel_id)?.name || "Unknown"})
          <br>since ${activeService.embarkation_date}<br><br>
          <button onclick="signOff('${activeService.id}')">Sign Off</button>
        </div>`
      : `<span style="color:gray;font-weight:bold;">ASHORE</span>`

    // ---------------- INTERVIEWS ----------------
    const intList = interviews?.filter(i=>i.seafarer_id===s.id)
      .map(i=>{
        let color="gray"
        if(i.decision==="Approved") color="green"
        if(i.decision==="Standby") color="orange"
        if(i.decision==="Rejected") color="red"

        return `<div style="margin-bottom:8px;background:#f1f3f6;padding:6px;border-radius:6px;">
  <b>${i.interview_date}</b>
  <span style="background:${color};color:white;padding:3px 8px;border-radius:6px;margin-left:6px;">
    ${i.decision}
  </span>
  <div style="white-space:pre-wrap; word-break:break-word; margin-top:6px;">
    ${i.comment || ""}
    <br><b>By: ${i.created_by || "Unknown"}</b>
  </div>
</div>`
      }).join("") || "-"

    // ---------------- DOCUMENTS ----------------
    const docList = documents?.filter(d=>d.seafarer_id===s.id)
      .map(d=>`
        <div>
          <a href="${d.file_url}" target="_blank">${d.doc_type}</a>
          <button onclick="deleteDocument('${d.id}')">Delete</button>
        </div>
      `).join("<br>") || "-"

    // ---------------- Filter ----------------
    const showOnboard = document.getElementById("filterOnboard")?.checked
const showAshore = document.getElementById("filterAshore")?.checked
const checkedPositions = Array.from(document.querySelectorAll(".filterPosition"))
  .filter(cb => cb.checked)
  .map(cb => cb.value)

    let filteredSeafarers = allSeafarers.filter(s=>{
  const activeService = seaService?.some(ss=>ss.seafarer_id===s.id && (!ss.disembarkation_date || ss.disembarkation_date===""))
  const statusOk = (activeService && showOnboard) || (!activeService && showAshore)
  const positionOk = checkedPositions.includes(s.rank)
  const searchOk = s.name.toLowerCase().includes(searchValue) || s.rank.toLowerCase().includes(searchValue)
  return statusOk && positionOk && searchOk
})
    // ---------------- OFFICE COMMENTS ----------------
    // Получаем все комментарии офиса для текущего моряка
const { data: officeComments } = await client
  .from("office_comments")
  .select("*")
  .eq("seafarer_id", s.id)

// Группируем по отделам
const commentsByDept = { QA: [], TSI: [], OPS: [] }
officeComments?.forEach(c => {
  if(c.department && commentsByDept[c.department]){
    commentsByDept[c.department].push(c)
  }
})

// Превращаем массивы в HTML с By и датой
const qaHTML = commentsByDept.QA.length 
  ? commentsByDept.QA.map(c => `${c.comment}<br><b>By: ${c.created_by}</b> (${c.created_at.split('T')[0]})`).join("<br><hr>") 
  : "-"
const tsiHTML = commentsByDept.TSI.length 
  ? commentsByDept.TSI.map(c => `${c.comment}<br><b>By: ${c.created_by}</b> (${c.created_at.split('T')[0]})`).join("<br><hr>") 
  : "-"
const opsHTML = commentsByDept.OPS.length 
  ? commentsByDept.OPS.map(c => `${c.comment}<br><b>By: ${c.created_by}</b> (${c.created_at.split('T')[0]})`).join("<br><hr>") 
  : "-"

    table.innerHTML += `
      <tr>
        <td>${s.name}</td>
        <td>
          <span id="rank_text_${s.id}">${currentRank}</span>
          <button onclick="editRank('${s.id}','${currentRank}')">✏</button>
        </td>
        <td>${statusHTML}</td>
        <td>${historyList}</td>
        <td>${intList}</td>
        <td>${qaHTML}</td>
        <td>${tsiHTML}</td>
        <td>${opsHTML}</td>
        <td>${docList}</td>
      </tr>`
  }

  setupDropdown("docSeafarerSearch","docSeafarer","docSeafarerDropdown", allSeafarers, ["name","rank"])
  setupDropdown("intSearch","intSeafarer","intDropdown", allSeafarers, ["name","rank"])
  setupDropdown("assignSeafarerSearch","assignSeafarer","assignSeafarerDropdown", allSeafarers, ["name","rank"])
  setupDropdown("assignVesselSearch","assignVessel","assignVesselDropdown", vessels || [], ["abbreviation"])
  setupDropdown("commentSeafarerSearch", "commentSeafarer", "commentSeafarerDropdown", allSeafarers, ["name","rank"])
}

// ---------------- Initial Load ----------------
loadAll()
</script>

</body>
</html>
