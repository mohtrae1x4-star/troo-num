(function(){
let fav=[],interval,stopped=!1,found=null,uiVisibleBox=!0,uiVisiblePanel=!0;
function h(f){
  let n=f.replace(/\D/g,"");
  return /^(050|055|053)/.test(n) ||
         /(\d)\1{3}/.test(n) ||
         /(\d)\1{4}/.test(n) ||
         /(\d)\1{2}(\d)\2{2}/.test(n) ||
         /(\d)\1{2}\d(\d)\2{2}/.test(n) ||
         /^(\d)(\d)\1\2\1\2/.test(n) ||
         /^(\d)(\d)\2\1\2\1/.test(n) ||
         /000/.test(n) ||
         /(\d)00(\d)00/.test(n) ||
         /(\d)0(\d)0(\d)0/.test(n) ||
         /(\d)\1{2}/.test(n) ||
         /100|200|300|400|500|600|700|800|900/.test(n)
}
function clickBtn(){if(stopped||found)return;let b=[...document.querySelectorAll("button")].find(x=>x.innerText.replace(/\s/g,'').includes("خياراتاضافية"));if(b)b.click()}
function notify(num){
  let o=document.createElement("div");
  o.style.cssText="position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,.6);display:flex;align-items:center;justify-content:center;z-index:9999;";
  let box=document.createElement("div");
  box.style.cssText="background:#fff;padding:30px 40px;border-radius:15px;text-align:center;font-size:24px;font-weight:bold;max-width:90%;word-wrap:break-word;";
  box.innerHTML="🔥 لقينا رقم مميز 🔥<br><br>"+num;
  let ok=document.createElement("button");ok.innerText="موافق";
  ok.style.cssText="margin:10px;font-size:20px;width:140px;height:40px;border-radius:8px;";
  ok.onclick=function(){stopped=!0;clearInterval(interval);document.body.removeChild(o)}
  let cancel=document.createElement("button");cancel.innerText="إلغاء";
  cancel.style.cssText="margin:10px;font-size:20px;width:140px;height:40px;border-radius:8px;";
  cancel.onclick=function(){found=null;document.body.removeChild(o)}
  box.appendChild(document.createElement("br"));box.appendChild(ok);box.appendChild(cancel);o.appendChild(box);document.body.appendChild(o)
}
function makeDraggable(el){
  el.style.touchAction="none";el.style.userSelect="none";
  let d=false,startX,startY,translateX=0,translateY=0;
  el.addEventListener("mousedown",e=>{d=true;startX=e.clientX;startY=e.clientY;});
  document.addEventListener("mouseup",()=>{d=false;});
  document.addEventListener("mousemove",e=>{if(!d)return;let dx=e.clientX-startX,dy=e.clientY-startY;el.style.transform=`translate(${translateX+dx}px,${translateY+dy}px)`});
  el.addEventListener("mouseup",()=>{let m=new WebKitCSSMatrix(el.style.transform);translateX=m.m41;translateY=m.m42;el.style.transform=`translate(${translateX}px,${translateY}px)`})
}
function panelButtons(){
  let p=document.createElement("div");p.id="controlPanel";p.style.cssText="display:flex;flex-direction:column;align-items:center;gap:8px;";
  let btnStyle="width:140px;height:40px;font-size:20px;border-radius:8px;";
  let stop=document.createElement("button");stop.innerText="⛔";stop.style.cssText=btnStyle+"background:#d33;color:white;border:none;";
  stop.onclick=function(){stopped=!0;clearInterval(interval);fav=[];found=null;console.clear();alert("تم الإيقاف")};
  let restart=document.createElement("button");restart.innerText="▶️";restart.style.cssText=btnStyle+"background:#3a3;color:white;border:none;";
  restart.onclick=function(){clearInterval(interval);stopped=!1;found=null;interval=setInterval(clickBtn,1500)};
  let toggleBox=document.createElement("button");toggleBox.innerText="🔍 طي / فتح";toggleBox.style.cssText=btnStyle+"background:#555;color:white;border:none;";
  toggleBox.onclick=function(){uiVisibleBox=!uiVisibleBox;let c=document.getElementById("checkingBox");if(c)c.style.display=uiVisibleBox?"block":"none";}
  p.appendChild(stop);p.appendChild(restart);p.appendChild(toggleBox);document.body.appendChild(p);makeDraggable(p);return p;
}
function checkBox(nums){
  let c=document.getElementById("checkingBox");
  if(!c){c=document.createElement("div");c.id="checkingBox";c.style.cssText="position:fixed;top:80px;right:20px;background:#222;color:lime;padding:10px;border-radius:8px;font-family:monospace;font-size:14px;z-index:9999;width:220px;height:auto;max-height:250px;overflow:auto;";document.body.appendChild(c);makeDraggable(c)}
  c.innerHTML="📡 جاري فحص الأرقام:<br>"+nums.join("<br>")
}
function toggleMenu(){
  let container=document.createElement("div");container.id="menuContainer";container.style.cssText="position:fixed;bottom:90px;left:20px;z-index:9999;display:flex;flex-direction:column;gap:6px;";
  document.body.appendChild(container);makeDraggable(container);
  let btnMenu=document.createElement("button");btnMenu.innerText="📋 القائمة";btnMenu.style.cssText="width:140px;height:40px;background:#444;color:white;border-radius:8px;font-size:18px;";
  let menuOpen=!1;
  btnMenu.onclick=function(){menuOpen=!menuOpen;for(let child of container.children){if(child!==btnMenu)child.style.display=menuOpen?"flex":"none"}};
  container.appendChild(btnMenu);let panel=panelButtons();panel.style.display="none";container.appendChild(panel)
}
const oOpen=XMLHttpRequest.prototype.open,oSend=XMLHttpRequest.prototype.send;
XMLHttpRequest.prototype.open=function(m,u){this._url=u;return oOpen.apply(this,arguments)}
XMLHttpRequest.prototype.send=function(){this.addEventListener("load",function(){if(stopped||found)return;if(this._url&&this._url.includes("/number/available/voice")){try{const r=JSON.parse(this.responseText);if(r.numbers&&Array.isArray(r.numbers)){const full=r.numbers.map(n=>n.startsWith("5")?"0"+n:n);checkBox(full);for(let num of full){if(h(num)&&!fav.includes(num)){fav.push(num);found=num;new Audio("https://www.soundjay.com/buttons/beep-07.mp3").play();notify(num);break}}if(fav.length>0)console.table(fav)}}catch(e){}}});return oSend.apply(this,arguments)};
interval=setInterval(clickBtn,1500);
toggleMenu();
})();


(function(){
    function autoSelectAndBuy() {
        console.log("🚀 السكربت يعمل.. الرجاء اختيار الرقم يدوياً الآن.");
        
        // دالة لمراقبة الضغط على الأرقام
        const handleManualSelection = (event) => {
            // التحقق إذا كان العنصر المضغوط هو أحد أزرار الأرقام
            if (event.target.classList.contains('btn_select_number') || event.target.closest('.btn_select_number')) {
                console.log("✅ تم رصد اختيارك للرقم.. جاري إكمال الخطوات تلقائياً.");
                
                // إزالة المستمع لعدم تكرار العملية
                document.removeEventListener('click', handleManualSelection);

                // البدء في تتبع ظهور زر السلة والمتابعة
                startCartProcess();
            }
        };

        // إضافة مستمع للضغط على الصفحة
        document.addEventListener('click', handleManualSelection);
    }

    function startCartProcess() {
        // 2️⃣ ننتظر زر السلة ليظهر ونضغطه
        const cartInterval = setInterval(() => {
            const cartIcon = document.querySelector('.stc-icon-cart-1');
            if (cartIcon) {
                cartIcon.closest('button').click();
                clearInterval(cartInterval);

                // ⏳ انتظار 2 ثانية قبل "اشتر الآن"
                setTimeout(() => {
                    waitForBuyNow();
                }, 2000);
            }
        }, 50);
    }

    function waitForBuyNow() {
        let checkBuyNow = setInterval(() => {
            let buyNowBtn = [...document.querySelectorAll('button')]
                .find(btn => btn.innerText.trim() === "اشتر الآن");

            if (buyNowBtn) {
                buyNowBtn.click();
                clearInterval(checkBuyNow);

                setTimeout(() => {
                    waitForContinue();
                }, 1000);
            }
        }, 50);
    }

    function waitForContinue() {
        let checkContinue = setInterval(() => {
            let continueBtn = [...document.querySelectorAll('button')]
                .find(btn => btn.innerText.trim() === "استمرار");

            if (continueBtn) {
                continueBtn.click();
                clearInterval(checkContinue);

                // ⏳ ننتظر 1 ثانية قبل ظهور مربع الرقم
                setTimeout(() => {
                    createNumberOverlay();
                }, 1000);
            }
        }, 50);
    }

    // ------------------- مربعات الرقم، التحقق، البريد -------------------
    let numberOverlay = null, numberInput = null;
    let verificationOverlay = null, verificationInput = null;
    let emailOverlay = null, emailInput = null;

    // --- مربع الرقم ---
    function createNumberOverlay() {
        if(numberOverlay) return;

        numberOverlay = document.createElement('div');
        numberOverlay.style.position = 'fixed';
        numberOverlay.style.top = '0';
        numberOverlay.style.left = '0';
        numberOverlay.style.width = '100%';
        numberOverlay.style.height = '100%';
        numberOverlay.style.backgroundColor = 'rgba(0,0,0,0.35)';
        numberOverlay.style.display = 'flex';
        numberOverlay.style.alignItems = 'center';
        numberOverlay.style.justifyContent = 'center';
        numberOverlay.style.zIndex = '9999';
        numberOverlay.style.backdropFilter = 'blur(4px)';

        let box = document.createElement('div');
        box.style.backgroundColor = '#fff';
        box.style.borderRadius = '20px';
        box.style.padding = '30px 20px';
        box.style.width = '300px';
        box.style.boxShadow = '0 6px 15px rgba(0,0,0,0.2)';
        box.style.textAlign = 'center';
        box.style.fontFamily = '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
        box.style.position = 'relative';

        const closeBtn = document.createElement('div');
        closeBtn.innerHTML = '✕';
        closeBtn.style.position = 'absolute';
        closeBtn.style.top = '10px';
        closeBtn.style.right = '12px';
        closeBtn.style.cursor = 'pointer';
        closeBtn.style.fontSize = '18px';
        closeBtn.style.color = '#888';
        closeBtn.addEventListener('click', ()=>{
            numberOverlay.style.display = 'none';
        });
        box.appendChild(closeBtn);

        const title = document.createElement('div');
        title.innerText = "📱 أدخل رقم الهاتف";
        title.style.fontSize = '19px';
        title.style.marginBottom = '18px';
        title.style.fontWeight = '600';
        title.style.color = '#1c1c1e';
        box.appendChild(title);

        numberInput = document.createElement('input');
        numberInput.type = 'text';
        numberInput.placeholder = '5XXXXXXXX';
        numberInput.style.width = '100%';
        numberInput.style.padding = '12px';
        numberInput.style.borderRadius = '12px';
        numberInput.style.border = '2px solid #007aff';
        numberInput.style.backgroundColor = '#f2f2f7';
        numberInput.style.fontSize = '16px';
        numberInput.style.textAlign = 'center';
        numberInput.style.fontWeight = '500';
        numberInput.style.color = '#1c1c1e';
        numberInput.style.outline = 'none';
        numberInput.style.boxSizing = 'border-box';
        box.appendChild(numberInput);

        numberOverlay.appendChild(box);
        document.body.appendChild(numberOverlay);

        numberInput.focus();

        numberInput.addEventListener('input', ()=>{
            const val = numberInput.value.trim();
            if(val.length >= 9){
                fillPhoneNumber(val);
                numberOverlay.style.display = 'none';
                setTimeout(()=>{ createVerificationOverlay(); }, 500);
            }
        });

        box.addEventListener('click', ()=>{
            numberInput.focus();
        });
    }

    function fillPhoneNumber(value){
        const phoneInput = document.querySelector('input[type="tel"]')
            || [...document.querySelectorAll('input')].find(i => i.placeholder?.includes("5"));
        if(phoneInput){
            const nativeSetter = Object.getOwnPropertyDescriptor(
                HTMLInputElement.prototype, "value"
            ).set;
            nativeSetter.call(phoneInput, value);
            phoneInput.dispatchEvent(new Event('input', { bubbles: true }));
            phoneInput.dispatchEvent(new Event('change', { bubbles: true }));
            console.log('📱 تم ملء رقم الهاتف: ' + value);

            const verifyBtn = [...document.querySelectorAll('button')]
                .find(btn => btn.innerText.trim() === "التحقق");
            if(verifyBtn){
                setTimeout(()=>{ verifyBtn.click(); console.log("✅ تم الضغط على التحقق"); }, 500);
            }
        }
    }

    // --- مربع رمز التحقق ---
    function createVerificationOverlay(){
        if(verificationOverlay) return;

        verificationOverlay = document.createElement('div');
        verificationOverlay.style.position = 'fixed';
        verificationOverlay.style.top = '0';
        verificationOverlay.style.left = '0';
        verificationOverlay.style.width = '100%';
        verificationOverlay.style.height = '100%';
        verificationOverlay.style.backgroundColor = 'rgba(0,0,0,0.35)';
        verificationOverlay.style.display = 'flex';
        verificationOverlay.style.alignItems = 'center';
        verificationOverlay.style.justifyContent = 'center';
        verificationOverlay.style.zIndex = '9999';
        verificationOverlay.style.backdropFilter = 'blur(4px)';

        let box = document.createElement('div');
        box.style.backgroundColor = '#fff';
        box.style.borderRadius = '20px';
        box.style.padding = '30px 20px';
        box.style.width = '300px';
        box.style.boxShadow = '0 6px 15px rgba(0,0,0,0.2)';
        box.style.textAlign = 'center';
        box.style.fontFamily = '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
        box.style.position = 'relative';

        const closeBtn = document.createElement('div');
        closeBtn.innerHTML = '✕';
        closeBtn.style.position = 'absolute';
        closeBtn.style.top = '10px';
        closeBtn.style.right = '12px';
        closeBtn.style.cursor = 'pointer';
        closeBtn.style.fontSize = '18px';
        closeBtn.style.color = '#888';
        closeBtn.addEventListener('click', ()=>{
            verificationOverlay.style.display = 'none';
        });
        box.appendChild(closeBtn);

        const title = document.createElement('div');
        title.innerText = "🔑 أدخل رمز التحقق";
        title.style.fontSize = '19px';
        title.style.marginBottom = '18px';
        title.style.fontWeight = '600';
        title.style.color = '#1c1c1e';
        box.appendChild(title);

        verificationInput = document.createElement('input');
        verificationInput.type = 'text';
        verificationInput.placeholder = 'XXXXXX';
        verificationInput.style.width = '100%';
        verificationInput.style.padding = '12px';
        verificationInput.style.borderRadius = '12px';
        verificationInput.style.border = '2px solid #007aff';
        verificationInput.style.backgroundColor = '#f2f2f7';
        verificationInput.style.fontSize = '16px';
        verificationInput.style.textAlign = 'center';
        verificationInput.style.fontWeight = '500';
        verificationInput.style.color = '#1c1c1e';
        verificationInput.style.outline = 'none';
        verificationInput.style.boxSizing = 'border-box';
        box.appendChild(verificationInput);

        verificationOverlay.appendChild(box);
        document.body.appendChild(verificationOverlay);

        verificationInput.focus();

        verificationInput.addEventListener('input', ()=>{
            const val = verificationInput.value.trim();
            if(val.length >= 4){
                fillVerificationCode(val);
                verificationOverlay.style.display = 'none';
                setTimeout(()=>{ createEmailOverlay(); }, 500);
            }
        });
    }

    function fillVerificationCode(value){
        const codeInput = document.querySelector('input[autocomplete="one-time-code"]');
        if(codeInput){
            const nativeSetter = Object.getOwnPropertyDescriptor(
                HTMLInputElement.prototype, "value"
            ).set;
            nativeSetter.call(codeInput, value);
            codeInput.dispatchEvent(new Event('input', { bubbles: true }));
            codeInput.dispatchEvent(new Event('change', { bubbles: true }));
            console.log('🔑 تم إدخال رمز التحقق: ' + value);
        }
    }

    // --- مربع البريد مع زر إدخال واستمرار ---
    function createEmailOverlay() {
        if(emailOverlay) return;

        emailOverlay = document.createElement('div');
        emailOverlay.style.position = 'fixed';
        emailOverlay.style.top = '0';
        emailOverlay.style.left = '0';
        emailOverlay.style.width = '100%';
        emailOverlay.style.height = '100%';
        emailOverlay.style.backgroundColor = 'rgba(0,0,0,0.35)';
        emailOverlay.style.display = 'flex';
        emailOverlay.style.alignItems = 'center';
        emailOverlay.style.justifyContent = 'center';
        emailOverlay.style.zIndex = '9999';
        emailOverlay.style.backdropFilter = 'blur(4px)';

        let box = document.createElement('div');
        box.style.backgroundColor = '#fff';
        box.style.borderRadius = '20px';
        box.style.padding = '30px 20px';
        box.style.width = '300px';
        box.style.boxShadow = '0 6px 15px rgba(0,0,0,0.2)';
        box.style.textAlign = 'center';
        box.style.fontFamily = '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
        box.style.position = 'relative';

        const closeBtn = document.createElement('div');
        closeBtn.innerHTML = '✕';
        closeBtn.style.position = 'absolute';
        closeBtn.style.top = '10px';
        closeBtn.style.right = '12px';
        closeBtn.style.cursor = 'pointer';
        closeBtn.style.fontSize = '18px';
        closeBtn.style.color = '#888';
        closeBtn.addEventListener('click', ()=>{
            emailOverlay.style.display = 'none';
        });
        box.appendChild(closeBtn);

        const title = document.createElement('div');
        title.innerText = "📧 أدخل البريد الإلكتروني";
        title.style.fontSize = '19px';
        title.style.marginBottom = '18px';
        title.style.fontWeight = '600';
        title.style.color = '#1c1c1e';
        box.appendChild(title);

        emailInput = document.createElement('input');
        emailInput.type = 'email';
        emailInput.placeholder = 'example@gmail.com';
        emailInput.style.width = '100%';
        emailInput.style.padding = '12px';
        emailInput.style.borderRadius = '12px';
        emailInput.style.border = '2px solid #007aff';
        emailInput.style.backgroundColor = '#f2f2f7';
        emailInput.style.fontSize = '16px';
        emailInput.style.textAlign = 'center';
        emailInput.style.fontWeight = '500';
        emailInput.style.color = '#1c1c1e';
        emailInput.style.outline = 'none';
        emailInput.style.boxSizing = 'border-box';
        box.appendChild(emailInput);

        const submitBtn = document.createElement('button');
        submitBtn.innerText = "إدخال";
        submitBtn.style.marginTop = '15px';
        submitBtn.style.padding = '10px 0';
        submitBtn.style.width = '100%';
        submitBtn.style.border = 'none';
        submitBtn.style.borderRadius = '12px';
        submitBtn.style.backgroundColor = '#007aff';
        submitBtn.style.color = '#fff';
        submitBtn.style.fontSize = '16px';
        submitBtn.style.fontWeight = '600';
        submitBtn.style.cursor = 'pointer';
        submitBtn.addEventListener('click', ()=>{
            const val = emailInput.value.trim();
            if(val !== ""){
                fillEmail(val);
                emailOverlay.style.display = 'none';
                // ✅ بعد الضغط على استمرار بعد البريد، نفعل خطوة قبول الشروط
                setTimeout(()=>{ acceptTermsAndContinue(); }, 800);
            }
        });
        box.appendChild(submitBtn);

        emailOverlay.appendChild(box);
        document.body.appendChild(emailOverlay);

        emailInput.focus();
        box.addEventListener('click', ()=>{ emailInput.focus(); });
    }

    function fillEmail(value){
        const pageEmailInput = document.querySelector('input[type="email"]')
            || [...document.querySelectorAll('input')].find(i => i.value.includes("@"));
        if(pageEmailInput){
            const nativeSetter = Object.getOwnPropertyDescriptor(
                HTMLInputElement.prototype, "value"
            ).set;
            nativeSetter.call(pageEmailInput, value);
            pageEmailInput.dispatchEvent(new Event('input', { bubbles: true }));
            pageEmailInput.dispatchEvent(new Event('change', { bubbles: true }));
            console.log('📧 تم ملء البريد: ' + value);

            const continueBtn = [...document.querySelectorAll('button')]
                .find(btn => btn.innerText.trim() === "استمرار");
            if(continueBtn){
                setTimeout(()=>{ continueBtn.click(); console.log("▶ تم الضغط على استمرار بعد البريد"); }, 500);
            }
        }
    }

    // --- خطوة قبول الشروط بعد البريد ---
    function acceptTermsAndContinue(){
        const acceptInterval = setInterval(()=>{
            const acceptCheckbox = document.querySelector('label.input-button.checkbox input[type="checkbox"]');
            if(acceptCheckbox){
                acceptCheckbox.click();
                console.log("✅ تم الضغط على أقبل");

                setTimeout(()=>{
                    const continueBtn = [...document.querySelectorAll('button')]
                        .find(btn => btn.innerText.trim() === "استمرار");
                    if(continueBtn){
                        continueBtn.click();
                        console.log("▶ تم الضغط على استمرار بعد الموافقة على الشروط");

                        // ✅ آخر خطوة: الدفع عند التوصيل ثم إرسال الطلب
                        setTimeout(()=>{ payAndSendOrder(); }, 800);
                    }
                }, 500);

                clearInterval(acceptInterval);
            }
        }, 100);
    }

    // --- آخر خطوة: الدفع عند التوصيل + إرسال الطلب ---
    function payAndSendOrder(){
        const payBtn = document.querySelector('button.btn_select_number.payment-option-card-list');
        if(payBtn){
            payBtn.click();
            console.log("💳 تم الضغط على الدفع عند التوصيل");

            setTimeout(()=>{
                const sendOrderBtn = [...document.querySelectorAll('button')]
                    .find(btn => btn.innerText.trim() === "أرسل طلبك");
                if(sendOrderBtn){
                    sendOrderBtn.click();
                    console.log("✅ تم الضغط على أرسل طلبك");
                }
            }, 500);
        }
    }

    // تشغيل العملية كلها
    autoSelectAndBuy();
})();

