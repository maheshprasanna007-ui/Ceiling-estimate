# Ceiling-estimate

<!DOCTYPE html>
<html lang="si">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Ceiling Estimate System</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#f1f5f3;color:#20392f;font-family:Arial,sans-serif}
header{background:#173b32;color:white;padding:22px 16px}
header h2{margin:0 0 8px}
main{max-width:680px;margin:auto;padding:14px}
.card{background:white;padding:17px;border-radius:13px;margin-bottom:15px}
h3{margin-top:0}
label{display:block;font-size:14px;margin-top:13px;font-weight:600}
input,select,textarea{width:100%;padding:12px;margin-top:6px;border:1px solid #ccd8d0;border-radius:8px;font-size:16px;background:white}
button{width:100%;padding:14px;margin-top:15px;border:0;border-radius:9px;background:#16734e;color:white;font-size:16px;font-weight:bold}
.hint{font-size:13px;color:#64756b;line-height:1.5}
.room{border:1px solid #d9e2dc;padding:12px;border-radius:9px;margin-top:10px}
.room label{display:flex;gap:10px;align-items:center;margin:0;font-weight:normal}
.room input[type=checkbox]{width:20px;height:20px;margin:0}
.measure{background:#f7faf8;border:1px solid #dbe5de;border-radius:10px;padding:13px;margin-top:12px}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.total{font-size:25px;font-weight:bold;color:#08784d}
button:active{opacity:.8}
</style>
</head>
<body>
<header>
<h2>🏠 Ceiling Estimate System</h2>
<div>Customer Request & Room Measurements</div>
</header>
<main>

<form id="requestForm">

<section class="card">
<h3>01. Customer Details</h3>

<label>Customer Name / නම</label>
<input id="customer" required placeholder="Customer ගේ නම">

<label>Location / ස්ථානය</label>
<input id="location" required placeholder="වැඩ කරන ස්ථානය">

<label>Contact Number</label>
<input id="contact" type="tel" required placeholder="සාමාන්‍ය දුරකථන අංකය">

<label>Customer WhatsApp Number</label>
<input id="customerWA" type="tel" required placeholder="WhatsApp අංකය">
</section>

<section class="card">
<h3>02. Ceiling Type</h3>
<label>Ceiling වර්ගය තෝරන්න</label>
<select id="ceiling" required>
<option value="">-- Select Ceiling Type --</option>
<option>01. Flat Ceiling - One Colour</option>
<option>02. Flat Ceiling - Two Colour</option>
<option>03. Box Type Design Ceiling</option>
</select>
</section>

<section class="card">
<h3>03. Ceiling ගහන ගෙදර කොටස්</h3>
<p class="hint">සිවිලිම ගහන්න අවශ්‍ය සියලු කොටස් තෝරන්න.</p>
<div id="roomChoices"></div>
</section>

<section class="card">
<h3>04. එක් එක් කොටසේ මිනුම්</h3>
<p class="hint">දිග සහ පළල අඩිවලින් ඇතුළත් කරන්න. වර්ග අඩි ගණන ස්වයංක්‍රීයව හැදේ.</p>
<div id="measurements">
තෝරාගත් කොටස් මෙහි පෙන්වයි.
</div>
</section>

<section class="card">
<h3>05. මුළු මිනුම් එකතුව</h3>
<p>තෝරාගත් කොටස් ගණන: <b id="roomCount">0</b></p>
<p>මුළු වර්ග අඩි ගණන:</p>
<div class="total" id="grandArea">0 sq.ft</div>
<label>අමතර විස්තර</label>
<textarea id="notes" rows="3" placeholder="Customer ගේ විශේෂ ඉල්ලීම්"></textarea>

<button type="submit">WhatsApp හරහා Request එක යවන්න</button>
<p class="hint">Submit කළ පසු WhatsApp විවෘත වේ. Message එක යැවීමට WhatsApp තුළ Send ඔබන්න.</p>
</section>

</form>
</main>

<script>
const OWNER_WHATSAPP = "94770154426";

const roomNames = [
"කාමරය (Bedroom)",
"කුස්සිය (Kitchen)",
"සාලය (Living Room)",
"Bathroom එක",
"Veranda / වැරන්ඩාව",
"ඉස්සරහ අඟුව",
"පිටුපස අඟුව",
"Passage / කොරිඩෝව"
];

const roomChoices = document.getElementById("roomChoices");
const measurements = document.getElementById("measurements");

roomNames.forEach((name, index) => {
 const box = document.createElement("div");
 box.className = "room";

 const label = document.createElement("label");
 const check = document.createElement("input");
 check.type = "checkbox";
 check.id = "room_" + index;
 check.value = name;

 const text = document.createElement("span");
 text.textContent = name;

 label.append(check, text);
 box.append(label);
 roomChoices.append(box);

 check.addEventListener("change", renderMeasurements);
});

function renderMeasurements(){
 measurements.replaceChildren();

 const selected = roomNames.filter((_,i) =>
 document.getElementById("room_"+i).checked
 );

 document.getElementById("roomCount").textContent = selected.length;

 if(!selected.length){
  measurements.textContent = "කරුණාකර ගෙදර කොටස් තෝරන්න.";
  updateArea();
  return;
 }

 selected.forEach((name) => {
  const box = document.createElement("div");
  box.className = "measure";

  const title = document.createElement("h4");
  title.textContent = name;
  box.append(title);

  const grid = document.createElement("div");
  grid.className = "grid";

  [["දිග (අඩි)","length"],["පළල (අඩි)","width"]]
  .forEach(([labelText, dimension]) => {
   const wrapper = document.createElement("div");
   const label = document.createElement("label");
   label.textContent = labelText;

   const input = document.createElement("input");
   input.type = "number";
   input.min = "0";
   input.step = "any";
   input.required = true;
   input.placeholder = "අඩි ගණන";
   input.id = dimension + "_" + roomNames.indexOf(name);

   input.addEventListener("input", updateArea);

   label.append(input);
   wrapper.append(label);
   grid.append(wrapper);
  });

  const result = document.createElement("p");
  result.id = "area_" + roomNames.indexOf(name);
  result.textContent = "වර්ග අඩි: 0 sq.ft";

  box.append(grid, result);
  measurements.append(box);
 });

 updateArea();
}

function getRoomData(){
 return roomNames
 .map((name,i) => {
  const checkbox = document.getElementById("room_"+i);
  if(!checkbox.checked) return null;

  const l = Number(document.getElementById("length_"+i).value);
  const w = Number(document.getElementById("width_"+i).value);

  return {name,l,w,area:l*w,i};
 })
 .filter(Boolean);
}

function updateArea(){
 const selected = roomNames.filter((_,i) =>
 document.getElementById("room_"+i).checked
 );

 let total = 0;

 selected.forEach(name => {
  const i = roomNames.indexOf(name);
  const lengthInput = document.getElementById("length_"+i);
  const widthInput = document.getElementById("width_"+i);
  if(!lengthInput || !widthInput) return;

  const l = Math.max(0,Number(lengthInput.value)||0);
  const w = Math.max(0,Number(widthInput.value)||0);
  const area = l*w;

  total += area;

  const result = document.getElementById("area_"+i);
  if(result) result.textContent =
  "වර්ග අඩි: "+area.toLocaleString()+" sq.ft";
 });

 document.getElementById("grandArea").textContent =
 total.toLocaleString()+" sq.ft";
}

document.getElementById("requestForm")
.addEventListener("submit", function(event){
 event.preventDefault();

 const selected = getRoomData();

 if(!selected.length){
  alert("කරුණාකර සිවිලිම ගහන ගෙදර කොටස් තෝරන්න.");
  return;
 }

 if(!document.getElementById("ceiling").value){
  alert("Ceiling වර්ගය තෝරන්න.");
  return;
 }

 const incomplete = selected.some(r =>
 !Number.isFinite(r.l) || !Number.isFinite(r.w) ||
 r.l<=0 || r.w<=0
 );

 if(incomplete){
  alert("තෝරාගත් හැම කොටසකම නිවැරදි දිග සහ පළල ඇතුළත් කරන්න.");
  return;
 }

 const total = selected.reduce((sum,r)=>sum+r.area,0);

 const lines = [
  "NEW CEILING REQUEST",
  "-------------------------",
  "Customer: "+document.getElementById("customer").value,
  "Location: "+document.getElementById("location").value,
  "Contact: "+document.getElementById("contact").value,
  "Customer WhatsApp: "+document.getElementById("customerWA").value,
  "Ceiling Type: "+document.getElementById("ceiling").value,
  "",
  "ROOM MEASUREMENTS"
 ];

 selected.forEach((r,i)=>{
  lines.push(
   (i+1)+". "+r.name,
   "Length: "+r.l+" ft",
   "Width: "+r.w+" ft",
   "Area: "+r.area.toLocaleString()+" sq.ft",
   ""
  );
 });

 lines.push(
  "TOTAL AREA: "+total.toLocaleString()+" sq.ft",
  "",
  "Notes: "+(document.getElementById("notes").value||"-"),
  "",
  "Please prepare a quotation for this request."
 );

 const url =
 "https://wa.me/"+OWNER_WHATSAPP+
 "?text="+encodeURIComponent(lines.join("\n"));

 window.open(url,"_blank");
});

renderMeasurements();
</script>
</body>
</html>
