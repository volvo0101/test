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
  <h3>Add Interview</h3>
  <select id="seafarerSelect"></select>
  <select id="rating">
    <option value="">Rating (1–7)</option>
    <option>1</option><option>2</option><option>3</option>
    <option>4</option><option>5</option><option>6</option><option>7</option>
  </select>
  <select id="decision">
    <option value="">Decision</option>
    <option>Approved</option>
    <option>Standby</option>
    <option>Rejected</option>
  </select>
  <textarea id="comment" placeholder="Comment"></textarea>
  <button onclick="addInterview()">Save Interview</button>
</div>

<div class="card">
  <h3>Seafarers</h3>
  <button onclick="loadSeafarers()">Refresh</button>
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>Rank</th>
        <th>Avg Rating</th>
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
    alert("Added")
    loadSeafarers()
  }
}

async function addInterview() {
  const seafarer_id = document.getElementById("seafarerSelect").value
  const rating = document.getElementById("rating").value
  const decision = document.getElementById("decision").value
  const comment = document.getElementById("comment").value

  const { error } = await client
    .from("interviews")
    .insert([{ seafarer_id, rating, decision, comment }])

  if (error) alert(error.message)
  else {
    alert("Interview saved")
    loadSeafarers()
  }
}

async function loadSeafarers() {
  const { data: seafarers } = await client
    .from("seafarers")
    .select("*")

  const { data: interviews } = await client
    .from("interviews")
    .select("*")

  const table = document.getElementById("crewTable")
  const select = document.getElementById("seafarerSelect")

  table.innerHTML = ""
  select.innerHTML = "<option value=''>Select Seafarer</option>"

  seafarers.forEach(s => {
    const related = interviews.filter(i => i.seafarer_id === s.id)
    const avg = related.length
      ? (related.reduce((sum,i)=>sum+i.rating,0) / related.length).toFixed(2)
      : "-"

    table.innerHTML += `
      <tr>
        <td>${s.name}</td>
        <td>${s.rank}</td>
        <td>${avg}</td>
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
