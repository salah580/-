<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title> إدارة العمال والأجور</title>

  <!-- أيقونات و Chart.js و xlsx -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

  <style>
    :root{
      --primary:#2c3e50;
      --accent:#3498db;
      --success:#2ecc71;
      --warning:#e67e22;
      --danger:#e74c3c;
      --bg:#f4f6f8;
      --card:#fff;
    }
    *{box-sizing:border-box}
    body{font-family:Tahoma, Arial, sans-serif;margin:0;background:var(--bg);color:#222}
    header{background:var(--primary);color:#fff;padding:12px;text-align:center}
    .container{max-width:1024px;margin:12px auto;padding:12px}
    .card{background:var(--card);border-radius:10px;padding:14px;margin-bottom:12px;box-shadow:0 2px 8px rgba(0,0,0,0.06)}
    h2,h3{margin:6px 0}
    input,select,textarea,button{font-size:14px}
    input,select,textarea{width:100%;padding:8px;margin:6px 0;border-radius:8px;border:1px solid #d6d6d6;background:#fff}
    .row{display:flex;gap:8px;flex-wrap:wrap}
    .col{flex:1;min-width:120px}
    .btn{border:0;padding:8px 10px;border-radius:8px;cursor:pointer}
    .btn-primary{background:var(--accent);color:#fff}
    .btn-success{background:var(--success);color:#fff}
    .btn-danger{background:var(--danger);color:#fff}
    .btn-warning{background:var(--warning);color:#fff}
    .muted{color:#666;font-size:13px}
    table{width:100%;border-collapse:collapse;margin-top:10px}
    th,td{border:1px solid #e0e0e0;padding:8px;text-align:center}
    th{background:#fafafa}
    ul{padding:0;margin:0}
    li.item{list-style:none;padding:8px;border-bottom:1px solid #eee;display:flex;justify-content:space-between;align-items:center}
    .item .meta{font-size:13px;color:#555}
    .small{font-size:13px;color:#555}
    .fade-in{opacity:0;transform:translateY(12px);animation:fadeIn .5s forwards}
    @keyframes fadeIn{to{opacity:1;transform:translateY(0)}}

    /* صفحات */
    .page{display:none}
    .page.active{display:block}

    /* شريط التنقل السفلي */
    .bottom-nav{position:fixed;left:0;right:0;bottom:0;background:var(--primary);display:flex;border-top:1px solid rgba(0,0,0,0.12);z-index:999}
    .bottom-nav button{flex:1;padding:10px;border:0;background:transparent;color:#cdd7df;font-size:13px;cursor:pointer;display:flex;flex-direction:column;align-items:center;gap:4px}
    .bottom-nav button i{font-size:18px}
    .bottom-nav button.active{color:var(--accent);border-top:3px solid var(--accent);background:linear-gradient(0deg, rgba(255,255,255,0.02), transparent)}
    @media (max-width:600px){ .container{padding:8px} input,select,button{font-size:13px} }
  </style>
</head>
<body onload="initApp()">
  <header><h2><i class="fas fa-briefcase"></i> تطبيق إدارة العمال والأجور</h2></header>

  <div class="container">
    <!-- الصفحة الرئيسية -->
    <div id="home" class="page active">
      <div class="card">
        <h3><i class="fas fa-home"></i> الصفحة الرئيسية</h3>
        <p id="summary" class="muted">جاري تحميل البيانات...</p>

        <div id="topPlace" style="font-weight:700;margin-top:8px"></div>
        <div id="placesRanking" style="margin-top:12px"></div>

        <div style="margin-top:10px;display:flex;gap:8px;align-items:center">
          <button class="btn btn-primary" onclick="toggleChart()"><i class="fas fa-chart-pie"></i> إظهار/إخفاء الرسم البياني</button>
          <button class="btn" onclick="updateTopPlace()"><i class="fas fa-sync-alt"></i> تحديث</button>
        </div>

        <div style="margin-top:12px">
          <canvas id="placesChart" style="max-width:480px;display:none"></canvas>
        </div>
      </div>
    </div>

    <!-- إدارة العمال -->
    <div id="workers" class="page">
      <div class="card">
        <h3><i class="fas fa-users"></i> إدارة العمال</h3>

        <div style="margin-top:8px" class="card">
          <div class="row">
            <div class="col"><input id="workerName" placeholder="اسم العامل"></div>
            <div class="col"><input id="workerPhone" placeholder="رقم الهاتف (اختياري)"></div>
            <div class="col"><select id="workerPlaceSelect"><option value="">اختر مكان العمل (اختياري)</option></select></div>
          </div>
          <div style="display:flex;gap:8px;margin-top:8px">
            <button class="btn btn-success" onclick="addWorker()"><i class="fas fa-plus"></i> إضافة</button>
            <button class="btn" onclick="clearWorkerInputs()"><i class="fas fa-eraser"></i> مسح الحقول</button>
          </div>
        </div>

        <div class="card">
          <label class="small">تصفية حسب مكان العمل:</label>
          <select id="filterPlace" onchange="displayWorkers()"><option value="all">الكل</option></select>

          <h4 style="margin-top:10px">قائمة العمال</h4>
          <ul id="workerList"></ul>
        </div>
      </div>
    </div>

    <!-- إدخال الأجور -->
    <div id="salaries" class="page">
      <div class="card">
        <h3><i class="fas fa-money-bill-wave"></i> إدخال الأجور</h3>

        <div class="card">
          <label>اختر العامل</label>
          <select id="salaryWorkerSelect"></select>
          <div class="row">
            <div class="col"><input id="hoursInput" type="number" placeholder="عدد الساعات"></div>
            <div class="col"><input id="rateInput" type="number" placeholder="السعر لكل ساعة"></div>
          </div>
          <div style="display:flex;gap:8px;margin-top:8px">
            <button class="btn btn-success" onclick="addSalary()"><i class="fas fa-save"></i> حفظ الأجر</button>
            <button class="btn" onclick="clearSalaryInputs()"><i class="fas fa-eraser"></i> مسح الحقول</button>
          </div>
          <p id="salaryResult" class="small" style="color:green;margin-top:8px"></p>
        </div>

        <div class="card">
          <h4>سجل الأجور الأخير</h4>
          <table id="recentSalariesTable">
            <thead><tr><th>العامل</th><th>الساعات</th><th>السعر</th><th>الأجر</th><th>التاريخ</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- إدارة الأماكن -->
    <div id="placesPage" class="page">
      <div class="card">
        <h3><i class="fas fa-building"></i> إدارة أماكن العمل</h3>

        <div class="card">
          <input id="placeNameInput" placeholder="اسم مكان العمل (مثال: منزل أ)">
          <div style="display:flex;gap:8px;margin-top:8px">
            <button class="btn btn-success" onclick="addPlace()"><i class="fas fa-plus"></i> إضافة</button>
            <button class="btn btn-danger" onclick="clearAllPlaces()"><i class="fas fa-trash"></i> حذف كل الأماكن</button>
          </div>
        </div>

        <div class="card">
          <h4>قائمة الأماكن</h4>
          <ul id="placeList"></ul>
        </div>
      </div>
    </div>

    <!-- السجل -->
    <div id="records" class="page">
      <div class="card">
        <h3><i class="fas fa-file-invoice-dollar"></i> السجل</h3>

        <div class="card">
          <input id="searchInput" placeholder="🔎 ابحث باسم العامل أو الهاتف أو مكان العمل" oninput="searchWorkerRecords()">
          <div id="searchResults" style="margin-top:8px"></div>
        </div>

        <div id="workerDetailsCard" class="card" style="display:none"></div>
      </div>
    </div>

    <!-- الإعدادات -->
    <div id="settings" class="page">
      <div class="card">
        <h3><i class="fas fa-cog"></i> الإعدادات</h3>
        <div style="display:flex;gap:8px;margin-top:8px">
          <button class="btn btn-danger" onclick="resetAllData()"><i class="fas fa-trash-alt"></i> مسح كل البيانات (reset)</button>
          <button class="btn" onclick="exportAllToExcel()"><i class="fas fa-file-export"></i> تصدير الكل (Excel)</button>
        </div>
      </div>
    </div>
  </div>

  <!-- شريط التنقل السفلي -->
  <div class="bottom-nav" role="navigation" aria-label="تنقّل التطبيق">
    <button onclick="showPage('home', event)" class="active"><i class="fas fa-home"></i>الرئيسية</button>
    <button onclick="showPage('workers', event)"><i class="fas fa-users"></i>العمال</button>
    <button onclick="showPage('salaries', event)"><i class="fas fa-money-bill-wave"></i>الأجور</button>
    <button onclick="showPage('placesPage', event)"><i class="fas fa-building"></i>الأماكن</button>
    <button onclick="showPage('records', event)"><i class="fas fa-file-invoice-dollar"></i>السجل</button>
    <button onclick="showPage('settings', event)"><i class="fas fa-cog"></i>الإعدادات</button>
  </div>

<script>
/* ===== بيانات = localStorage ===== */
let workers = [];    // {id, name, phone, placeId|null}
let salaries = [];   // {id, workerId, hours, rate, total, date}
let places = [];     // {id, name}
let chartInstance = null;

function saveData(){
  localStorage.setItem('workers', JSON.stringify(workers));
  localStorage.setItem('salaries', JSON.stringify(salaries));
  localStorage.setItem('places', JSON.stringify(places));
}
function loadData(){
  workers = JSON.parse(localStorage.getItem('workers') || '[]');
  salaries = JSON.parse(localStorage.getItem('salaries') || '[]');
  places = JSON.parse(localStorage.getItem('places') || '[]');
}

/* ===== تنقل الصفحات ===== */
function showPage(id, e){
  document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  // تمييز زر الشريط السفلي
  document.querySelectorAll('.bottom-nav button').forEach(b=>b.classList.remove('active'));
  if(e && e.currentTarget) e.currentTarget.classList.add('active');
  // تحديثات صفحية
  if(id==='home') updateTopPlace();
  if(id==='workers'){ refreshPlaceDropdowns(); displayWorkers(); }
  if(id==='salaries'){ refreshWorkerSelect(); displayRecentSalaries(); }
  if(id==='placesPage'){ displayPlaces(); }
  if(id==='records'){ document.getElementById('workerDetailsCard').style.display='none'; document.getElementById('searchResults').innerHTML=''; }
}

/* ===== الأماكن ===== */
function addPlace(){
  const name = document.getElementById('placeNameInput').value.trim();
  if(!name) return alert('ادخل اسم المكان');
  const id = Date.now().toString();
  places.push({id, name});
  document.getElementById('placeNameInput').value = '';
  saveData();
  refreshPlaceDropdowns();
  displayPlaces();
  updateTopPlace();
}
function displayPlaces(){
  const ul = document.getElementById('placeList'); ul.innerHTML = '';
  if(!places.length){ ul.innerHTML = '<li class="item">لا يوجد أماكن بعد</li>'; return; }
  places.forEach(p=>{
    const li = document.createElement('li'); li.className='item';
    li.innerHTML = `<div><strong>${p.name}</strong></div>
      <div>
        <button class="btn" title="عرض العمال في هذا المكان" onclick="showWorkersByPlace('${p.id}')"><i class="fas fa-users"></i></button>
        <button class="btn" title="تعديل" onclick="promptEditPlace('${p.id}')"><i class="fas fa-edit"></i></button>
        <button class="btn btn-danger" title="حذف" onclick="deletePlace('${p.id}')"><i class="fas fa-trash"></i></button>
      </div>`;
    ul.appendChild(li);
  });
}
function promptEditPlace(id){
  const p = places.find(x=>x.id===id); if(!p) return;
  const newName = prompt('الاسم الجديد:', p.name);
  if(newName === null) return;
  p.name = newName.trim();
  saveData(); refreshPlaceDropdowns(); displayPlaces(); updateTopPlace();
}
function deletePlace(id){
  if(!confirm("هل تريد حذف هذا المكان؟ سيتم تحويل العمال المرتبطين به إلى 'بدون مكان'")) return;
  workers.forEach(w => { if(w.placeId === id) w.placeId = null; });
  places = places.filter(p => p.id !== id);
  saveData(); refreshPlaceDropdowns(); displayPlaces(); displayWorkers(); updateTopPlace();
}
function clearAllPlaces(){
  if(!confirm("حذف كل الأماكن؟ سيصبح كل العمال 'بدون مكان'")) return;
  places = []; workers.forEach(w => w.placeId = null);
  saveData(); refreshPlaceDropdowns(); displayPlaces(); displayWorkers(); updateTopPlace();
}

/* ===== العمال ===== */
function addWorker(){
  const name = document.getElementById('workerName').value.trim();
  const phone = document.getElementById('workerPhone').value.trim();
  const placeId = document.getElementById('workerPlaceSelect').value || null;
  if(!name) return alert('ادخل اسم العامل');
  const id = Date.now().toString();
  workers.push({id, name, phone, placeId});
  document.getElementById('workerName').value=''; document.getElementById('workerPhone').value='';
  saveData(); refreshWorkerSelect(); displayWorkers(); updateTopPlace();
}
function displayWorkers(){
  const ul = document.getElementById('workerList'); ul.innerHTML = '';
  const filter = document.getElementById('filterPlace')?.value || 'all';
  let list = workers;
  if(filter !== 'all') list = workers.filter(w => (w.placeId || '') === filter);
  if(!list.length){ ul.innerHTML = '<li class="item">لا يوجد عمال</li>'; return; }
  list.forEach(w=>{
    const place = places.find(p=>p.id===w.placeId);
    const li = document.createElement('li'); li.className='item';
    li.innerHTML = `<div>
        <strong>${w.name}</strong>
        <div class="meta">${w.phone || ''} — ${place ? place.name : 'بدون مكان'}</div>
      </div>
      <div>
        <button class="btn" title="تعديل" onclick="promptEditWorker('${w.id}')"><i class="fas fa-edit"></i></button>
        <button class="btn btn-danger" title="حذف" onclick="deleteWorker('${w.id}')"><i class="fas fa-trash"></i></button>
      </div>`;
    ul.appendChild(li);
  });
}
function promptEditWorker(id){
  const w = workers.find(x=>x.id===id); if(!w) return;
  const newName = prompt('الاسم الجديد:', w.name);
  if(newName !== null) w.name = newName.trim();
  const newPhone = prompt('الهاتف الجديد:', w.phone || '');
  if(newPhone !== null) w.phone = newPhone.trim();
  // اختيار مكان عبر قائمة بسيطة
  let list = places.map(p=>`${p.name} (id:${p.id})`).join('\n');
  let newPlace = prompt(`قائمة الأماكن:\n${list}\n\nادخل id المكان الجديد أو اترك فارغ:`, w.placeId || '');
  if(newPlace !== null) w.placeId = newPlace.trim() || null;
  saveData(); displayWorkers(); refreshWorkerSelect(); updateTopPlace();
}
function deleteWorker(id){
  if(!confirm('هل تريد حذف هذا العامل؟ سيتم حذف سجلات الأجر الخاصة به أيضًا')) return;
  workers = workers.filter(w => w.id !== id);
  salaries = salaries.filter(s => s.workerId !== id);
  saveData(); displayWorkers(); refreshWorkerSelect(); displayRecentSalaries(); updateTopPlace();
}
function clearWorkerInputs(){
  document.getElementById('workerName').value=''; document.getElementById('workerPhone').value=''; document.getElementById('workerPlaceSelect').value='';
}

/* ===== تحديث قوائم الاختيار ===== */
function refreshPlaceDropdowns(){
  // workerPlaceSelect and filterPlace
  const sel = document.getElementById('workerPlaceSelect');
  const filter = document.getElementById('filterPlace');
  if(sel){
    sel.innerHTML = "<option value=''>اختر مكان العمل (اختياري)</option>";
    places.forEach(p=>{ const o = document.createElement('option'); o.value = p.id; o.textContent = p.name; sel.appendChild(o); });
  }
  if(filter){
    const prev = filter.value || 'all';
    filter.innerHTML = "<option value='all'>الكل</option>";
    places.forEach(p=>{ const o = document.createElement('option'); o.value = p.id; o.textContent = p.name; filter.appendChild(o); });
    filter.value = prev;
  }
}
function refreshWorkerSelect(){
  const sel = document.getElementById('salaryWorkerSelect');
  if(!sel) return;
  sel.innerHTML = '';
  if(!workers.length){ sel.innerHTML = "<option value=''>لا يوجد عمال</option>"; return; }
  workers.forEach(w=>{ const p = places.find(x=>x.id===w.placeId); const o = document.createElement('option'); o.value = w.id; o.textContent = `${w.name}${p ? ' ('+p.name+')' : ''}`; sel.appendChild(o); });
}

/* ===== أجور ===== */
function addSalary(){
  const workerId = document.getElementById('salaryWorkerSelect').value;
  const hours = parseFloat(document.getElementById('hoursInput').value);
  const rate = parseFloat(document.getElementById('rateInput').value);
  if(!workerId) return alert('اختر عاملًا');
  if(isNaN(hours) || isNaN(rate)) return alert('أدخل قيمة صحيحة للساعات والسعر');
  const total = hours * rate;
  const id = Date.now().toString();
  const date = new Date().toISOString();
  salaries.push({id, workerId, hours, rate, total, date});
  document.getElementById('hoursInput').value=''; document.getElementById('rateInput').value='';
  saveData(); document.getElementById('salaryResult').textContent = `✅ تم حفظ: ${total}`;
  displayRecentSalaries(); updateTopPlace();
}
function displayRecentSalaries(){
  const tbody = document.querySelector('#recentSalariesTable tbody'); tbody.innerHTML = '';
  const sorted = salaries.slice().sort((a,b)=> new Date(b.date) - new Date(a.date)).slice(0,10);
  if(!sorted.length){ tbody.innerHTML = '<tr><td colspan="5">لا توجد سجلات</td></tr>'; return; }
  sorted.forEach(s=>{
    const w = workers.find(x=>x.id===s.workerId) || {name:'غير معروف'};
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${w.name}</td><td>${s.hours}</td><td>${s.rate}</td><td>${s.total}</td><td>${new Date(s.date).toLocaleString('ar-EG')}</td>`;
    tbody.appendChild(tr);
  });
}
function clearSalaryInputs(){ document.getElementById('hoursInput').value=''; document.getElementById('rateInput').value=''; document.getElementById('salaryResult').textContent=''; }

/* ===== السجل/بحث/عرض تفاصيل العامل ===== */
function searchWorkerRecords(){
  const q = document.getElementById('searchInput').value.trim().toLowerCase();
  const resultsDiv = document.getElementById('searchResults'); resultsDiv.innerHTML = '';
  if(!q) return;
  const list = workers.filter(w=>{
    const p = places.find(x=>x.id===w.placeId);
    return (w.name && w.name.toLowerCase().includes(q)) ||
           (w.phone && w.phone.includes(q)) ||
           (p && p.name.toLowerCase().includes(q));
  });
  if(!list.length){ resultsDiv.innerHTML = '<div class="small muted">لا توجد نتائج</div>'; return; }
  list.forEach(w=>{
    const btn = document.createElement('button'); btn.className='btn'; btn.style.display='block'; btn.style.marginBottom='6px';
    btn.textContent = `${w.name} — ${w.phone || ''}`;
    btn.onclick = ()=> showWorkerDetails(w.id);
    resultsDiv.appendChild(btn);
  });
}
function showWorkerDetails(workerId){
  const w = workers.find(x=>x.id===workerId); if(!w) return;
  const place = places.find(p=>p.id===w.placeId);
  const details = document.getElementById('workerDetailsCard');
  details.style.display='block';
  const wSalaries = salaries.filter(s=>s.workerId===workerId).sort((a,b)=> new Date(b.date)-new Date(a.date));
  const fromId = `from_${workerId}`, toId = `to_${workerId}`, rowsId = `rows_${workerId}`, sumId = `sum_${workerId}`;
  let html = `<h4>👤 ${w.name}</h4><p>📞 ${w.phone || 'لا يوجد'} — 🏠 ${place ? place.name : 'بدون مكان'}</p>`;
  html += `<div style="margin-top:8px"><button class="btn btn-warning" onclick="exportWorkerToExcel('${workerId}')"><i class="fas fa-file-download"></i> تحميل سجل</button></div>`;
  html += `<div style="margin-top:10px"><label>من: <input type="date" id="${fromId}"></label> <label>إلى: <input type="date" id="${toId}"></label> <button class="btn btn-success" onclick="filterWorkerByDate('${workerId}')">تصفية</button></div>`;
  if(wSalaries.length){
    html += `<table style="margin-top:10px"><thead><tr><th>الساعات</th><th>السعر/ساعة</th><th>الأجر</th><th>التاريخ</th></tr></thead><tbody id="${rowsId}">`;
    wSalaries.forEach(s=> html += `<tr><td>${s.hours}</td><td>${s.rate}</td><td>${s.total}</td><td>${new Date(s.date).toLocaleString('ar-EG')}</td></tr>`);
    html += `</tbody></table>`;
    html += `<p id="${sumId}" style="font-weight:700;margin-top:8px">مجموع: ${wSalaries.reduce((a,b)=>a+b.total,0)}</p>`;
  } else {
    html += `<p>❌ لا يوجد سجل أجور لهذا العامل</p>`;
  }
  details.innerHTML = html;
}
function filterWorkerByDate(workerId){
  const fromEl = document.getElementById(`from_${workerId}`);
  const toEl = document.getElementById(`to_${workerId}`);
  const rowsEl = document.getElementById(`rows_${workerId}`);
  const sumEl = document.getElementById(`sum_${workerId}`);
  if(!rowsEl) return;
  let list = salaries.filter(s=>s.workerId===workerId);
  const from = fromEl && fromEl.value ? new Date(fromEl.value) : null;
  const to = toEl && toEl.value ? new Date(toEl.value + 'T23:59:59') : null;
  if(from) list = list.filter(s=> new Date(s.date) >= from);
  if(to) list = list.filter(s=> new Date(s.date) <= to);
  rowsEl.innerHTML = ''; let sum=0;
  list.forEach(s=>{ rowsEl.innerHTML += `<tr><td>${s.hours}</td><td>${s.rate}</td><td>${s.total}</td><td>${new Date(s.date).toLocaleString('ar-EG')}</td></tr>`; sum+=s.total; });
  if(sumEl) sumEl.textContent = `مجموع (بعد التصفية): ${sum}`;
}

/* ===== تصدير Excel ===== */
function exportWorkerToExcel(workerId){
  const w = workers.find(x=>x.id===workerId); if(!w) return alert('خطأ');
  const rows = salaries.filter(s=>s.workerId===workerId).map(s=> {
    const place = places.find(p=>p.id===w.placeId);
    return {"العامل": w.name, "الهاتف": w.phone||"", "المكان": place?place.name:"بدون مكان", "الساعات": s.hours, "السعر": s.rate, "الأجر": s.total, "التاريخ": new Date(s.date).toLocaleString('ar-EG')};
  });
  if(!rows.length) return alert('لا يوجد سجلات للتصدير');
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, `سجل_${w.name}`); XLSX.writeFile(wb, `سجل_${w.name}.xlsx`);
}
function exportAllToExcel(){
  if(!salaries.length) return alert('لا توجد بيانات للتصدير');
  const rows = salaries.map(s=>{
    const w = workers.find(x=>x.id===s.workerId) || {};
    const place = places.find(p=>p.id === (w.placeId || '')) || {};
    return {"العامل": w.name || "غير معروف", "الهاتف": w.phone || "", "المكان": place.name || "بدون مكان", "الساعات": s.hours, "السعر": s.rate, "الأجر": s.total, "التاريخ": new Date(s.date).toLocaleString('ar-EG')};
  });
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "جميع_الأجور"); XLSX.writeFile(wb, "سجل_الأجور.xlsx");
}

/* ===== الصفحة الرئيسية: ترتيب الأماكن + رسم بياني + نسب ===== */
function toggleChart(){
  const canvas = document.getElementById('placesChart');
  if(canvas.style.display === 'none' || canvas.style.display === ''){ canvas.style.display = 'block'; updateTopPlace(); }
  else { canvas.style.display = 'none'; if(chartInstance){ chartInstance.destroy(); chartInstance = null; } }
}
function updateTopPlace(){
  const topDiv = document.getElementById('topPlace');
  const rankingDiv = document.getElementById('placesRanking');
  const canvas = document.getElementById('placesChart');
  topDiv.textContent = ''; rankingDiv.innerHTML = '';

  // ملخص
  document.getElementById('summary').textContent = `👥 عدد العمال: ${workers.length} — 💰 مجموع الأجور: ${salaries.reduce((a,b)=>a+(b.total||0),0)}`;

  if(!places.length && !salaries.length){
    topDiv.textContent = '📊 لا توجد بيانات كافية بعد لعرض الترتيب';
    if(chartInstance){ chartInstance.destroy(); chartInstance=null; }
    return;
  }

  // احسب عدد الأجور لكل مكان (بما في ذلك بدون مكان)
  const counts = {};
  places.forEach(p=>counts[p.id]=0);
  counts['_none'] = 0;
  salaries.forEach(s=>{
    const w = workers.find(x=>x.id===s.workerId);
    const pid = (w && w.placeId) ? w.placeId : '_none';
    counts[pid] = (counts[pid]||0) + 1;
  });

  const arr = Object.entries(counts).map(([pid,ct])=>{
    const name = pid === '_none' ? 'بدون مكان' : (places.find(p=>p.id===pid)?.name || 'غير معروف');
    return {id: pid, name, count: ct};
  }).sort((a,b)=>b.count - a.count);

  const totalOps = arr.reduce((a,b)=>a + b.count, 0) || 1;
  const top = arr[0] || {name:'لا يوجد', count:0};
  topDiv.textContent = `📊 أكثر مكان عمل حركة: ${top.name} — ${top.count} عملية (${((top.count/totalOps)*100).toFixed(1)}%)`;

  // عرض ترتيب مع نسب مئوية
  const ol = document.createElement('ol');
  arr.forEach((it,i)=>{ const li = document.createElement('li'); li.className='fade-in'; li.style.animationDelay = (i*0.12)+'s'; li.textContent = `${it.name} — ${it.count} عملية (${((it.count/totalOps)*100).toFixed(1)}%)`; ol.appendChild(li); });
  rankingDiv.appendChild(ol);

  // رسم دائري إذا كان مرئيًا
  if(canvas.style.display !== 'none'){
    const labels = arr.map(a=>a.name);
    const data = arr.map(a=>a.count);
    const palette = ["#3498db","#e74c3c","#2ecc71","#f1c40f","#9b59b6","#e67e22","#1abc9c","#34495e","#95a5a6"];
    const backgroundColor = data.map((_,i)=>palette[i % palette.length]);
    if(chartInstance){ chartInstance.destroy(); chartInstance = null; }
    chartInstance = new Chart(canvas.getContext('2d'), {
      type:'pie',
      data:{ labels, datasets:[{ data, backgroundColor }] },
      options:{ responsive:true, plugins:{ legend:{position:'bottom'}, tooltip:{callbacks:{ label:(ctx)=>{ const percent = ((ctx.raw/totalOps)*100).toFixed(1); return `${ctx.label}: ${ctx.raw} (${percent}%)`; } } } } }
    });
  }
}

/* ===== حذف/إعادة تعيين كامل ===== */
function resetAllData(){
  if(!confirm('هل تريد مسح كل البيانات نهائياً؟')) return;
  workers = []; salaries = []; places = [];
  saveData();
  location.reload();
}

/* ===== تهيئة عند الفتح ===== */
function initApp(){
  loadData();
  refreshPlaceDropdowns();
  refreshWorkerSelect();
  displayPlaces();
  displayWorkers();
  displayRecentSalaries();
  updateTopPlace();
}
</script>
</body>
</html>

حساب الاجور العمال اونلاين
