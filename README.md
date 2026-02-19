<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Crew Management System</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js"></script>
  <style>
    body { font-family: Arial; margin:40px; background:#f4f6f8; }
    .card { background:white; padding:20px; margin-bottom:20px; border-radius:8px; }
    button { padding:8px 14px; cursor:pointer; }
    input { padding:6px; margin:5px 0; width:100%; }
    table { width:100%; border-collapse: collapse; margin-top:10px; }
    th, td { border:1px solid #ddd; padding:8px; }
    th { background:#eee; }
  </style>
</head>
<body>

<h2>⚓ Crew Management MVP</h2>

<div class="card">
  <h3>Add Seafarer</h3>
  <input id="internal_id" placeholder="Internal ID">
  <input id="name" placeholder="Full Name">
  <input id="rank" placeholder="Rank">
  <button onclick="addSeafarer()">Add</button>
</div>

<div class="card">
  <h3>Seafarers List</h3>
  <button onclick="loadSeafarers()">Refresh</button>
  <table>
    <thead>
      <tr>
        <th>Internal ID</th>
        <th>Name</th>
        <th>Rank</th>
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

  if (error) {
    alert("Error: " + error.message)
  } else {
    alert("Added successfully")
    loadSeafarers()
  }
}

async function loadSeafarers() {
  const { data, error } = await client
    .from("seafarers")
    .select("*")

  if (error) {
    alert(error.message)
    return
  }

  const table = document.getElementById("crewTable")
  table.innerHTML = ""

  data.forEach(row => {
    table.innerHTML += `
      <tr>
        <td>${row.internal_id}</td>
        <td>${row.name}</td>
        <td>${row.rank}</td>
      </tr>
    `
  })
}

loadSeafarers()
</script>

</body>
</html>
