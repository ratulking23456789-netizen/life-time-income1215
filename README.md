<!doctype html>
<html lang="bn">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Fake OTP Call Simulator — নিরাপদ সিমুলেশন</title>
<style>
  :root{--bg:#0b1220;--card:#0f1724;--accent:#06b6d4;--text:#e6f0f6;--muted:#9fb3bd}
  *{box-sizing:border-box}
  body{margin:0;font-family:Inter, "Noto Sans Bengali", system-ui;background:linear-gradient(180deg,#071021,#071022);color:var(--text);min-height:100vh;display:flex;align-items:center;justify-content:center;padding:20px}
  .wrap{max-width:760px;width:100%}
  .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);padding:20px;border-radius:12px;box-shadow:0 10px 30px rgba(2,6,23,0.6)}
  h1{margin:0 0 8px;font-size:20px}
  p{margin:0 0 12px;color:var(--muted)}
  .controls{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:12px}
  input,select,button{padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--text);outline:none}
  button{cursor:pointer;background:linear-gradient(90deg,var(--accent),#60f7ff);color:#022;font-weight:700;border:0}
  .phone-preview{display:flex;align-items:center;gap:12px;margin-top:12px}
  .phone{width:160px;height:320px;border-radius:18px;background:#071426;display:flex;flex-direction:column;align-items:center;padding:12px;border:1px solid rgba(255,255,255,0.03)}
  .screen{flex:1;width:100%;display:flex;align-items:center;justify-content:center;flex-direction:column;gap:8px}
  .incoming{background:linear-gradient(90deg,#ff7b7b,#ffb86b);padding:10px 14px;border-radius:10px;color:#071018;font-weight:800}
  .otp{font-size:24px;letter-spacing:6px;font-weight:800}
  .actions{display:flex;gap:8px;margin-top:10px}
  .muted{color:var(--muted);font-size:13px;margin-top:8px}
  .log{margin-top:12px;background:rgba(255,255,255,0.02);padding:10px;border-radius:8px;max-height:160px;overflow:auto;font-size:13px}
  @media(max-width:640px){ .phone{width:120px;height:260px} }
</style>
</head>
<body>
  <div class="wrap">
    <div class="card" role="main" aria-labelledby="title">
      <h1 id="title">Fake OTP Call Simulator — নিরাপদ সিমুলেশন</h1>
      <p>এই সিমুলেটরটি **কোনো রিয়েল SMS/কল পাঠাবে না** — শুধুই তোমার ব্রাউজারে একটি ইনকামিং কলের UI, ভয়েস (Text-to-Speech) এবং OTP ভেরিফিকেশন সিমুলেট করবে। বন্ধুদের সঙ্গে মজা করতে আগে অবশ্যই তাদের সম্মতি নেবো।</p>

      <div class="controls" aria-hidden="false">
        <input id="displayName" placeholder="Caller নাম (যেমন: Bank OTP)" value="Bank OTP" />
        <input id="phoneNumber" placeholder="ফোন নম্বর (ডেমো)" value="+88 01712345678" />
        <select id="voiceLang">
          <option value="bn-BD">Bengali (bd) — bn-BD</option>
          <option value="en-US">English (US) — en-US</option>
          <option value="en-GB">English (GB) — en-GB</option>
        </select>
        <button id="startCall">Simulate Call</button>
        <button id="stopCall" disabled>End Call</button>
      </div>

      <div style="display:flex;gap:16px;flex-wrap:wrap;align-items:flex-start">
        <div class="phone" aria-hidden="false">
          <div class="screen" id="screen">
            <div style="text-align:center;color:var(--muted);font-size:13px">Phone</div>
            <div id="incomingBox" style="display:none;align-items:center;flex-direction:column">
              <div class="incoming" id="callerName">Bank OTP</div>
              <div class="muted" id="callerNumber">+88 01712345678</div>
              <div style="height:8px"></div>
              <div class="otp" id="otpCode">••••</div>
              <div class="actions">
                <button id="answerBtn">Answer</button>
                <button id="declineBtn" style="background:transparent;color:var(--text);border:1px solid rgba(255,255,255,0.04)">Decline</button>
              </div>
            </div>

            <div id="idleBox" style="display:flex;flex-direction:column;align-items:center">
              <div style="font-weight:700;font-size:18px">Awaiting</div>
              <div class="muted">Simulate an incoming OTP call</div>
            </div>
          </div>
        </div>

        <div style="flex:1;min-width:220px">
          <div>
            <label class="muted">Generated OTP:</label>
            <div style="font-size:20px;font-weight:800;margin:6px 0" id="generatedOtp">—</div>
            <div style="display:flex;gap:8px">
              <button id="regenOtp" type="button">Regenerate OTP</button>
              <button id="ttsOtp" type="button">Play OTP Voice</button>
            </div>
            <div class="muted">ভয়েসটি ব্রাউজারের Text-to-Speech ব্যবহার করে বাজবে (ছন্দ-সই)।</div>
          </div>

          <div style="margin-top:12px">
            <label class="muted">Verify OTP (enter):</label>
            <div style="display:flex;gap:8px;margin-top:6px">
              <input id="verifyInput" placeholder="OTP code" />
              <button id="verifyBtn" type="button">Verify</button>
            </div>
            <div id="verifyMsg" class="muted"></div>
          </div>

          <div class="log" id="log" aria-live="polite">
            <div><strong>Log:</strong></div>
          </div>
        </div>
      </div>

    </div>
  </div>

<script>
(function(){
  const startCall = document.getElementById('startCall');
  const stopCall = document.getElementById('stopCall');
  const incomingBox = document.getElementById('incomingBox');
  const idleBox = document.getElementById('idleBox');
  const callerName = document.getElementById('callerName');
  const callerNumber = document.getElementById('callerNumber');
  const otpCodeEl = document.getElementById('otpCode');
  const generatedOtpEl = document.getElementById('generatedOtp');
  const regenOtp = document.getElementById('regenOtp');
  const ttsOtp = document.getElementById('ttsOtp');
  const answerBtn = document.getElementById('answerBtn');
  const declineBtn = document.getElementById('declineBtn');
  const displayName = document.getElementById('displayName');
  const phoneNumber = document.getElementById('phoneNumber');
  const voiceLang = document.getElementById('voiceLang');
  const logEl = document.getElementById('log');
  const verifyInput = document.getElementById('verifyInput');
  const verifyBtn = document.getElementById('verifyBtn');
  const verifyMsg = document.getElementById('verifyMsg');
  const stopCallBtn = document.getElementById('stopCall');

  let currentOtp = null;
  let callTimer = null;
  let isRinging = false;

  function log(text){
    const time = new Date().toLocaleTimeString();
    const d = document.createElement('div');
    d.textContent = `[${time}] ${text}`;
    logEl.appendChild(d);
    logEl.scrollTop = logEl.scrollHeight;
  }

  function genOtp(){
    currentOtp = Math.floor(100000 + Math.random()*900000).toString();
    generatedOtpEl.textContent = currentOtp;
    otpCodeEl.textContent = '••••';
    log('Generated OTP: ' + currentOtp);
  }

  function showIncoming(){
    callerName.textContent = displayName.value || 'Unknown';
    callerNumber.textContent = phoneNumber.value || '—';
    incomingBox.style.display = 'flex';
    idleBox.style.display = 'none';
    isRinging = true;
    startCall.disabled = true;
    stopCall.disabled = false;
    log('Incoming simulated call from ' + callerName.textContent + ' (' + callerNumber.textContent + ')');
    // auto-play ringing beep (short)
    playRingtoneLoop();
    // auto-stop after 30s if not answered
    callTimer = setTimeout(() => {
      if(isRinging){ endCall('Missed (no answer)'); }
    }, 30000);
  }

  function endCall(reason = 'Ended'){
    isRinging = false;
    incomingBox.style.display = 'none';
    idleBox.style.display = 'flex';
    startCall.disabled = false;
    stopCall.disabled = true;
    stopRingtone();
    if(callTimer) { clearTimeout(callTimer); callTimer = null; }
    log('Call ' + reason);
  }

  // Ringtone using WebAudio simple oscillator for short beep loops
  let audioCtx = null, osc = null, ringtoneInterval = null;
  function playRingtoneLoop(){
    if(!window.AudioContext && !window.webkitAudioContext) return;
    if(audioCtx) return;
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    let gain = audioCtx.createGain();
    gain.gain.value = 0.02;
    gain.connect(audioCtx.destination);
    function ringOnce(){
      let o = audioCtx.createOscillator();
      o.type = 'sine';
      o.frequency.value = 440;
      o.connect(gain);
      o.start();
      setTimeout(()=>{ o.stop() }, 350);
    }
    ringOnce();
    ringtoneInterval = setInterval(ringOnce, 800);
  }
  function stopRingtone(){
    if(ringtoneInterval) { clearInterval(ringtoneInterval); ringtoneInterval = null; }
    if(audioCtx){ audioCtx.close(); audioCtx = null; }
  }

  // Text-to-Speech OTP readout
  function speakOTP(){
    if(!('speechSynthesis' in window)){
      alert('Your browser does not support speechSynthesis.');
      return;
    }
    if(!currentOtp) genOtp();
    const lang = voiceLang.value || 'bn-BD';
    // create a friendly phrase (in selected language)
    let phrase = '';
    if(lang.startsWith('bn')) phrase = `তোমার ওটিপি নম্বর ${currentOtp.split('').join(' ')}.`;
    else phrase = `Your OTP code is ${currentOtp.split('').join(' ')}.`;
    const u = new SpeechSynthesisUtterance(phrase);
    u.lang = lang;
    u.rate = 0.95;
    window.speechSynthesis.cancel();
    window.speechSynthesis.speak(u);
    log('Played OTP voice (lang: ' + lang + ')');
  }

  // Event bindings
  startCall.addEventListener('click', () => {
    genOtp();
    showIncoming();
  });
  stopCall.addEventListener('click', () => {
    endCall('Manually ended');
  });
  regenOtp.addEventListener('click', () => {
    genOtp();
  });
  ttsOtp.addEventListener('click', () => {
    speakOTP();
  });
  answerBtn.addEventListener('click', () => {
    if(!isRinging) return;
    stopRingtone();
    // show OTP on screen when answered
    otpCodeEl.textContent = currentOtp || '—';
    log('Call answered — OTP shown on screen');
    // also speak OTP
    speakOTP();
    // after a bit, auto end
    setTimeout(()=> endCall('Completed'), 6000);
  });
  declineBtn.addEventListener('click', () => {
    if(!isRinging) return;
    endCall('Declined');
  });

  verifyBtn.addEventListener('click', () => {
    const v = (verifyInput.value || '').trim();
    if(!currentOtp){ verifyMsg.textContent = 'OTP জেনারেট করা হয়নি।'; return; }
    if(v === currentOtp){
      verifyMsg.textContent = 'ভেরিফায়েড ✓';
      log('OTP verified successfully');
    } else {
      verifyMsg.textContent = 'ভুল OTP — আবার চেষ্টা করো।';
      log('OTP verification failed (entered: ' + v + ')');
    }
    setTimeout(()=>{ verifyMsg.textContent = '' }, 3000);
  });

  // keyboard: Enter to verify when input focused
  verifyInput.addEventListener('keydown', (e) => {
    if(e.key === 'Enter') verifyBtn.click();
  });

  // initialize
  genOtp();
})();
</script>
</body>
</html>
