# MeWe
متجر لبيع الشنط الحريمي
<!doctype html>
<html lang="ar">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>متجر الشنط</title>
  <style>
    :root{--accent:#b84a6b;--muted:#666}
    body{font-family:system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial;direction:rtl;margin:0;color:#111;background:#f7f7f8}
    header{background:linear-gradient(90deg,#fff 0,#fff);padding:18px 20px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #eee}
    .brand{display:flex;align-items:center;gap:12px}
    .logo{width:44px;height:44px;border-radius:8px;background:var(--accent);display:flex;align-items:center;justify-content:center;color:#fff;font-weight:700}
    .container{max-width:1100px;margin:24px auto;padding:0 16px}
    .grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:18px}
    .card{background:#fff;border-radius:10px;padding:12px;box-shadow:0 6px 18px rgba(20,20,30,0.05);display:flex;flex-direction:column}
    .card img{width:100%;height:180px;object-fit:cover;border-radius:8px}
    .title{font-weight:600;margin:10px 0 6px}
    .price{color:var(--accent);font-weight:700;font-size:1.05rem}
    .btn{background:var(--accent);color:#fff;border:0;padding:10px;border-radius:8px;cursor:pointer;margin-top:auto}
    .cart{position:fixed;left:18px;bottom:18px;background:#fff;border-radius:12px;padding:10px 12px;box-shadow:0 8px 24px rgba(0,0,0,0.12);width:320px;max-width:90%}
    .cart h4{margin:0 0 8px}
    .row{display:flex;justify-content:space-between;align-items:center;gap:8px}
    .small{font-size:0.9rem;color:var(--muted)}
    .checkout{margin-top:12px}
    input,select,textarea{width:100%;padding:10px;border:1px solid #e6e6e6;border-radius:8px}
    footer{padding:18px;text-align:center;color:#777;margin-top:40px}
    @media (max-width:640px){.card img{height:160px}}
  </style>
</head>
<body>
  <header>
    <div class="brand">
      <div class="logo">MM</div>
      <div>
        <div style="font-weight:700">متجر شنط</div>
        <div class="small">شنط حريمي — توصيل لجميع المحافظات</div>
      </div>
    </div>
    <nav class="small">تواصل: 0123456789</nav>
  </header>

  <main class="container">
    <section>
      <h2>منتجاتنا</h2>
      <div class="grid" id="products"></div>
    </section>
  </main>

  <div class="cart" id="cartBox" style="display:none">
    <h4>سلة المشتريات</h4>
    <div id="cartItems"></div>
    <div class="row small" style="margin-top:8px">
      <div>المجموع:</div>
      <div id="cartTotal">0 ج.م</div>
    </div>

    <div class="checkout">
      <label class="small">اسم العميل</label>
      <input id="custName" placeholder="اسمك" />
      <label class="small" style="margin-top:8px">رقم الهاتف</label>
      <input id="custPhone" placeholder="01XXXXXXXXX" />
      <label class="small" style="margin-top:8px">طريقة الدفع</label>
      <select id="payMethod">
        <option value="cod">كاش عند الاستلام</option>
        <option value="bank">تحويل بنكي / فوري</option>
        <option value="paypal">دفع عبر PayPal (تجريبي)</option>
        <option value="gateway">بوابة دفع إلكترونية (Paymob/PayTabs)</option>
      </select>
      <div style="margin-top:8px" id="payNote" class="small">اختر طريقة الدفع ثم اضغط تأكيد الطلب</div>
      <button class="btn" style="width:100%;margin-top:10px" id="placeOrder">تأكيد الطلب</button>
    </div>
  </div>

  <script>
    // ----- بيانات منتجات تجريبية: استبدلها بمنتجاتك الحقيقية -----
    const PRODUCTS = [
      {id:1,title:"شنطة كلاسيك جلد",price:1200, img:"https://images.unsplash.com/photo-1542291026-7eec264c27ff?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=1"},
      {id:2,title:"شنطة صغيرة سهره",price:850, img:"https://images.unsplash.com/photo-1600180758895-9d43d35b6a6a?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=2"},
      {id:3,title:"شنطة كتف مودرن",price:950, img:"https://images.unsplash.com/photo-1585386959984-a415522a2b86?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=3"},
      {id:4,title:"شنطة يد فاخرة",price:2200, img:"https://images.unsplash.com/photo-1536305030018-4b1b1a4b0a46?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=4"}
    ];

    // رندر المنتجات
    const productsEl = document.getElementById('products');
    PRODUCTS.forEach(p=>{
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `
        <img src="${p.img}" alt="${p.title}">
        <div class="title">${p.title}</div>
        <div class="small">وصف مختصر للشنطة هنا</div>
        <div class="row" style="margin-top:8px">
          <div class="price">${p.price} ج.م</div>
          <button class="btn" onclick="addToCart(${p.id})">أضف للسلة</button>
        </div>
      `;
      productsEl.appendChild(card);
    });

    // سلة المشتريات (ببساطة في الذاكرة المحلية)
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');

    function saveCart(){ localStorage.setItem('cart', JSON.stringify(cart)); renderCart(); }

    function addToCart(id){
      const item = PRODUCTS.find(x=>x.id===id);
      const existing = cart.find(c=>c.id===id);
      if(existing) existing.qty++;
      else cart.push({id:item.id,title:item.title,price:item.price,qty:1});
      saveCart();
      showCart();
    }

    function renderCart(){
      const box = document.getElementById('cartBox');
      const itemsEl = document.getElementById('cartItems');
      itemsEl.innerHTML = '';
      if(cart.length===0){ box.style.display='none'; return; }
      box.style.display='block';
      let total=0;
      cart.forEach(ci=>{
        total += ci.price * ci.qty;
        const row = document.createElement('div');
        row.className='row';
        row.style.marginTop='8px';
        row.innerHTML = `<div>
            <div style="font-weight:600">${ci.title}</div>
            <div class="small">${ci.qty} × ${ci.price} ج.م</div>
          </div>
          <div style="text-align:left">
            <button onclick="changeQty(${ci.id},1)" style="margin-bottom:6px">＋</button>
            <button onclick="changeQty(${ci.id},-1)">−</button>
          </div>`;
        itemsEl.appendChild(row);
      });
      document.getElementById('cartTotal').innerText = total + ' ج.م';
    }

    function changeQty(id,delta){
      const it = cart.find(c=>c.id===id);
      if(!it) return;
      it.qty += delta;
      if(it.qty<=0) cart = cart.filter(c=>c.id!==id);
      saveCart();
    }

    function showCart(){ document.getElementById('cartBox').style.display='block'; renderCart(); }
    renderCart();

    // زر تأكيد الطلب (هنا نُريّح التعامل مع الدفع: نرسل طلب لخادم أو تعرض تعليمات)
    document.getElementById('placeOrder').addEventListener('click', ()=>{
      const name = document.getElementById('custName').value.trim();
      const phone = document.getElementById('custPhone').value.trim();
      const method = document.getElementById('payMethod').value;
      if(cart.length===0){ alert('السلة فارغة'); return; }
      if(!name || !phone){ alert('ادخل الاسم ورقم الهاتف'); return; }

      // حساب المجموع
      const total = cart.reduce((s,i)=>s + i.price*i.qty, 0);

      if(method==='cod'){
        alert(`تم استلام الطلب (كاش عند الاستلام).\nالمجموع: ${total} ج.م\nسيتم الاتصال بك لتأكيد الطلب.`);
        // هنا يمكنك ارسال بيانات الطلب لقاعدة بيانات / ايميل
      } else if(method==='bank'){
        // مثال تعليمات تحويل بنكي / فوري
        alert(`تحويل بنكي:\nإسم البنك: بنك المثال\nرقم الحساب: 123-456-789\nمبلغ: ${total} ج.م\nأرسل إيصال التحويل لرقم الواتساب.`);
      } else if(method==='paypal'){
        // زر PayPal تجريبي — في البيئة الحقيقية تستدعي SDK/Checkout
        alert('PayPal: سيتم توجيهك لبوابة PayPal (تجريبي).');
      } else if(method==='gateway'){
        alert('بوابة دفع إلكترونية: سيتم توجيهك إلى صفحة الدفع (بعد الربط بالبوابة في الإعدادات).');
      }

      // فرّغ السلة بعد الطلب (اختياري)
      cart = [];
      saveCart();
    });

    // تحديث نص الملاحظات عند تغيير طريقة الدفع
    document.getElementById('payMethod').addEventListener('change', (e)=>{
      const v=e.target.value;
      const note = document.getElementById('payNote');
      if(v==='cod') note.innerText='الدفع عند الاستلام — لا حاجة لربط بوابة دفع.';
      else if(v==='bank') note.innerText='عرض بيانات التحويل البنكي أو فوري بعد الضغط على تأكيد.';
      else if(v==='paypal') note.innerText='PayPal متوفر دولياً — يتطلب حساب PayPal وتكامل.';
      else note.innerText='ربط بوابة دفع محلية (Paymob/PayTabs) يتطلب مفاتيح API وأعدادات على الخادم.';
    });
  </script>
</body>
</html>