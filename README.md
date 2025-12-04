<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Mercatus Globalis</title>

<style>
/* ---------- GLOBAL STYLE ---------- */
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #ffffff;
  color: #222;
}

/* ---------- HEADER ---------- */
header {
  background: #ff7a00;
  color: white;
  padding: 15px;
  text-align: center;
  font-size: 22px;
  font-weight: bold;
  position: sticky;
  top: 0;
  z-index: 1000;
}

/* ---------- CATEGORY BAR ---------- */
.category-bar {
  display: flex;
  overflow-x: auto;
  white-space: nowrap;
  background: #fff4e6;
  padding: 10px;
  border-bottom: 2px solid #ff7a00;
}
.category-bar div {
  margin-right: 18px;
  font-size: 15px;
  padding: 8px 12px;
  border-radius: 6px;
  background: #ff7a00;
  color: white;
  cursor: pointer;
  flex-shrink: 0;
}
.category-bar div:hover {
  background: #e56d00;
}

/* ---------- PRODUCT GRID ---------- */
.products {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 12px;
  padding: 15px;
}

.product {
  background: white;
  padding: 10px;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.12);
  text-align: center;
  transition: 0.2s;
}
.product:hover {
  transform: scale(1.03);
}

.product img {
  width: 100%;
  height: 160px;
  object-fit: cover;
  border-radius: 8px;
}

/* ---------- BUTTON ---------- */
.btn {
  background: #ff7a00;
  color: white;
  padding: 10px;
  border: none;
  width: 100%;
  border-radius: 6px;
  font-size: 15px;
  margin-top: 8px;
  cursor: pointer;
}
.btn:hover {
  background: #e56d00;
}

/* ---------- MODAL ---------- */
.modal-bg {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.6);
  display: none;
  justify-content: center;
  align-items: center;
  padding: 20px;
}
.modal {
  background: white;
  padding: 20px;
  border-radius: 10px;
  width: 100%;
  max-width: 350px;
}
.modal input, .modal select {
  width: 100%;
  padding: 10px;
  margin: 6px 0;
  border-radius: 6px;
  border: 1px solid #ccc;
}

/* ---------- FIX FLOATING BUTTONS ---------- */
.closeBtn {
  background: #ff7a00;
  color: #fff;
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  margin-top: 10px;
}

</style>
</head>

<body>

<header>Mercatus Globalis</header>

<!-- CATEGORY BAR -->
<div class="category-bar">
  <div onclick="filterCategory('All')">All</div>
  <div onclick="filterCategory('Top')">Top</div>
  <div onclick="filterCategory('T-Shirts')">T-Shirts</div>
  <div onclick="filterCategory('Pants')">Pants</div>
  <div onclick="filterCategory('Jeans')">Jeans</div>
  <div onclick="filterCategory('Hoodies')">Hoodies</div>
  <div onclick="filterCategory('Jackets')">Jackets</div>
  <div onclick="filterCategory('Shoes')">Shoes</div>
  <div onclick="filterCategory('Gadgets')">Gadgets</div>
  <div onclick="filterCategory('Other Accessories')">Other Accessories</div>
</div>

<!-- PRODUCT GRID -->
<div id="productContainer" class="products"></div>

<!-- ORDER MODAL -->
<div class="modal-bg" id="modalBg">
  <div class="modal">
    <h3 id="modalProduct"></h3>
    <label>Size</label>
    <select id="orderSize"></select>

    <input id="orderName" placeholder="Name">
    <input id="orderPhone" placeholder="Phone">
    <input id="orderState" placeholder="State">
    <input id="orderCity" placeholder="City">
    <input id="orderVillage" placeholder="Village">
    <input id="orderPin" placeholder="Pincode">

    <select id="orderPayment">
      <option>Online Payment</option>
      <option>Cash on Delivery</option>
    </select>

    <button class="btn" onclick="sendOrder()">Send WhatsApp</button>
    <button class="closeBtn" onclick="closeModal()">Back</button>
  </div>
</div>

<script>
let products = [];
let activeProduct = {};
const SHEET_URL =
 "https://docs.google.com/spreadsheets/d/1mdwssLvXos6BG-ajTUlIbwzguTHjD7LP3jRiDITMbIY/gviz/tq?tqx=out:json&gid=0";

const WHATSAPP = "9332307996";

/* ---------- LOAD PRODUCTS FROM SHEET ---------- */
async function loadProducts() {
  let raw = await fetch(SHEET_URL);
  raw = await raw.text();
  const json = JSON.parse(raw.substring(47, raw.length - 2));

  products = json.table.rows.map(r => {
    return {
      name: r.c[0]?.v || "",
      price: r.c[1]?.v || "",
      category: r.c[2]?.v || "",
      size: r.c[3]?.v || "",
      image: r.c[4]?.v || "",
    };
  });

  showProducts(products);
}

/* ---------- DISPLAY PRODUCTS ---------- */
function showProducts(list) {
  const container = document.getElementById("productContainer");
  container.innerHTML = "";

  list.forEach(p => {
    container.innerHTML += `
      <div class="product">
        <img src="${p.image}">
        <h4>${p.name}</h4>
        <p><b>₹${p.price}</b></p>
        <button class="btn" onclick='openModal(${JSON.stringify(p)})'>Order</button>
      </div>
    `;
  });
}

/* ---------- CATEGORY FILTER ---------- */
function filterCategory(cat) {
  if (cat === "All") return showProducts(products);
  showProducts(products.filter(p => p.category === cat));
}

/* ---------- OPEN MODAL ---------- */
function openModal(p) {
  activeProduct = p;
  document.getElementById("modalProduct").innerText = p.name;

  let sizes = p.size.split(",");
  let sizeHTML = "";
  sizes.forEach(s => sizeHTML += `<option>${s.trim()}</option>`);
  document.getElementById("orderSize").innerHTML = sizeHTML;

  document.getElementById("modalBg").style.display = "flex";
}

/* ---------- CLOSE MODAL ---------- */
function closeModal() {
  document.getElementById("modalBg").style.display = "none";
}

/* ---------- SEND WHATSAPP ORDER ---------- */
function sendOrder() {
  let msg = `
🚨‼️ Alert Mercatus Globalis new Order requests
Order Request:
Product: ${activeProduct.name}
Price: ${activeProduct.price}
Size: ${orderSize.value}
Name: ${orderName.value}
Phone: ${orderPhone.value}
State: ${orderState.value}
City: ${orderCity.value}
Village: ${orderVillage.value}
Pincode: ${orderPin.value}
Payment Method: ${orderPayment.value}
`;

  window.open(`https://wa.me/${WHATSAPP}?text=` + encodeURIComponent(msg));
}

loadProducts();
</script>

</body>
</html>
