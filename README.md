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
<select id="intSeafarer"></select>
<input type="date" id="intDate">
<select id="intResult">
<option value="">Select decision</option>
<option value="Approved">Approved</option>
<option value="Standby">Standby</option>
<option value="Rejected">Rejected</option>
</select>
<input id="intComments" placeholder="Comment">
<button onclick="addInterview()">Add Interview</button>
</div>

<div class="card">
<h3>Upload PDF</h3>
<select id="docSeafarer"></select>
<select id="docType">
<option value="Certificate">Certificate</option>
<option value="Appraisal">Appraisal</option>
</select>
<input type="file" id="fileInput" accept="application/pdf">
<button onclick="uploadDocument()">Upload</button>
</div>

<div class="card">
<h3>Add Vessel</h3>
<input id="vesselName" placeholder="Vessel Name">
<input id="vesselAbbr" placeholder="Abbreviation (e.g. MSC)">
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
<button onclick="loadAll()">Refresh</button>
<table>
<thead>
<tr>
<th>Name</th>
<th>Rank</th>
<th>Interviews</th>
<th>Documents</th>
<th>Status</th>
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

async function addSeafarer() {
  const internal_id = document.getElementById("internal_id").value
  const name = document.getElementById("name").value
  const rank = document.getElementById("rank").value

  const { error } = await client.from("seafarers")
    .insert([{ internal_id, name, rank }])

  if(error) return alert("Error: " + error.message)

  loadAll()
}

async function addInterview() {
  const seafarer_id = document.getElementById("intSeafarer").value
  const interview_date = document.getElementById("intDate").value
  const decision = document.getElementById("intResult").value
  const comment = document.getElementById("intComments").value

  if(!interview_date) return alert("Select interview date")

  const valid = ["Approved","Standby","Rejected"]
  if(!valid.includes(decision))
    return alert("Decision must be Approved, Standby, or Rejected")

  const { error } = await client.from("interviews")
    .insert([{ seafarer_id, interview_date, decision, comment }])

  if(error) return alert("Error: " + error.message)

  loadAll()
}

async function uploadDocument() {
  const seafarer_id = document.getElementById("docSeafarer").value
  const files = document.getElementById("fileInput").files
  const doc_type = document.getElementById("docType").value

  if (!seafarer_id || files.length === 0)
    return alert("Select seafarer and file")

  for (let file of files) {
    const filePath = `${seafarer_id}/${Date.now()}_${file.name}`

    const { error: uploadError } = await client.storage
      .from("crew-documents")
      .upload(filePath, file)

    if (uploadError) return alert(uploadError.message)

    const { data: { publicUrl } } = client.storage
      .from("crew-documents")
      .getPublicUrl(filePath)

    const { error: insertError } = await client.from("documents")
      .insert([{ seafarer_id, file_name: file.name, file_url: publicUrl, doc_type }])

    if (insertError) return alert(insertError.message)
  }

  loadAll()
}

async function loadAll() {
  const { data: seafarers } = await client.from("seafarers").select("*")
  const { data: interviews } = await client.from("interviews").select("*")
  const { data: documents } = await client.from("documents").select("*")
  const { data: vessels } = await client.from("vessels").select("*")

  const safeSeafarers = seafarers || []
  const safeInterviews = interviews || []
  const safeDocuments = documents || []
  const safeVessels = vessels || []

  const table = document.getElementById("crewTable")
  table.innerHTML = ""

  // 🔹 Заполняем все select моряками
  const seafarerSelects = ["intSeafarer","docSeafarer","assignSeafarer"]
  seafarerSelects.forEach(id => {
    const sel = document.getElementById(id)
    sel.innerHTML = ""
    safeSeafarers.forEach(s=>{
      sel.innerHTML += `<option value="${s.id}">${s.name}</option>`
    })
  })

  // 🔹 Заполняем суда
  const vesselSelect = document.getElementById("assignVessel")
  vesselSelect.innerHTML = ""
  safeVessels.forEach(v=>{
    vesselSelect.innerHTML += `<option value="${v.id}">${v.abbreviation}</option>`
  })

  safeSeafarers.forEach(s=>{

    const intList = safeInterviews
      .filter(i => i.seafarer_id == s.id)
      .map(i => `
        <div style="margin-bottom:6px;padding:6px;border:1px solid #eee;border-radius:6px;">
          <b>${i.interview_date}</b>
          <span style="
            font-weight:bold;
            color:${i.decision === 'Approved' ? 'green' : i.decision === 'Rejected' ? 'red' : 'orange'};
          ">
            (${i.decision})
          </span>
          <br>
          <span style="color:#555;">
            ${i.comment ? i.comment : "<i>No comment</i>"}
          </span>
        </div>
      `)
      .join("") || "-"

    const docList = safeDocuments
      .filter(d => d.seafarer_id == s.id)
      .map(d => `<a href="${d.file_url}" target="_blank">${d.doc_type}</a>`)
      .join("<br>") || "-"

    table.innerHTML += `
      <tr>
        <td>${s.name}</td>
        <td>${s.rank}</td>
        <td>${intList}</td>
        <td>${docList}</td>
      </tr>
    `
  })
}

async function assignToVessel() {

  const seafarer_id = document.getElementById("assignSeafarer").value
  const vessel_id = document.getElementById("assignVessel").value
  const embarkation_date = document.getElementById("embarkDate").value
  const disembarkation_date = document.getElementById("disembarkDate").value || null

  if(!seafarer_id || !vessel_id || !embarkation_date)
    return alert("Seafarer, vessel and embarkation date are required")

  const { error } = await client.from("sea_service")
    .insert([{
      seafarer_id,
      vessel_id,
      embarkation_date,
      disembarkation_date
    }])

  if(error) return alert(error.message)

  alert("Assigned successfully")
  loadAll()
}

loadAll()
  async function addVessel() {

  const name = document.getElementById("vesselName").value
  const abbreviation = document.getElementById("vesselAbbr").value

  if(!name || !abbreviation)
    return alert("Name and abbreviation required")

  const { error } = await client.from("vessels")
    .insert([{ name, abbreviation }])

  if(error) return alert(error.message)

  alert("Vessel added")

  document.getElementById("vesselName").value = ""
  document.getElementById("vesselAbbr").value = ""

async function loadAll() {

  const { data: seafarers } = await client.from("seafarers").select("*")
  const { data: interviews } = await client.from("interviews").select("*")
  const { data: documents } = await client.from("documents").select("*")
  const { data: vessels } = await client.from("vessels").select("*")
  const { data: seaService } = await client.from("sea_service").select("*")

  const safeSeafarers = seafarers || []
  const safeInterviews = interviews || []
  const safeDocuments = documents || []
  const safeVessels = vessels || []
  const safeSeaService = seaService || []

  const table = document.getElementById("crewTable")
  table.innerHTML = ""

  // Заполняем селекты
  const seafarerSelects = ["intSeafarer","docSeafarer","assignSeafarer"]
  seafarerSelects.forEach(id => {
    const sel = document.getElementById(id)
    sel.innerHTML = ""
    safeSeafarers.forEach(s=>{
      sel.innerHTML += `<option value="${s.id}">${s.name}</option>`
    })
  })

  const vesselSelect = document.getElementById("assignVessel")
  vesselSelect.innerHTML = ""
  safeVessels.forEach(v=>{
    vesselSelect.innerHTML += `<option value="${v.id}">${v.name}</option>`
  })

  safeSeafarers.forEach(s=>{

    // 🔹 НАХОДИМ АКТИВНЫЙ РЕЙС
    const activeService = safeSeaService.find(ss =>
      ss.seafarer_id == s.id && !ss.disembarkation_date
    )

    let statusHTML = `<span style="color:gray;font-weight:bold;">ASHORE</span>`

    if(activeService){
      const vessel = safeVessels.find(v => v.id == activeService.vessel_id)
      statusHTML = `
        <span style="color:green;font-weight:bold;">
          ON BOARD (${vessel ? vessel.name : "Unknown vessel"})
          <br>
          since ${activeService.embarkation_date}
        </span>
      `
    }

    const intList = safeInterviews
      .filter(i => i.seafarer_id == s.id)
      .map(i => `
        <div style="margin-bottom:6px;padding:6px;border:1px solid #eee;border-radius:6px;">
          <b>${i.interview_date}</b>
          <span style="
            font-weight:bold;
            color:${i.decision === 'Approved' ? 'green' : i.decision === 'Rejected' ? 'red' : 'orange'};
          ">
            (${i.decision})
          </span>
          <br>
          <span style="color:#555;">
            ${i.comment ? i.comment : "<i>No comment</i>"}
          </span>
        </div>
      `)
      .join("") || "-"

    const docList = safeDocuments
      .filter(d => d.seafarer_id == s.id)
      .map(d => `<a href="${d.file_url}" target="_blank">${d.doc_type}</a>`)
      .join("<br>") || "-"

    table.innerHTML += `
      <tr>
        <td>${s.name}</td>
        <td>${s.rank}</td>
        <td>${statusHTML}</td>
        <td>${intList}</td>
        <td>${docList}</td>
      </tr>
    `
  })
}
}
</script>

</body>
</html>
