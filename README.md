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
<h3>Upload Document (PDF)</h3>
<select id="docSeafarer"></select>
<select id="docType">
<option value="">Document Type</option>
<option value="Certificate">Certificate</option>
<option value="Appraisal">Appraisal</option>
</select>
<input type="file" id="fileInput" accept="application/pdf">
<button onclick="uploadDocument()">Upload</button>
</div>

<div class="card">
<h3>Seafarers</h3>
<button onclick="loadSeafarers()">Refresh</button>
<table>
<thead>
<tr>
<th>Name</th>
<th>Rank</th>
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

const { error } = await client
.from("seafarers")
.insert([{ internal_id, name, rank }])

if (error) alert(error.message)
else {
alert("Seafarer added")
loadSeafarers()
}
}

async function uploadDocument() {
const seafarer_id = document.getElementById("docSeafarer").value
const file = document.getElementById("fileInput").files[0]
const doc_type = document.getElementById("docType").value

if (!file || !seafarer_id || !doc_type) {
alert("Fill all fields")
return
}

const filePath = `${seafarer_id}/${Date.now()}_${file.name}`

const { error: uploadError } = await client
.storage
.from("crew-documents")
.upload(filePath, file)

if (uploadError) {
alert(uploadError.message)
return
}

const publicUrl = client
.storage
.from("crew-documents")
.getPublicUrl(filePath).data.publicUrl

const { error: dbError } = await client
.from("documents")
.insert([{
seafarer_id,
file_name: file.name,
file_url: publicUrl,
doc_type
}])

if (dbError) alert(dbError.message)
else {
alert("Document uploaded")
loadSeafarers()
}
}

async function loadSeafarers() {
const { data: seafarers } = await client
.from("seafarers")
.select("*")

const { data: documents } = await client
.from("documents")
.select("*")

const table = document.getElementById("crewTable")
const select = document.getElementById("docSeafarer")

table.innerHTML = ""
select.innerHTML = "<option value=''>Select Seafarer</option>"

seafarers.forEach(s => {

const docs = documents.filter(d => d.seafarer_id == s.id)

let docLinks = docs.length
? docs.map(d => `<div>${d.doc_type}: <a href="${d.file_url}" target="_blank">View</a></div>`).join("")
: "-"

table.innerHTML += `
<tr>
<td>${s.name}</td>
<td>${s.rank}</td>
<td>${docLinks}</td>
</tr>
`

select.innerHTML += `
<option value="${s.id}">${s.name} (${s.rank})</option>
`
})
}

loadSeafarers()
</script>

</body>
</html>
