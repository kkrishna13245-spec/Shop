<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mithai Mahal</title>

<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@600;700&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/emailjs-com@3/dist/email.min.js"></script>

<style>
:root{
--orange:#E8732A;
--brown:#3B1A0A;
--cream:#FDF6EC;
--gold:#C9933A;
}

body{margin:0;font-family:Lato;background:var(--cream);color:var(--brown)}
h1,h2,h3{font-family:'Playfair Display'}

.nav{
position:sticky;top:0;background:#fff;padding:10px;
display:flex;justify-content:space-between;
box-shadow:0 2px 10px rgba(0,0,0,.1)
}

.btn{
background:var(--orange);color:#fff;border:none;
padding:10px;border-radius:8px;margin:3px
}

.container{padding:15px}
.card{background:#fff;padding:10px;margin:8px 0;border-radius:10px}

.hidden{display:none}

.grid{display:grid;grid-template-columns:repeat(2,1fr);gap:10px}

.hero{
background:linear-gradient(135deg,var(--orange),var(--gold));
color:#fff;padding:40px;text-align:center
}

.otp-box input{
width:35px;height:35px;text-align:center;margin:2px
}

.toast{
position:fixed;bottom:20px;left:50%;
transform:translateX(-50%);
background:var(--brown);color:#fff;padding:10px 20px;border-radius:20px
}

.track{display:flex;justify-content:space-between;font-size:12px}
.active{color:var(--orange);font-weight:bold}
</style>
</head>

<body>

<!-- NAV -->
<div class="nav">
<b>Mithai Mahal</b>
<div>
<button onclick="nav('home')">Home</button>
<button onclick="nav('products')">Products</button>
<button onclick="nav('cart')">Cart (<span id="cartCount">0</span>)</button>
<button onclick="nav('ordersPage')">My Orders</button>
<button onclick="nav('help')">Help</button>
<button onclick="nav('dashboard')">Owner</button>
</div>
</div>

<!-- HOME -->
<div id="home">
<div class="hero">
<h1>Mithai Mahal</h1>
<p>Since 1975 • Barabanki</p>
<button class="btn" onclick="nav('products')">Shop Now</button>
</div>
<div class="container">
<h2>Best Sellers</h2>
<div id="plist" class="grid"></div>
</div>
</div>

<!-- PRODUCTS -->
<div id="products" class="hidden container">
<input placeholder="Search" oninput="renderProducts(this.value)">
<div id="productList" class="grid"></div>
</div>

<!-- CART -->
<div id="cart" class="hidden container">
<h2>Cart</h2>
<div id="cartItems"></div>
<h3 id="cartTotal"></h3>
<button class="btn" onclick="nav('checkout')">Checkout</button>
</div>

<!-- CHECKOUT -->
<div id="checkout" class="hidden container">
<div id="step1">
<input id="name" placeholder="Name">
<input id="phone" placeholder="Phone">
<textarea id="addr"></textarea>
<button onclick="step(2)">Next</button>
</div>

<div id="step2" class="hidden">
<button onclick="pay='UPI';showUPI()">UPI</button>
<button onclick="pay='COD'">COD</button>
<p id="upi"></p>
<button onclick="step(3)">Next</button>
</div>

<div id="step3" class="hidden">
<div id="otpBox"></div>
<button onclick="verifyOTP()">Verify</button>
</div>
</div>

<!-- ORDERS -->
<div id="ordersPage" class="hidden container">
<h2>My Orders</h2>
<div id="ordersList"></div>
</div>

<!-- HELP -->
<div id="help" class="hidden container">
<select id="complaintType">
<option>Order Issue</option>
<option>Payment Issue</option>
</select>
<input id="complaintOrderId">
<textarea id="complaintMsg"></textarea>
<button onclick="submitComplaint()">Submit</button>
</div>

<!-- DASHBOARD -->
<div id="dashboard" class="hidden container">
<input id="op">
<input id="oe">
<button onclick="ownerLogin()">Login</button>

<div id="ootp" class="hidden">
<div id="ownerOtp"></div>
<button onclick="verifyOwner()">Verify</button>
</div>

<div id="panel" class="hidden">
<h2>Orders</h2>
<div id="ordersAdmin"></div>

<h2>Products</h2>
<div id="padmin"></div>

<h2>Settings</h2>
<input id="emailKey">
<button onclick="copyCode()">Copy Code</button>
</div>
</div>

<div id="toast" class="toast hidden"></div>

<script>

let products=JSON.parse(localStorage.getItem("products")||JSON.stringify([
{name:"Kaju Katli",price:450},
{name:"Besan Ladoo",price:320,disc:10},
{name:"Jalebi",price:280},
{name:"Rasgulla",price:300,disc:5},
{name:"Gulab Jamun",price:280},
{name:"Soan Papdi",price:350,disc:15},
{name:"Namkeen Mix",price:200},
{name:"Sweet Gift Box",price:599}
]));

let cart=JSON.parse(localStorage.getItem("cart")||"[]");
let otp="",pay="",attempts=0;

function nav(p){
document.querySelectorAll("body>div").forEach(d=>d.classList?.add("hidden"));
document.getElementById(p).classList.remove("hidden");
if(p=="products")renderProducts();
if(p=="cart")renderCart();
if(p=="ordersPage")loadOrders();
}

function renderProducts(q=""){
let html="";
products.filter(p=>p.name.toLowerCase().includes(q.toLowerCase()))
.forEach((p,i)=>{
html+=`
<div class="card">
<h3>${p.name}</h3>
₹${p.price}
<button onclick="add(${i})">Add</button>
</div>`;
});
productList.innerHTML=html;
plist.innerHTML=html;
}

function add(i){
cart.push({i,qty:.5});
save();toast("Added");
}

function renderCart(){
let t=0,html="";
cart.forEach((c,i)=>{
let p=products[c.i];
t+=p.price*c.qty;
html+=`${p.name} ${c.qty}kg 
<button onclick="cart[i].qty+=.5;renderCart()">+</button>
<button onclick="cart[i].qty-=.5;renderCart()">-</button>`;
});
cartItems.innerHTML=html;
let delivery=t>=500?0:40;
cartTotal.innerText="₹"+(t+delivery);
}

function save(){
localStorage.setItem("cart",JSON.stringify(cart));
cartCount.innerText=cart.reduce((a,b)=>a+b.qty,0);
}

function step(n){
step1.classList.add("hidden");
step2.classList.add("hidden");
step3.classList.add("hidden");
document.getElementById("step"+n).classList.remove("hidden");
if(n==3)genOTP();
}

function genOTP(){
otp=Math.floor(100000+Math.random()*900000)+"";
console.log("OTP:",otp);
otpBox.innerHTML="";
for(let i=0;i<6;i++){
let inp=document.createElement("input");
inp.maxLength=1;
inp.oninput=e=>inp.nextSibling?.focus();
otpBox.appendChild(inp);
}
}

function getOTP(){
return [...otpBox.children].map(i=>i.value).join("");
}

function verifyOTP(){
if(getOTP()==otp){
placeOrder();
}else{
attempts++;
if(attempts>=3){toast("Blocked");nav("home")}
else toast("Wrong");
}
}

function placeOrder(){
let id="ORD"+Date.now();
let code=Math.random().toString(36).slice(2,10).toUpperCase();

let orders=JSON.parse(localStorage.getItem("orders")||"[]");
orders.push({id,cart,status:0,code});
localStorage.setItem("orders",JSON.stringify(orders));

window.open(`https://wa.me/919264945100?text=Order ${id}`);

cart=[];save();toast("Order placed");
nav("home");
}

function loadOrders(){
let orders=JSON.parse(localStorage.getItem("orders")||"[]");
let html="";
orders.forEach(o=>{
html+=`
<div class="card">
${o.id}
<div class="track">
<div class="${o.status>=0?'active':''}">Placed</div>
<div class="${o.status>=1?'active':''}">Confirmed</div>
<div class="${o.status>=2?'active':''}">On Way</div>
<div class="${o.status>=3?'active':''}">Delivered</div>
</div>
</div>`;
});
ordersList.innerHTML=html;
}

function submitComplaint(){
let c=JSON.parse(localStorage.getItem("complaints")||"[]");
c.push({msg:complaintMsg.value});
localStorage.setItem("complaints",JSON.stringify(c));
toast("Submitted");
}

function ownerLogin(){
if(op.value=="9264945100" && oe.value=="vermakrishna1324@gmail.com"){
genOTP();ootp.classList.remove("hidden");
}else toast("Wrong");
}

function verifyOwner(){
if(getOTP()==otp){
panel.classList.remove("hidden");
loadOrdersAdmin();loadProductsAdmin();
}else toast("Wrong");
}

function loadOrdersAdmin(){
let orders=JSON.parse(localStorage.getItem("orders")||"[]");
let html="";
orders.forEach(o=>{
html+=`<div>${o.id}</div>`;
});
ordersAdmin.innerHTML=html;
}

function loadProductsAdmin(){
let html="";
products.forEach((p,i)=>{
html+=`
<div>
${p.name}
<input value="${p.price}" onchange="products[${i}].price=this.value;saveProducts()">
</div>`;
});
padmin.innerHTML=html;
}

function saveProducts(){
localStorage.setItem("products",JSON.stringify(products));
}

function copyCode(){
navigator.clipboard.writeText(document.documentElement.outerHTML);
toast("Copied");
}

function toast(t){
toast.innerText=t;
toast.classList.remove("hidden");
setTimeout(()=>toast.classList.add("hidden"),2000);
}

save();renderProducts();

</script>

</body>
</html>
