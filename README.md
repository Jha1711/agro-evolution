# agro-evolution
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Agro Evolution Dashboard</title>

<style>
/* Your existing CSS stays the same */
* {margin:0;padding:0;box-sizing:border-box;font-family:"Segoe UI",sans-serif;}
body {display:flex;background:#f4f7f6;}
.sidebar {width:240px;background:#1b5e20;color:white;height:100vh;padding:20px;}
.sidebar h2 {margin-bottom:30px;}
.sidebar ul {list-style:none;}
.sidebar ul li {padding:12px 0;cursor:pointer;transition:0.3s;}
.sidebar ul li:hover {padding-left:8px;color:#c8e6c9;}
.main {flex:1;padding:20px;}
.topbar {display:flex;justify-content:space-between;align-items:center;margin-bottom:20px;}
.topbar input {padding:8px;width:250px;}
.cards {display:grid;grid-template-columns:repeat(4,1fr);gap:15px;margin-bottom:20px;}
.card {background:white;padding:20px;border-radius:8px;box-shadow:0 3px 8px rgba(0,0,0,0.1);}
.card h3 {margin-bottom:10px;color:#2e7d32;}
table {width:100%;border-collapse:collapse;background:white;margin-bottom:20px;}
table th, table td {padding:10px;border-bottom:1px solid #ddd;text-align:left;}
table th {background:#2e7d32;color:white;}
.product-grid {display:grid;grid-template-columns:repeat(3,1fr);gap:15px;}
.product {background:white;padding:15px;border-radius:8px;box-shadow:0 3px 8px rgba(0,0,0,0.1);}
.product h4 {color:#1b5e20;}
#reportList li {margin:10px 0;}
#reportList a {text-decoration:none;color:#2e7d32;font-weight:bold;}
@media(max-width:900px){.cards{grid-template-columns:repeat(2,1fr);}.product-grid{grid-template-columns:repeat(2,1fr);}}
@media(max-width:600px){.sidebar{display:none;}.cards{grid-template-columns:1fr;}.product-grid{grid-template-columns:1fr;}}
</style>
</head>

<body>

<div class="sidebar">
  <h2>🌾 Agro Evolution</h2>
  <ul>
    <li onclick="scrollToTop()">Dashboard</li>
    <li>Farmers</li>
    <li>Crops</li>
    <li>Products</li>
    <li onclick="scrollToReports()">Reports</li>
    <li>Settings</li>
  </ul>
</div>

<div class="main">

  <div class="topbar">
    <h2>Agro Found</h2>
    <input type="text" id="searchInput" placeholder="Search Crop..." onkeyup="searchCrop()">
  </div>

  <!-- CARDS -->
  <div class="cards">
    <div class="card">
      <h3>Total Crops</h3>
      <p id="totalCrops"></p>
    </div>
    <div class="card">
      <h3>Total Products</h3>
      <p id="totalProducts"></p>
    </div>
    <div class="card">
      <h3>Climate Zones</h3>
      <p>5 Types</p>
    </div>
    <div class="card">
      <h3>Research Reports</h3>
      <p id="reportCount">0</p>
    </div>
  </div>

  <!-- TABLE -->
  <h3>Crop Research Data</h3>
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>Season</th>
        <th>Soil</th>
        <th>Water</th>
        <th>Yield</th>
      </tr>
    </thead>
    <tbody id="cropTable"></tbody>
  </table>

  <!-- PRODUCTS -->
  <h3>Available Agricultural Products</h3>
  <div class="product-grid" id="productList"></div>

  <!-- REPORT SECTION -->
  <h3>Research Reports</h3>
  <input type="file" id="fileUpload" accept=".pdf,.doc,.docx" onchange="uploadFile()">
  <ul id="reportList"></ul>

</div>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-storage-compat.js"></script>

<script>
// 🔹 Initialize Firebase
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT.firebaseio.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com"
};
firebase.initializeApp(firebaseConfig);

const db = firebase.database();
const storage = firebase.storage();

// 🔹 Load Dashboard
function loadDashboard() {
  // Crops
  db.ref("crops").on("value", snap => {
    const cropTable = document.getElementById("cropTable");
    cropTable.innerHTML = "";
    snap.forEach(child => {
      const crop = child.val();
      cropTable.innerHTML += `
        <tr>
          <td>${crop.name}</td>
          <td>${crop.season}</td>
          <td>${crop.soil}</td>
          <td>${crop.water}</td>
          <td>${crop.yield}</td>
        </tr>
      `;
    });
    document.getElementById("totalCrops").innerText = snap.numChildren();
  });

  // Products
  db.ref("products").on("value", snap => {
    const productList = document.getElementById("productList");
    productList.innerHTML = "";
    snap.forEach(child => {
      const product = child.val();
      productList.innerHTML += `
        <div class="product">
          <h4>${product.name}</h4>
          <p><strong>Type:</strong> ${product.type}</p>
          <p><strong>Price:</strong> ${product.price}</p>
        </div>
      `;
    });
    document.getElementById("totalProducts").innerText = snap.numChildren();
  });

  // Reports
  db.ref("reports").on("value", snap => {
    const reportList = document.getElementById("reportList");
    reportList.innerHTML = "";
    snap.forEach(child => {
      const report = child.val();
      const li = document.createElement("li");
      li.innerHTML = `<a href="${report.url}" target="_blank">${report.name}</a>`;
      reportList.appendChild(li);
    });
    document.getElementById("reportCount").innerText = snap.numChildren();
  });
}

// 🔹 Search Crops
function searchCrop() {
  const keyword = document.getElementById("searchInput").value.toLowerCase();
  db.ref("crops").once("value", snap => {
    const cropTable = document.getElementById("cropTable");
    cropTable.innerHTML = "";
    snap.forEach(child => {
      const crop = child.val();
      if(crop.name.toLowerCase().includes(keyword)) {
        cropTable.innerHTML += `
          <tr>
            <td>${crop.name}</td>
            <td>${crop.season}</td>
            <td>${crop.soil}</td>
            <td>${crop.water}</td>
            <td>${crop.yield}</td>
          </tr>
        `;
      }
    });
  });
}

// 🔹 Upload File to Firebase Storage
function uploadFile() {
  const fileInput = document.getElementById("fileUpload");
  const file = fileInput.files[0];
  if(!file) return;

  const storageRef = storage.ref("reports/" + file.name);
  storageRef.put(file).then(() => {
    storageRef.getDownloadURL().then(url => {
      db.ref("reports").push({name: file.name, url: url});
    });
  });
}

// 🔹 Scroll
function scrollToReports() {
  document.getElementById("fileUpload").scrollIntoView({ behavior: "smooth" });
}

function scrollToTop() {
  window.scrollTo({ top: 0, behavior: "smooth" });
}

// 🔹 Load everything on page load
loadDashboard();
</script>

</body>
</html>
