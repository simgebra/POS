<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Interactive POS Mobile Demo</title>
<style>
body { font-family: Arial; background:#f4f6f8; display:flex; justify-content:center; align-items:center; height:100vh; }
#app { background:#fff; padding:20px; width:320px; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.1); }
.screen { display:none; }
.screen.active { display:block; }
input, select { display:block; width:100%; margin:10px 0; padding:8px; }
button { width:100%; padding:10px; margin:5px 0; background:#007bff; color:#fff; border:none; border-radius:5px; cursor:pointer; }
button:hover { background:#0056b3; }
ul { padding-left:20px; }
li { margin:5px 0; }
</style>
</head>
<body>
<div id="app">
  <!-- Login Screen -->
  <div id="login-screen" class="screen active">
    <h2>POS Mobile App</h2>
    <input type="email" id="email" placeholder="Email">
    <input type="password" id="password" placeholder="Password">
    <button onclick="login()">Login</button>
  </div>

  <!-- Dashboard Screen -->
  <div id="dashboard-screen" class="screen">
    <h2>Dashboard</h2>
    <p id="welcome"></p>
    <button onclick="showScreen('products')">Products</button>
    <button onclick="showScreen('sales')">Create Sale</button>
    <button onclick="showScreen('reports')">Reports</button>
    <button onclick="logout()">Logout</button>
  </div>

  <!-- Products Screen -->
  <div id="products-screen" class="screen">
    <h2>Products</h2>
    <ul id="products-list"></ul>
    <button onclick="showScreen('dashboard')">Back</button>
  </div>

  <!-- Sales Screen -->
  <div id="sales-screen" class="screen">
    <h2>New Sale</h2>
    <div id="sale-items"></div>
    <p>Total: $<span id="total">0</span></p>
    <input type="number" id="paid" placeholder="Amount Paid" value="0" oninput="updateBalance()">
    <p>Balance: $<span id="balance">0</span></p>
    <select id="payment-type">
      <option value="cash">Cash</option>
      <option value="credit">Credit</option>
    </select>
    <button onclick="submitSale()">Submit Sale</button>
    <button onclick="showScreen('dashboard')">Back</button>
  </div>

  <!-- Reports Screen -->
  <div id="reports-screen" class="screen">
    <h2>Reports</h2>
    <ul>
      <li>Sales Report</li>
      <li>Purchases Report</li>
      <li>Stock Report</li>
      <li>Cash Flow Report</li>
      <li>Profit & Loss Report</li>
    </ul>
    <button onclick="showScreen('dashboard')">Back</button>
  </div>
</div>

<script>
// Global Variables
let currentUser = null;
let products = [
  {id:1, name:'Product A', stock:50, price:100},
  {id:2, name:'Product B', stock:30, price:200},
  {id:3, name:'Product C', stock:10, price:150}
];
let saleItems = [];

// Functions
function showScreen(screenId) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById(screenId + '-screen').classList.add('active');

  if(screenId==='dashboard' && currentUser){
    document.getElementById('welcome').innerText=`Welcome, ${currentUser.name}`;
  }

  if(screenId==='products'){
    const productsList=document.getElementById('products-list');
    productsList.innerHTML='';
    products.forEach(p=>{
      const li=document.createElement('li');
      li.innerText=`${p.name} | Stock: ${p.stock} | $${p.price}`;
      productsList.appendChild(li);
    });
  }

  if(screenId==='sales'){
    const saleDiv = document.getElementById('sale-items');
    saleDiv.innerHTML='';
    saleItems = products.map(p => ({...p, qty:0}));
    saleItems.forEach((p,i)=>{
      const div = document.createElement('div');
      div.innerHTML=`${p.name} ($${p.price}) Qty: <input type="number" min="0" max="${p.stock}" value="0" onchange="updateQty(${i}, this.value)">`;
      saleDiv.appendChild(div);
    });
    updateTotal();
  }
}

function login() {
  const email = document.getElementById('email').value;
  const password = document.getElementById('password').value;
  if(email && password){
    currentUser={name:'John Doe', email};
    showScreen('dashboard');
  } else alert('Enter email and password');
}

function logout(){
  currentUser=null;
  showScreen('login');
}

function updateQty(index, value){
  saleItems[index].qty = parseInt(value);
  updateTotal();
}

function updateTotal(){
  let total = saleItems.reduce((sum,p)=>sum + (p.qty*p.price),0);
  document.getElementById('total').innerText = total;
  updateBalance();
}

function updateBalance(){
  let paid = parseFloat(document.getElementById('paid').value) || 0;
  let total = saleItems.reduce((sum,p)=>sum + (p.qty*p.price),0);
  document.getElementById('balance').innerText = (paid - total).toFixed(2);
}

function submitSale(){
  let total = saleItems.reduce((sum,p)=>sum + (p.qty*p.price),0);
  if(total===0){ alert('Select at least one product'); return; }
  let paid = parseFloat(document.getElementById('paid').value) || 0;
  let balance = paid - total;
  let paymentType = document.getElementById('payment-type').value;

  // Update stock
  saleItems.forEach(p=>{
    let prod = products.find(x=>x.id===p.id);
    prod.stock -= p.qty;
  });

  alert(`Sale Submitted!\nTotal: $${total}\nPaid: $${paid}\nBalance: $${balance}\nPayment Type: ${paymentType}`);
  showScreen('dashboard');
}
</script>
</body>
</html>