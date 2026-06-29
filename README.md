# laporankeuangan

<html lang="id">
<head
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FinancePro - Pengatur Keuangan Pribadi</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:'Segoe UI',sans-serif;
}

body{
background:#0f172a;
color:white;
padding:20px;
}

.container{
max-width:1300px;
margin:auto;
}

.header{
margin-bottom:20px;
}

.header h1{
font-size:32px;
font-weight:700;
}

.header p{
color:#94a3b8;
}

.dashboard{
display:grid;
grid-template-columns:repeat(auto-fit,minmax(250px,1fr));
gap:15px;
margin-bottom:20px;
}

.card{
background:rgba(255,255,255,.05);
backdrop-filter:blur(10px);
border:1px solid rgba(255,255,255,.1);
padding:20px;
border-radius:20px;
}

.card h3{
color:#94a3b8;
font-size:14px;
margin-bottom:10px;
}

.card h2{
font-size:28px;
}

.green{
color:#22c55e;
}

.red{
color:#ef4444;
}

.blue{
color:#38bdf8;
}

.orange{
color:#f59e0b;
}

.actions{
display:flex;
flex-wrap:wrap;
gap:10px;
margin-bottom:20px;
}

button{
padding:12px 20px;
border:none;
border-radius:12px;
cursor:pointer;
font-weight:600;
}

.btn-income{
background:#22c55e;
color:white;
}

.btn-expense{
background:#ef4444;
color:white;
}

.btn-balance{
background:#3b82f6;
color:white;
}

.btn-budget{
background:#f59e0b;
color:white;
}

.form-box{
background:rgba(255,255,255,.05);
padding:20px;
border-radius:20px;
margin-bottom:20px;
}

input,select{
width:100%;
padding:12px;
margin-top:10px;
margin-bottom:10px;
border:none;
border-radius:10px;
background:#1e293b;
color:white;
}

.chart-box{
background:rgba(255,255,255,.05);
padding:20px;
border-radius:20px;
margin-bottom:20px;
}

.table-box{
background:rgba(255,255,255,.05);
padding:20px;
border-radius:20px;
overflow:auto;
}

table{
width:100%;
border-collapse:collapse;
}

th,td{
padding:12px;
border-bottom:1px solid rgba(255,255,255,.1);
text-align:left;
}

.warning{
background:#7f1d1d;
padding:15px;
border-radius:12px;
margin-bottom:15px;
display:none;
}
</style>
</head>
<body>

<div class="container">

<div class="header">
<h1>💰 FinancePro</h1>
<p>Pengatur Keuangan Pribadi</p>
</div>

<div id="warning" class="warning"></div>

<div class="dashboard">

<div class="card">
<h3>Total Saldo</h3>
<h2 class="blue" id="saldo">Rp 0</h2>
</div>

<div class="card">
<h3>Total Pemasukan</h3>
<h2 class="green" id="income">Rp 0</h2>
</div>

<div class="card">
<h3>Total Pengeluaran</h3>
<h2 class="red" id="expense">Rp 0</h2>
</div>

<div class="card">
<h3>Sisa Uang</h3>
<h2 class="orange" id="remaining">Rp 0</h2>
</div>

</div>

<div class="actions">
<button class="btn-balance" onclick="setType('saldo')">+ Tambah Saldo</button>
<button class="btn-income" onclick="setType('income')">+ Tambah Pemasukan</button>
<button class="btn-expense" onclick="setType('expense')">+ Tambah Pengeluaran</button>
<button class="btn-budget" onclick="setBudget()">Atur Anggaran</button>
</div>

<div class="form-box">

<input type="date" id="date">

<input type="text" id="description" placeholder="Keterangan">

<select id="category">
<option>Makan</option>
<option>Bensin</option>
<option>Transportasi</option>
<option>Listrik</option>
<option>Internet</option>
<option>Cicilan</option>
<option>Hiburan</option>
<option>Tabungan</option>
<option>Investasi</option>
<option>Belanja</option>
<option>Kesehatan</option>
<option>Pendidikan</option>
<option>Lainnya</option>
</select>

<input type="number" id="amount" placeholder="Jumlah">

<button style="width:100%;background:#22c55e;color:white;"
onclick="addTransaction()">
Simpan Transaksi
</button>

</div>

<div class="chart-box">
<canvas id="expenseChart"></canvas>
</div>

<div class="table-box">

<h2 style="margin-bottom:15px;">
Riwayat Transaksi
</h2>

<table>
<thead>
<tr>
<th>Tanggal</th>
<th>Jenis</th>
<th>Kategori</th>
<th>Keterangan</th>
<th>Jumlah</th>
</tr>
</thead>

<tbody id="history"></tbody>

</table>

</div>

</div>

<script>

let currentType="expense";

let transactions=
JSON.parse(localStorage.getItem("transactions"))||[];

let budget=
Number(localStorage.getItem("budget"))||0;

function setType(type){
currentType=type;
}

function format(num){
return new Intl.NumberFormat('id-ID').format(num);
}

function setBudget(){

let value=prompt(
"Masukkan Anggaran Bulanan:"
);

if(value){
budget=Number(value);
localStorage.setItem("budget",budget);
render();
}

}

function addTransaction(){

const date=document.getElementById("date").value;
const category=document.getElementById("category").value;
const description=document.getElementById("description").value;
const amount=Number(
document.getElementById("amount").value
);

if(!amount)return;

transactions.push({
date,
category,
description,
amount,
type:currentType
});

localStorage.setItem(
"transactions",
JSON.stringify(transactions)
);

document.getElementById("amount").value="";
document.getElementById("description").value="";

render();
}

function render(){

let saldoManual=0;
let income=0;
let expense=0;

transactions.forEach(t=>{

if(t.type==="saldo")
saldoManual+=t.amount;

if(t.type==="income")
income+=t.amount;

if(t.type==="expense")
expense+=t.amount;

});

let saldo=
saldoManual+income-expense;

document.getElementById("saldo").innerText=
"Rp "+format(saldo);

document.getElementById("income").innerText=
"Rp "+format(income);

document.getElementById("expense").innerText=
"Rp "+format(expense);

document.getElementById("remaining").innerText=
"Rp "+format(saldo);

const history=
document.getElementById("history");

history.innerHTML="";

transactions
.slice()
.reverse()
.forEach(t=>{

history.innerHTML+=`
<tr>
<td>${t.date}</td>
<td>${t.type}</td>
<td>${t.category}</td>
<td>${t.description}</td>
<td>Rp ${format(t.amount)}</td>
</tr>
`;

});

if(budget>0 && expense>budget){

const over=expense-budget;

warning.style.display="block";

warning.innerHTML=
`⚠ Pengeluaran melebihi anggaran sebesar Rp ${format(over)}`;

}else{

warning.style.display="none";

}

renderChart();
}

let chart;

function renderChart(){

const categories={};

transactions.forEach(t=>{

if(t.type==="expense"){

categories[t.category]=
(categories[t.category]||0)+t.amount;

}

});

const labels=Object.keys(categories);
const values=Object.values(categories);

if(chart) chart.destroy();

chart=new Chart(
document.getElementById("expenseChart"),
{
type:"doughnut",
data:{
labels:labels,
datasets:[{
data:values
}]
}
}
);

}

document.getElementById("date").value=
new Date().toISOString().split("T")[0];

render();

</script>

</body>
</html>