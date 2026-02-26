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
input, select { padding:6px; margin:5px 0; width:100%; }
table { width:100%; border-collapse: collapse; margin-top:10px; }
th, td { border:1px solid #ddd; padding:8px; vertical-align: top; }
th { background:#eee; }
a { text-decoration:none; color:blue; }
</style>
</head>

<body>

<h2>⚓ Crew Management System</h2>

<div class="card">
<h3>Add Seafarer</h3>
<input id="internal_id" placeholder="Internal ID">
<input id="name" placeholder="Full Name">
<input id="rank" placeholder="Rank">
<button onclick="addSeafarer()">Add</button>
</div>

<div class="card">
<h3>Add Interview</h3>

<div style="position:relative;width:300px;">
  <input 
    type="text" 
    id="intSearch"
    placeholder="Type name or rank..."
    style="width:100%;padding:8px;border:1px solid #ccc;border-radius:6px;"
    autocomplete="off"
  >

  <div id="intDropdown" style="
    position:absolute;
    top:100%;
    left:0;
    right:0;
    background:white;
    border:1px solid #ccc;
    border-top:none;
    max-height:200px;
    overflow-y:auto;
    display:none;
    z-index:1000;
  "></div>
</div>

<input type="hidden" id="intSeafarer">

<input type="date" id="intDate">

<select id="intResult">
<option value="">Select decision</option>
<option value="Approved">Approved</option>
<option value="Standby">Standby</option>
<option value="Rejected">Rejected</option>
</select>

<textarea 
  id="intComments"
  rows="4"
  placeholder="Interview comments..."
  style="
    width:300px;
    padding:8px;
    border-radius:6px;
    border:1px solid #ccc;
    resize:vertical;
  "
></textarea>

<button onclick="addInterview()">Add Interview</button>
</div>

<div class="card">
<h3>Upload PDF</h3>
<select id="docSeafarer"></select>
<select id="docType">
<option value="Certificate">Certificate</option>
<option value="Appraisal">Appraisal</option>
</select>
<input type="file" id="fileInput" accept="application/pdf" multiple>
<button onclick="uploadDocument()">Upload</button>
</div>

<div class="card">
<h3>Add Vessel</h3>
<input id="vesselName" placeholder="Vessel Name">
<input id="vesselAbbr" placeholder="Abbreviation">
<button onclick="addVessel()">Add Vessel</button>
</div>

<div class="card">
<h3>Assign to Vessel</h3>
<select id="assignSeafarer"></select>
<select id="assignVessel"></select>
<label>Embarkation Date</label>
<input type="date" id="embarkDate">
<label>Disembarkation Date</label>
<input type="date" id="disembarkDate">
<button onclick="assignToVessel()">Save</button>
</div>

<div class="card">
<h3>Crew List</h3>

<input 
  type="text" 
  id="searchInput" 
  placeholder="Search by name or rank..."
  style="width:300px;margin-bottom:10px;"
>

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

const client = createClient(
"https://kjtigzaevodgpdtndyqs.supabase.co",
"sb_publishable_qZEENkcQYkmw4oxJP3Lekw_pRerDtsE"
)
  
let allSeafarers = []
  
async function addSeafarer(){

  const name = document.getElementById("name").value
  const rank = document.getElementById("rank").value
  const internal_id = document.getElementById("internal_id").value

  if(!name || !rank){
    alert("Fill all fields")
    return
  }

  const { error } = await client
    .from("seafarers")
    .insert([{
      name: name,
      rank: rank,
      internal_id: internal_id
    }])

  if(error){
    alert(error.message)
    return
  }

  loadAll()
}
  
function editRank(id, currentRank){

  const cell = document.getElementById("rank_text_" + id).parentElement

  cell.innerHTML = `
    <input id="rank_edit_${id}" value="${currentRank}" style="width:70%">
    <button onclick="updateRank('${id}')">💾</button>
  `
}
  
async function addInterview() {
  const seafarer_id = intSeafarer.value
  const interview_date = intDate.value
  const decision = intResult.value
  const comment = intComments.value

  if(!interview_date) return alert("Select interview date")

  await client.from("interviews")
    .insert([{ seafarer_id, interview_date, decision, comment }])

  loadAll()
}

async function uploadDocument() {
  const seafarer_id = docSeafarer.value
  const files = fileInput.files
  const doc_type = docType.value

  if (!seafarer_id || files.length === 0)
    return alert("Select seafarer and file")

  for (let file of files) {

    const filePath = `${seafarer_id}/${Date.now()}_${file.name}`

    await client.storage.from("crew-documents").upload(filePath, file)

    const { data: { publicUrl } } = client.storage
      .from("crew-documents")
      .getPublicUrl(filePath)

    await client.from("documents")
      .insert([{ seafarer_id, file_name: file.name, file_url: publicUrl, doc_type }])
  }

  loadAll()
}

async function deleteDocument(docId){

  if(!confirm("Delete document?")) return

  // 1. Получаем документ
  const { data: doc, error: fetchError } = await client
    .from("documents")
    .select("*")
    .eq("id", docId)
    .single()

  if(fetchError) {
    alert(fetchError.message)
    return
  }

  // 2. Извлекаем путь файла из URL
  const urlParts = doc.file_url.split("/crew-documents/")
  const filePath = urlParts[1]

  // 3. Удаляем файл из storage
  if(filePath){
    await client
      .storage
      .from("crew-documents")
      .remove([filePath])
  }

  // 4. Удаляем запись из БД
  await client
    .from("documents")
    .delete()
    .eq("id", docId)

  loadAll()
}

async function addVessel() {
  const name = vesselName.value
  const abbreviation = vesselAbbr.value
  if(!name || !abbreviation) return alert("Fill all fields")

  await client.from("vessels").insert([{ name, abbreviation }])
  vesselName.value = ""
  vesselAbbr.value = ""
  loadAll()
}

async function assignToVessel() {
  const seafarer_id = assignSeafarer.value
  const vessel_id = assignVessel.value
  const embarkation_date = embarkDate.value
  const disembarkation_date = disembarkDate.value || null

  if(!seafarer_id || !vessel_id || !embarkation_date)
    return alert("All fields required")

  await client.from("sea_service")
    .insert([{ seafarer_id, vessel_id, embarkation_date, disembarkation_date }])

  loadAll()
}

async function signOff(serviceId){
  const date = prompt("Enter sign off date (YYYY-MM-DD):")
  if(!date) return

  await client
    .from("sea_service")
    .update({ disembarkation_date: date })
    .eq("id", serviceId)

  loadAll()
}

  async function updateRank(id){

  const input = document.getElementById("rank_edit_" + id)
  const newRank = input.value

  if(!newRank) return alert("Rank cannot be empty")

  const { error } = await client
    .from("seafarers")
    .update({ rank: newRank })
    .eq("id", id)

  if(error) return alert(error.message)

  loadAll()
}
async function loadAll() {

  const { data: seafarers } = await client.from("seafarers").select("*")
  allSeafarers = seafarers || []
  const searchValue = document.getElementById("searchInput")?.value?.toLowerCase() || ""
  const { data: interviews } = await client.from("interviews").select("*")
  const { data: documents } = await client.from("documents").select("*")
  const { data: vessels } = await client.from("vessels").select("*")
  const { data: seaService } = await client.from("sea_service").select("*")

  const table = crewTable
  table.innerHTML = ""

  ;["intSeafarer","docSeafarer","assignSeafarer"].forEach(id=>{
    const sel = document.getElementById(id)
    sel.innerHTML = ""
    seafarers
?.filter(s => 
  s.name?.toLowerCase().includes(searchValue) ||
  s.rank?.toLowerCase().includes(searchValue)
)
.forEach(s=>{
      sel.innerHTML += `<option value="${s.id}">${s.name}</option>`
    })
  })

  assignVessel.innerHTML = ""
  vessels?.forEach(v=>{
    assignVessel.innerHTML += `<option value="${v.id}">${v.name}</option>`
  })

  seafarers?.forEach(s=>{

    const activeService = seaService?.find(ss =>
      ss.seafarer_id === s.id &&
      (ss.disembarkation_date === null || ss.disembarkation_date === "")
    )
    const historyList = seaService
  ?.filter(ss => ss.seafarer_id === s.id)
  .sort((a,b)=> new Date(b.embarkation_date) - new Date(a.embarkation_date))
  .map(ss => {

    const vessel = vessels?.find(v => v.id === ss.vessel_id)

    const signOffDate = ss.disembarkation_date 
      ? ss.disembarkation_date 
      : "Present"

    return `
      <div style="
        font-size:12px;
        background:#f1f3f6;
        padding:6px;
        margin-bottom:4px;
        border-radius:6px;
      ">
        <b>${vessel ? vessel.name : "Unknown"}</b><br>
        ${ss.embarkation_date} → ${signOffDate}
      </div>
    `
  }).join("") || "-"

    let statusHTML = `<span style="color:gray;font-weight:bold;">ASHORE</span>`

    if(activeService){
      const vessel = vessels?.find(v => v.id === activeService.vessel_id)

      statusHTML = `
        <div style="color:green;font-weight:bold;">
          ON BOARD (${vessel ? vessel.name : "Unknown"})
          <br>since ${activeService.embarkation_date}
          <br><br>
          <button onclick="signOff('${activeService.id}')">
            Sign Off
          </button>
        </div>
      `
    }

   const intList = interviews?.filter(i => i.seafarer_id === s.id)
.sort((a,b)=> new Date(b.interview_date) - new Date(a.interview_date))
.map(i=>{

  let color = "#555"
  if(i.decision==="Approved") color="green"
  if(i.decision==="Standby") color="orange"
  if(i.decision==="Rejected") color="red"

  return `
    <div style="
      margin-bottom:8px;
      padding:6px;
      background:#f1f3f6;
      border-radius:6px;
      font-size:13px;
    ">
      <b>${i.interview_date}</b>
      <span style="
        background:${color};
        color:white;
        padding:3px 8px;
        border-radius:6px;
        margin-left:6px;
        font-weight:bold;
      ">
        ${i.decision}
      </span>
    <div style="
  margin-top:6px;
  color:#333;
  white-space:pre-wrap;
  word-break:break-word;
">
  ${i.comment ? i.comment : ""}
</div>
  `
}).join("") || "-"

    const docList = documents?.filter(d => d.seafarer_id === s.id)
    .map(d => `
      <div>
        <a href="${d.file_url}" target="_blank">${d.doc_type}</a>
        <button onclick="deleteDocument('${d.id}')">Delete</button>
      </div>
    `).join("<br>") || "-"

    table.innerHTML += `
      <tr>
        <td>${s.name}</td>
        <td>
  <span id="rank_text_${s.id}">${s.rank}</span>
  <button onclick="editRank('${s.id}', '${s.rank}')">✏</button>
</td>
        <td>${statusHTML}</td>
        <td>${historyList}</td>
        <td>${intList}</td>
        <td>${docList}</td>
      </tr>
    `
  })
}

  document.getElementById("searchInput").addEventListener("keyup", function(){

  const value = this.value.toLowerCase()
  const rows = document.querySelectorAll("#crewTable tr")

  rows.forEach(row => {

    const name = row.children[0]?.innerText.toLowerCase()
    const rank = row.children[1]?.innerText.toLowerCase()

    if(name.includes(value) || rank.includes(value)){
      row.style.display = ""
    } else {
      row.style.display = "none"
    }

  })
})
  
loadAll()

document.getElementById("intSearch").addEventListener("input", function(){

  const value = this.value.toLowerCase()
  const dropdown = document.getElementById("intDropdown")

  dropdown.innerHTML = ""

  if(!value){
    dropdown.style.display = "none"
    return
  }

  const filtered = allSeafarers.filter(s =>
    (s.name && s.name.toLowerCase().includes(value)) ||
    (s.rank && s.rank.toLowerCase().includes(value))
  )

  if(filtered.length === 0){
    dropdown.style.display = "none"
    return
  }

  filtered.forEach(s=>{
    const item = document.createElement("div")
    item.style.padding = "8px"
    item.style.cursor = "pointer"
    item.style.borderBottom = "1px solid #eee"

    item.innerHTML = `<b>${s.name}</b> — ${s.rank}`

    item.onclick = () => {
      document.getElementById("intSearch").value = s.name
      document.getElementById("intSeafarer").value = s.id
      dropdown.style.display = "none"
    }

    dropdown.appendChild(item)
  })

  dropdown.style.display = "block"
})

  const filtered = allSeafarers.filter(s =>
    s.name.toLowerCase().includes(value) ||
    s.rank.toLowerCase().includes(value)
  )

  if(filtered.length === 0){
    dropdown.style.display = "none"
    return
  }

  filtered.forEach(s=>{
    const item = document.createElement("div")
    item.style.padding = "6px"
    item.style.cursor = "pointer"
    item.innerHTML = `<b>${s.name}</b> — ${s.rank}`

    item.onclick = () => {
      document.getElementById("intSearch").value = s.name
      document.getElementById("intSeafarer").value = s.id
      dropdown.style.display = "none"
    }

    dropdown.appendChild(item)
  })

  dropdown.style.display = "block"
})
  document.addEventListener("click", function(e){
  if(!e.target.closest("#intSearch")){
    document.getElementById("intDropdown").style.display = "none"
  }
})
</script>

</body>
</html>
