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
th, td { border:1px solid #ddd; padding:8px; }
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
<input id="intResult" placeholder="Decision">
<input id="intComments" placeholder="Comment">
<button onclick="addInterview()">Add Interview</button>
</div>

<div class="card">
<h3>Add Appraisal</h3>
<select id="appSeafarer"></select>
<input type="date" id="appDate">
<input id="appScore" placeholder="Score">
<input id="appComments" placeholder="Comment">
<button onclick="addAppraisal()">Add Appraisal</button>
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
<h3>Crew List</h3>
<button onclick="loadAll()">Refresh</button>
<table>
<thead>
<tr>
<th>Name</th>
<th>Rank</th>
<th>Interviews</th>
<th>Appraisals</th>
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

async function addSeafarer() {
const internal_id = document.getElementById("internal_id").value
const name = document.getElementById("name").value
const rank = document.getElementById("rank").value

await client.from("seafarers").insert([{ internal_id, name, rank }])
loadAll()
}

async function addInterview() {
const seafarer_id = parseInt(document.getElementById("intSeafarer").value)
const date = document.getElementById("intDate").value
const decision = document.getElementById("intResult").value
const text_comment = document.getElementById("intComments").value

await client.from("interviews")
.insert([{ seafarer_id, date, decision, text_comment }])

loadAll()
}

async function addAppraisal() {
const seafarer_id = parseInt(document.getElementById("appSeafarer").value)
const date = document.getElementById("appDate").value
const score = document.getElementById("appScore").value
const comments = document.getElementById("appComments").value

await client.from("appraisals")
.insert([{ seafarer_id, date, score, comments }])

loadAll()
}

async function uploadDocument() {
const seafarer_id = parseInt(document.getElementById("docSeafarer").value)
const file = document.getElementById("fileInput").files[0]
const doc_type = document.getElementById("docType").value

if (!file) return alert("Select file")

const filePath = `${seafarer_id}/${Date.now()}_${file.name}`

await client.storage.from("crew-documents")
.upload(filePath, file)

const publicUrl = client.storage
.from("crew-documents")
.getPublicUrl(filePath).data.publicUrl

await client.from("documents")
.insert([{ seafarer_id, file_name: file.name, file_url: publicUrl, doc_type }])

loadAll()
}

async function loadAll() {

const { data: seafarers } = await client.from("seafarers").select("*")
const { data: interviews } = await client.from("interviews").select("*")
const { data: appraisals } = await client.from("appraisals").select("*")
const { data: documents } = await client.from("documents").select("*")

const safeSeafarers = seafarers || []
const safeInterviews = interviews || []
const safeAppraisals = appraisals || []
const safeDocuments = documents || []

const table = document.getElementById("crewTable")
table.innerHTML = ""

const selects = ["intSeafarer","appSeafarer","docSeafarer"]

selects.forEach(id => {
const sel = document.getElementById(id)
sel.innerHTML = ""
safeSeafarers.forEach(s=>{
sel.innerHTML += `<option value="${s.id}">${s.name}</option>`
})
})

safeSeafarers.forEach(s=>{

const intList = safeInterviews
.filter(i=>i.seafarer_id==s.id)
.map(i=>`${i.date} (${i.decision})`)
.join("<br>") || "-"

const appList = safeAppraisals
.filter(a=>a.seafarer_id==s.id)
.map(a=>`${a.date} (Score: ${a.score})`)
.join("<br>") || "-"

const docList = safeDocuments
.filter(d=>d.seafarer_id==s.id)
.map(d=>`<a href="${d.file_url}" target="_blank">${d.doc_type}</a>`)
.join("<br>") || "-"

table.innerHTML += `
<tr>
<td>${s.name}</td>
<td>${s.rank}</td>
<td>${intList}</td>
<td>${appList}</td>
<td>${docList}</td>
</tr>
`
})
}

loadAll()

</script>
</body>
</html>
