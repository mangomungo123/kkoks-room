<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<title>단수 카운터</title>
<style>
  :root{
    --bg:#211F26; --surface:#2A2830; --surface-2:#34313B; --surface-3:#3E3A46;
    --ink:#F2EFE9; --ink-muted:#ABA6B3; --ink-faint:#726D7A;
    --accent:#D9A24B; --accent-ink:#211F26; --highlight:#6FAE9F; --danger:#C1666B; --border:#413D48;
  }
  *{box-sizing:border-box; -webkit-tap-highlight-color:transparent;}
  html,body{ margin:0; padding:0; height:100%; background:var(--bg); color:var(--ink);
    font-family:-apple-system,BlinkMacSystemFont,"SF Pro Text","Apple SD Gothic Neo",system-ui,sans-serif;
    overscroll-behavior:none; }
  #app{ display:flex; flex-direction:column; height:100dvh; max-width:640px; margin:0 auto; position:relative; }

  .icon-btn{ width:34px; height:34px; border-radius:10px; border:1px solid var(--border); background:var(--surface); color:var(--ink-muted); display:flex; align-items:center; justify-content:center; font-size:16px; cursor:pointer; }
  .icon-btn:active{ background:var(--surface-2); }

  #tabbar{ flex:0 0 auto; display:flex; gap:6px; padding:8px 12px; border-bottom:1px solid var(--border); background:var(--surface); }
  #tabbar button{ flex:1; padding:9px 0; border-radius:10px; border:1px solid var(--border); background:var(--surface-2); color:var(--ink-muted); font-size:13.5px; font-weight:600; cursor:pointer; }
  #tabbar button.active{ background:var(--accent); color:var(--accent-ink); border-color:var(--accent); }

  #view-screen{ flex:1 1 auto; min-height:0; display:flex; flex-direction:column; overflow:hidden; }
  #empty-state{ flex:1; display:flex; flex-direction:column; align-items:center; justify-content:center; padding:32px 24px; text-align:center; gap:14px; color:var(--ink-muted); }
  #empty-state p{ margin:0; font-size:14px; line-height:1.6; max-width:280px; }
  .primary-btn{ background:var(--accent); color:var(--accent-ink); border:none; border-radius:12px; padding:13px 22px; font-size:15px; font-weight:600; cursor:pointer; }
  .primary-btn:active{ opacity:0.85; }
  .ghost-btn{ background:transparent; color:var(--ink-muted); border:1px solid var(--border); border-radius:12px; padding:9px 16px; font-size:13px; cursor:pointer; }

  #page-toolbar{ flex:0 0 auto; display:flex; flex-direction:column; gap:6px; padding:8px 12px; border-bottom:1px solid var(--border); background:var(--surface); }
  .toolbar-row{ display:flex; align-items:center; gap:8px; }
  .pagenav{ display:flex; align-items:center; gap:6px; }
  button.nav{ width:30px; height:30px; border-radius:8px; border:1px solid var(--border); background:var(--surface-2); color:var(--ink); font-size:14px; cursor:pointer; }
  .pagelabel{ font-size:13px; color:var(--accent); min-width:52px; text-align:center; cursor:pointer; text-decoration:underline dotted; text-underline-offset:3px; }
  .tool-btn{ font-size:12px; padding:6px 10px; border-radius:8px; border:1px solid var(--border); background:var(--surface-2); color:var(--ink-muted); cursor:pointer; white-space:nowrap; }
  .tool-btn.active{ background:var(--accent); color:var(--accent-ink); border-color:var(--accent); }
  .zoom-group{ display:flex; align-items:center; gap:4px; margin-left:auto; }
  .zoom-group button{ width:28px; height:28px; border-radius:8px; border:1px solid var(--border); background:var(--surface-2); color:var(--ink); font-size:14px; cursor:pointer; }
  .zoom-label{ font-size:12px; color:var(--ink-muted); min-width:38px; text-align:center; }

  #canvas-scroll{ flex:1 1 auto; overflow:auto; background:#17161B; display:flex; justify-content:center; padding:14px 10px 24px; position:relative; }
  #canvas-scroll.no-scroll{ overflow:hidden; touch-action:none; }
  #canvas-holder{ position:relative; line-height:0; touch-action:manipulation; margin:auto; -webkit-touch-callout:none; }
  #canvas-holder.no-scroll{ touch-action:none; }
  #pdf-canvas{ display:block; border-radius:2px; box-shadow:0 0 0 1px rgba(255,255,255,0.06); }
  #highlight-band{ position:absolute; left:0; right:0; pointer-events:none; background:rgba(111,174,159,0.38); transition:top 0.16s ease, height 0.16s ease; display:none; }
  #select-box{ position:absolute; border:2px dashed var(--accent); background:rgba(217,162,75,0.14); pointer-events:none; display:none; }
  #inline-banner{ position:fixed; left:8px; right:8px; top:140px; background:rgba(33,31,38,0.94); border:1px solid var(--accent); border-radius:10px; padding:10px 12px; font-size:12.5px; color:var(--ink); display:none; line-height:1.55; z-index:15; }
  #inline-banner button{ margin-top:8px; }
  #toast{ position:fixed; left:50%; top:140px; transform:translateX(-50%); background:rgba(33,31,38,0.94); border:1px solid var(--border); color:var(--ink); font-size:12.5px; padding:7px 13px; border-radius:20px; opacity:0; transition:opacity 0.2s ease; pointer-events:none; z-index:16; white-space:nowrap; }
  #toast.show{ opacity:1; }

  #calib-bar{ position:fixed; left:8px; right:8px; top:140px; z-index:18; background:rgba(33,31,38,0.96); border:1px solid var(--accent); border-radius:12px; padding:10px 12px; display:none; flex-direction:column; gap:9px; }
  #calib-bar .calib-hint{ font-size:11.5px; color:var(--ink-muted); line-height:1.4; }
  #calib-bar .calib-controls{ display:flex; align-items:center; gap:8px; }
  .calib-step{ width:34px; height:34px; flex:0 0 auto; border-radius:9px; border:1px solid var(--border); background:var(--surface-2); color:var(--ink); font-size:18px; display:flex; align-items:center; justify-content:center; cursor:pointer; }
  #calib-slider{ flex:1; -webkit-appearance:none; appearance:none; height:4px; border-radius:2px; background:var(--surface-3); outline:none; margin:0 2px; }
  #calib-slider::-webkit-slider-thumb{ -webkit-appearance:none; width:26px; height:26px; border-radius:50%; background:var(--accent); border:3px solid var(--accent-ink); box-shadow:0 0 0 1px var(--accent); cursor:pointer; }
  #calib-slider::-moz-range-thumb{ width:26px; height:26px; border-radius:50%; background:var(--accent); border:3px solid var(--accent-ink); cursor:pointer; }
  .calib-bottom-row{ display:flex; align-items:center; justify-content:space-between; gap:8px; }
  #calib-count-label{ font-size:14.5px; font-weight:700; cursor:pointer; }
  .calib-btns{ display:flex; gap:8px; }
  .calib-confirm{ background:var(--accent); color:var(--accent-ink); border:none; border-radius:9px; padding:8px 13px; font-size:13px; font-weight:600; cursor:pointer; }
  .calib-cancel{ background:transparent; color:var(--ink-muted); border:1px solid var(--border); border-radius:9px; padding:8px 11px; font-size:13px; cursor:pointer; }
  #calib-lines{ position:absolute; pointer-events:none; display:none; z-index:4; }
  .calib-line{ position:absolute; left:0; right:0; height:0; border-top:2px solid var(--accent); box-shadow:0 0 0 1px rgba(0,0,0,0.4); }

  #count-screen{ flex:1 1 auto; min-height:0; overflow-y:auto; padding:22px 18px calc(18px + env(safe-area-inset-bottom)); display:none; }
  .count-top-row{ display:flex; align-items:center; gap:8px; margin-bottom:16px; }
  #progress-track{ height:5px; background:var(--surface-2); border-radius:3px; overflow:hidden; margin-bottom:22px; display:none; }
  #progress-fill{ height:100%; background:var(--accent); width:0%; }
  .main-counter{ display:flex; align-items:center; justify-content:center; gap:22px; padding:10px 0 26px; }
  .step-btn{ width:58px; height:58px; border-radius:18px; border:1px solid var(--border); background:var(--surface-2); color:var(--ink); font-size:24px; display:flex; align-items:center; justify-content:center; cursor:pointer; user-select:none; }
  .step-btn:active{ background:var(--surface-3); }
  #row-tap{ display:flex; flex-direction:column; align-items:center; justify-content:center; min-width:130px; cursor:pointer; user-select:none; padding:2px 8px; }
  #row-count{ font-size:56px; font-weight:700; letter-spacing:-1.5px; line-height:1; color:var(--ink); font-variant-numeric:tabular-nums; }
  #row-caption{ font-size:12.5px; color:var(--ink-muted); margin-top:5px; }
  #section-label{ font-size:12.5px; color:var(--ink-faint); margin:0 2px 10px; }
  #pattern-counters{ display:flex; flex-wrap:wrap; gap:10px; }
  .pc-chip{ background:var(--surface-2); border:1px solid var(--border); border-radius:14px; padding:10px 12px; display:flex; flex-direction:column; align-items:center; gap:7px; width:calc(33.333% - 7px); }
  .pc-label{ font-size:11.5px; color:var(--ink-muted); max-width:100%; overflow:hidden; text-overflow:ellipsis; white-space:nowrap; }
  .pc-row{ display:flex; align-items:center; gap:8px; }
  .pc-count{ font-size:19px; font-weight:700; min-width:22px; text-align:center; font-variant-numeric:tabular-nums; }
  .pc-mini-btn{ width:27px; height:27px; border-radius:8px; border:1px solid var(--border); background:var(--surface-3); color:var(--ink); font-size:14px; display:flex; align-items:center; justify-content:center; cursor:pointer; }
  .pc-add{ width:calc(33.333% - 7px); border-radius:14px; border:1px dashed var(--ink-faint); background:transparent; color:var(--ink-faint); font-size:12.5px; display:flex; align-items:center; justify-content:center; cursor:pointer; min-height:74px; }

  dialog{ border:none; border-radius:16px; background:var(--surface); color:var(--ink); padding:20px; width:min(320px, 86vw); box-shadow:0 12px 40px rgba(0,0,0,0.5); }
  dialog::backdrop{ background:rgba(0,0,0,0.55); }
  dialog h3{ margin:0 0 4px; font-size:16px; }
  dialog p.hint{ margin:0 0 14px; font-size:12.5px; color:var(--ink-muted); line-height:1.5; }
  dialog label{ font-size:12.5px; color:var(--ink-muted); display:block; margin:10px 0 5px; }
  dialog input[type=text], dialog input[type=number]{ width:100%; background:var(--surface-2); border:1px solid var(--border); border-radius:9px; color:var(--ink); font-size:15px; padding:9px 10px; }
  .dialog-actions{ display:flex; gap:8px; margin-top:18px; justify-content:flex-end; }
  .dialog-actions button{ border-radius:10px; padding:9px 16px; font-size:14px; border:none; cursor:pointer; }
  .dialog-actions .cancel{ background:transparent; color:var(--ink-muted); border:1px solid var(--border); }
  .dialog-actions .confirm{ background:var(--accent); color:var(--accent-ink); font-weight:600; }
  .dialog-actions .delete{ background:transparent; color:var(--danger); border:1px solid var(--danger); }

  #file-input{ display:none; }
</style>
</head>
<body>
<div id="app">

  <div id="tabbar">
    <button id="tab-view" class="active">도안</button>
    <button id="tab-count">카운터</button>
  </div>

  <div id="view-screen">
    <div id="empty-state">
      <p>PDF 도안을 불러온 뒤 영역을 드래그해 줄/칸 수를 맞추면, 하이라이트 위쪽을 탭할 땐 위로, 아래쪽을 탭할 땐 아래로 이동해요.</p>
      <button class="primary-btn" id="load-pdf-btn">PDF 도안 불러오기</button>
    </div>

    <div id="page-toolbar" style="display:none;">
      <div class="toolbar-row">
        <div class="pagenav">
          <button class="nav" id="prev-page">‹</button>
          <span class="pagelabel" id="page-label">1 / 1</span>
          <button class="nav" id="next-page">›</button>
        </div>
        <div class="zoom-group">
          <button id="zoom-out">－</button>
          <span class="zoom-label" id="zoom-label">100%</span>
          <button id="zoom-in">＋</button>
        </div>
      </div>
      <div class="toolbar-row">
        <button class="tool-btn" id="resel-btn">영역 재지정</button>
        <button class="tool-btn active" id="highlight-toggle">하이라이트 켬</button>
        <button class="icon-btn" id="reload-pdf-btn" title="도안 교체" style="margin-left:auto;">📄</button>
      </div>
    </div>

    <div id="canvas-scroll" style="display:none;">
      <div id="canvas-holder">
        <canvas id="pdf-canvas"></canvas>
        <div id="highlight-band"></div>
        <div id="select-box"></div>
        <div id="calib-lines"></div>
        <div id="inline-banner"></div>
        <div id="calib-bar">
          <div class="calib-hint">슬라이더로 선을 도안의 실제 칸/줄 경계에 맞춘 뒤 완료를 누르세요</div>
          <div class="calib-controls">
            <button class="calib-step" id="calib-minus">－</button>
            <input type="range" id="calib-slider" min="1" max="80" value="10" step="1">
            <button class="calib-step" id="calib-plus">＋</button>
          </div>
          <div class="calib-bottom-row">
            <div id="calib-count-label">10단</div>
            <div class="calib-btns">
              <button class="calib-cancel" id="calib-cancel">취소</button>
              <button class="calib-confirm" id="calib-confirm">완료</button>
            </div>
          </div>
        </div>
        <div id="toast"></div>
      </div>
    </div>
  </div>

  <div id="count-screen">
    <div class="count-top-row">
      <button class="tool-btn" id="target-btn">목표 단수</button>
      <button class="tool-btn" id="reset-btn" style="margin-left:auto;">초기화</button>
    </div>
    <div id="progress-track"><div id="progress-fill"></div></div>
    <div class="main-counter">
      <button class="step-btn" id="row-minus">－</button>
      <div id="row-tap">
        <div id="row-count">0</div>
        <div id="row-caption">단 · 탭하면 +1</div>
      </div>
      <button class="step-btn" id="row-plus">＋</button>
    </div>
    <div id="section-label">무늬 카운터</div>
    <div id="pattern-counters"></div>
  </div>

</div>

<input type="file" id="file-input" accept="application/pdf">

<dialog id="page-dialog">
  <h3>페이지 이동</h3>
  <p class="hint">이동할 페이지 번호를 입력하세요 (1 ~ <span id="page-dialog-max">1</span>)</p>
  <label>페이지 번호</label>
  <input type="number" id="page-dialog-input" min="1" value="1">
  <div class="dialog-actions">
    <button class="cancel" id="page-dialog-cancel">취소</button>
    <button class="confirm" id="page-dialog-confirm">이동</button>
  </div>
</dialog>

<dialog id="manual-dialog">
  <h3>단 수 빠르게 입력</h3>
  <p class="hint">숫자를 입력하면 미리보기 선 개수가 바로 바뀌어요. 완료를 눌러야 최종 반영됩니다.</p>
  <label>총 단 수</label>
  <input type="number" id="manual-total" min="1" value="10">
  <div class="dialog-actions">
    <button class="cancel" id="manual-cancel">취소</button>
    <button class="confirm" id="manual-confirm">적용</button>
  </div>
</dialog>

<dialog id="pc-dialog">
  <h3 id="pc-dialog-title">무늬 카운터 추가</h3>
  <p class="hint">예: 꽈배기, 무늬 반복, 감소단 등 메인 단수와 별개로 세고 싶은 항목이에요.</p>
  <label>이름</label>
  <input type="text" id="pc-name" placeholder="예: 꽈배기">
  <div class="dialog-actions">
    <button class="delete" id="pc-delete" style="display:none;">삭제</button>
    <button class="cancel" id="pc-cancel">취소</button>
    <button class="confirm" id="pc-confirm">저장</button>
  </div>
</dialog>

<dialog id="target-dialog">
  <h3>목표 단수</h3>
  <p class="hint">전체 도안의 총 단수를 입력하면 진행률이 표시돼요. 비워두면 진행률 표시가 꺼져요.</p>
  <label>목표 단수</label>
  <input type="number" id="target-input" min="1" placeholder="예: 120">
  <div class="dialog-actions">
    <button class="cancel" id="target-cancel">취소</button>
    <button class="confirm" id="target-confirm">저장</button>
  </div>
</dialog>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<script>
pdfjsLib.GlobalWorkerOptions.workerSrc = "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js";

let project = {
  rowCount: 0,
  targetRows: null,
  counters: [],
  pages: {},        // n -> {segments:[{top,bottom} 0..1], startIndex, region}
  pdfName: null,
  pdfBase64: null,
  zoom: 1,
  highlightVisible: true
};

let pdfDoc=null, currentPage=1, numPages=1, curViewport=null;
let selecting=false, dragStart=null, dragCur=null, pointerDownPos=null;
let calibActive=false, calibRegion=null, calibCount=10;
const el = id => document.getElementById(id);

/* ---------------- fixed-overlay positioning (viewport-relative, unaffected by zoom/scroll) ---------------- */
function positionFixedOverlays(){
  const appRect = el('app').getBoundingClientRect();
  const toolbar = el('page-toolbar');
  const toolbarVisible = toolbar.offsetParent !== null;
  const top = (toolbarVisible ? toolbar.getBoundingClientRect().bottom : appRect.top + 50) + 8;
  const leftPx = appRect.left + 8;
  const rightPx = (window.innerWidth - appRect.right) + 8;
  ['calib-bar','inline-banner'].forEach(id=>{
    const node = el(id);
    node.style.left = leftPx+'px'; node.style.right = rightPx+'px'; node.style.top = top+'px';
  });
  const toastEl = el('toast');
  toastEl.style.left = (appRect.left + appRect.width/2)+'px';
  toastEl.style.top = top+'px';
}
window.addEventListener('resize', positionFixedOverlays);
window.addEventListener('orientationchange', positionFixedOverlays);

/* ---------------- storage ---------------- */
async function loadProject(){
  try{
    const res = await window.storage.get('project', false);
    if(res && res.value){ project = Object.assign(project, JSON.parse(res.value)); }
  }catch(e){}
}
let saveTimer=null;
function saveProject(){
  clearTimeout(saveTimer);
  saveTimer = setTimeout(async ()=>{
    try{ await window.storage.set('project', JSON.stringify(project), false); }
    catch(e){ console.error('저장 실패', e); }
  }, 150);
}

/* ---------------- tabs ---------------- */
el('tab-view').onclick = ()=>{
  el('tab-view').classList.add('active'); el('tab-count').classList.remove('active');
  el('view-screen').style.display='flex'; el('count-screen').style.display='none';
  positionFixedOverlays();
};
el('tab-count').onclick = ()=>{
  el('tab-count').classList.add('active'); el('tab-view').classList.remove('active');
  el('view-screen').style.display='none'; el('count-screen').style.display='block';
};

/* ---------------- counter screen ---------------- */
function updateRowUI(){
  el('row-count').textContent = project.rowCount;
  if(project.targetRows && project.targetRows>0){
    el('progress-track').style.display='block';
    el('progress-fill').style.width = Math.min(100,(project.rowCount/project.targetRows)*100)+'%';
    el('row-caption').textContent = `단 · ${project.rowCount}/${project.targetRows}`;
  } else {
    el('progress-track').style.display='none';
    el('row-caption').textContent = '단 · 탭하면 +1';
  }
  renderHighlight();
}
function renderPatternCounters(){
  const wrap = el('pattern-counters'); wrap.innerHTML='';
  project.counters.forEach(c=>{
    const chip = document.createElement('div'); chip.className='pc-chip';
    chip.innerHTML = `<div class="pc-label">${escapeHtml(c.label)}</div>
      <div class="pc-row"><button class="pc-mini-btn" data-act="minus">－</button>
      <div class="pc-count">${c.count}</div><button class="pc-mini-btn" data-act="plus">＋</button></div>`;
    chip.querySelector('[data-act=minus]').onclick=(e)=>{e.stopPropagation(); c.count=Math.max(0,c.count-1); renderPatternCounters(); saveProject();};
    chip.querySelector('[data-act=plus]').onclick=(e)=>{e.stopPropagation(); c.count+=1; renderPatternCounters(); saveProject();};
    let t; chip.addEventListener('touchstart',()=>{t=setTimeout(()=>openPcDialog(c),480);});
    chip.addEventListener('touchend',()=>clearTimeout(t));
    chip.addEventListener('mousedown',()=>{t=setTimeout(()=>openPcDialog(c),480);});
    chip.addEventListener('mouseup',()=>clearTimeout(t));
    wrap.appendChild(chip);
  });
  const add=document.createElement('button'); add.className='pc-add'; add.textContent='+ 추가';
  add.onclick=()=>openPcDialog(null); wrap.appendChild(add);
}
function escapeHtml(s){ return s.replace(/[&<>"']/g,m=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m])); }

let editingCounter=null;
function openPcDialog(c){
  editingCounter=c;
  el('pc-dialog-title').textContent = c?'무늬 카운터 편집':'무늬 카운터 추가';
  el('pc-name').value = c? c.label:'';
  el('pc-delete').style.display = c?'inline-block':'none';
  el('pc-dialog').showModal();
}
el('pc-cancel').onclick=()=>el('pc-dialog').close();
el('pc-delete').onclick=()=>{ project.counters=project.counters.filter(c=>c!==editingCounter); renderPatternCounters(); saveProject(); el('pc-dialog').close(); };
el('pc-confirm').onclick=()=>{
  const name = el('pc-name').value.trim(); if(!name) return;
  if(editingCounter){ editingCounter.label=name; } else { project.counters.push({id:Date.now(), label:name, count:0}); }
  renderPatternCounters(); saveProject(); el('pc-dialog').close();
};

function changeRow(delta){
  project.rowCount = Math.max(0, project.rowCount+delta);
  updateRowUI(); saveProject();
}
el('row-plus').onclick=()=>changeRow(1);
el('row-minus').onclick=()=>changeRow(-1);
el('row-tap').onclick=()=>changeRow(1);

el('reset-btn').onclick=()=>{
  if(confirm('메인 단수와 모든 무늬 카운터를 0으로 되돌릴까요? (도안/인식 결과는 유지돼요)')){
    project.rowCount=0; project.counters.forEach(c=>c.count=0);
    updateRowUI(); renderPatternCounters(); saveProject();
  }
};
el('target-btn').onclick=()=>{ el('target-input').value = project.targetRows||''; el('target-dialog').showModal(); };
el('target-cancel').onclick=()=>el('target-dialog').close();
el('target-confirm').onclick=()=>{
  const v=parseInt(el('target-input').value,10);
  project.targetRows=(v&&v>0)?v:null;
  updateRowUI(); saveProject(); el('target-dialog').close();
};

/* ---------------- pdf load ---------------- */
el('load-pdf-btn').onclick=()=>el('file-input').click();
el('reload-pdf-btn').onclick=()=>el('file-input').click();
el('file-input').onchange = async (e)=>{
  const file=e.target.files[0]; if(!file) return;
  const buf = await file.arrayBuffer();
  await openPdfFromArrayBuffer(buf.slice(0));
  project.pdfName=file.name;
  try{
    const b64=arrayBufferToBase64(buf);
    project.pdfBase64 = (b64.length<4500000) ? b64 : null;
  }catch(err){ project.pdfBase64=null; }
  saveProject();
};
function arrayBufferToBase64(buffer){
  let binary=''; const bytes=new Uint8Array(buffer); const chunk=0x8000;
  for(let i=0;i<bytes.length;i+=chunk){ binary+=String.fromCharCode.apply(null, bytes.subarray(i,i+chunk)); }
  return btoa(binary);
}
function base64ToArrayBuffer(b64){
  const binary=atob(b64); const bytes=new Uint8Array(binary.length);
  for(let i=0;i<binary.length;i++) bytes[i]=binary.charCodeAt(i);
  return bytes.buffer;
}
async function openPdfFromArrayBuffer(buf){
  pdfDoc = await pdfjsLib.getDocument({data:buf}).promise;
  numPages = pdfDoc.numPages; currentPage=1;
  el('empty-state').style.display='none';
  el('page-toolbar').style.display='flex';
  el('canvas-scroll').style.display='flex';
  updateZoomLabel(); updateHighlightToggleBtn();
  positionFixedOverlays();
  await renderPage(currentPage);
}
async function renderPage(n){
  const page = await pdfDoc.getPage(n);
  const containerWidth = el('canvas-scroll').clientWidth - 20;
  const baseViewport = page.getViewport({scale:1});
  const zoom = project.zoom || 1;
  const scale = (containerWidth / baseViewport.width) * zoom;
  const viewport = page.getViewport({scale});
  curViewport = viewport;
  const canvas = el('pdf-canvas'); const ctx = canvas.getContext('2d');
  const dpr = window.devicePixelRatio || 1;
  canvas.width = viewport.width*dpr; canvas.height = viewport.height*dpr;
  canvas.style.width = viewport.width+'px'; canvas.style.height = viewport.height+'px';
  ctx.setTransform(dpr,0,0,dpr,0,0);
  await page.render({canvasContext:ctx, viewport}).promise;
  el('page-label').textContent = `${n} / ${numPages}`;
  cancelCalib(); exitSelecting(); hideBanner();

  const st = getPageState(n);
  if(!st.segments){
    await tryAutoDetectText(page, viewport);
  }
  renderHighlight();
}
el('prev-page').onclick=async()=>{ if(currentPage>1) await goToPage(currentPage-1); };
el('next-page').onclick=async()=>{ if(currentPage<numPages) await goToPage(currentPage+1); };
async function goToPage(n){
  n = Math.min(Math.max(1,n), numPages);
  if(n===currentPage) return;
  currentPage = n;
  await renderPage(currentPage);
}
el('page-label').onclick=()=>{
  if(!pdfDoc) return;
  el('page-dialog-max').textContent = numPages;
  const input = el('page-dialog-input');
  input.max = numPages; input.value = currentPage;
  el('page-dialog').showModal();
};
el('page-dialog-cancel').onclick=()=>el('page-dialog').close();
el('page-dialog-confirm').onclick=()=>{
  let v = parseInt(el('page-dialog-input').value,10);
  if(!v || v<1) v=1; if(v>numPages) v=numPages;
  el('page-dialog').close();
  goToPage(v);
};

/* ---------------- zoom ---------------- */
function updateZoomLabel(){ el('zoom-label').textContent = Math.round((project.zoom||1)*100)+'%'; }
el('zoom-in').onclick=async ()=>{
  project.zoom = Math.min(3, +( (project.zoom||1)+0.2 ).toFixed(2));
  updateZoomLabel(); saveProject();
  if(pdfDoc) await renderPage(currentPage);
};
el('zoom-out').onclick=async ()=>{
  project.zoom = Math.max(0.5, +( (project.zoom||1)-0.2 ).toFixed(2));
  updateZoomLabel(); saveProject();
  if(pdfDoc) await renderPage(currentPage);
};

/* ---------------- highlight visibility toggle ---------------- */
function updateHighlightToggleBtn(){
  const btn = el('highlight-toggle');
  btn.textContent = project.highlightVisible ? '하이라이트 켬' : '하이라이트 끔';
  btn.classList.toggle('active', project.highlightVisible);
}
el('highlight-toggle').onclick=()=>{
  project.highlightVisible = !project.highlightVisible;
  updateHighlightToggleBtn(); renderHighlight(); saveProject();
};

/* ---------------- page state ---------------- */
function getPageState(n){ if(!project.pages[n]) project.pages[n]={}; return project.pages[n]; }
el('resel-btn').onclick=()=>{ armSelecting(); };

/* ---------------- banner / toast ---------------- */
function showBanner(msg, withBtn){
  positionFixedOverlays();
  const b = el('inline-banner');
  b.innerHTML = msg + (withBtn ? '<br><button class="ghost-btn" id="banner-select-btn">영역 선택 시작</button>' : '');
  b.style.display='block';
  if(withBtn){ el('banner-select-btn').onclick = armSelecting; }
}
function hideBanner(){ el('inline-banner').style.display='none'; }
let toastTimer=null;
function showToast(msg){
  positionFixedOverlays();
  const t=el('toast'); t.textContent=msg; t.classList.add('show');
  clearTimeout(toastTimer); toastTimer=setTimeout(()=>t.classList.remove('show'), 1800);
}

/* ---------------- drag-select ---------------- */
function armSelecting(){
  cancelCalib();
  selecting=true; hideBanner();
  el('canvas-scroll').classList.add('no-scroll');
  holder.classList.add('no-scroll');
  showToast('상단에서 하단으로 영역을 드래그하세요 (도안은 고정돼요)');
}
function exitSelecting(){
  selecting=false; dragStart=null; dragCur=null; el('select-box').style.display='none';
  el('canvas-scroll').classList.remove('no-scroll');
  holder.classList.remove('no-scroll');
}

const holder = el('canvas-holder');
let longPressTimer=null, longPressFired=false, longPressStartY=null;
const LONG_PRESS_MS = 420, MOVE_CANCEL_PX = 10;

holder.addEventListener('pointerdown', (e)=>{
  if(calibActive) return;
  const rect = el('pdf-canvas').getBoundingClientRect();
  pointerDownPos = {x:e.clientX, y:e.clientY};
  if(selecting){
    dragStart = { xf:(e.clientX-rect.left)/rect.width, yf:(e.clientY-rect.top)/rect.height };
    dragCur = dragStart;
  } else {
    longPressFired = false;
    longPressStartY = e.clientY;
    clearTimeout(longPressTimer);
    longPressTimer = setTimeout(()=>{
      const st = getPageState(currentPage);
      if(!st.segments || !st.segments.length) return;
      longPressFired = true;
      const yFrac = (longPressStartY-rect.top)/rect.height;
      jumpToPosition(yFrac);
      if(navigator.vibrate){ try{ navigator.vibrate(15); }catch(err){} }
    }, LONG_PRESS_MS);
  }
});
holder.addEventListener('pointermove', (e)=>{
  if(selecting && dragStart){
    const rect = el('pdf-canvas').getBoundingClientRect();
    dragCur = { xf:(e.clientX-rect.left)/rect.width, yf:(e.clientY-rect.top)/rect.height };
    const box = el('select-box');
    const x0=Math.min(dragStart.xf,dragCur.xf), x1=Math.max(dragStart.xf,dragCur.xf);
    const y0=Math.min(dragStart.yf,dragCur.yf), y1=Math.max(dragStart.yf,dragCur.yf);
    box.style.display='block';
    box.style.left=(x0*100)+'%'; box.style.width=((x1-x0)*100)+'%';
    box.style.top=(y0*100)+'%'; box.style.height=((y1-y0)*100)+'%';
    return;
  }
  if(longPressTimer && pointerDownPos){
    const dx=e.clientX-pointerDownPos.x, dy=e.clientY-pointerDownPos.y;
    if(Math.hypot(dx,dy) > MOVE_CANCEL_PX){ clearTimeout(longPressTimer); longPressTimer=null; }
  }
});
holder.addEventListener('pointercancel', ()=>{ clearTimeout(longPressTimer); longPressTimer=null; });
holder.addEventListener('contextmenu', (e)=>e.preventDefault());
holder.addEventListener('pointerup', async (e)=>{
  clearTimeout(longPressTimer); longPressTimer=null;
  if(calibActive) return;
  if(selecting && dragStart && dragCur){
    const x0=Math.min(dragStart.xf,dragCur.xf), x1=Math.max(dragStart.xf,dragCur.xf);
    const y0=Math.min(dragStart.yf,dragCur.yf), y1=Math.max(dragStart.yf,dragCur.yf);
    exitSelecting();
    if((x1-x0)>0.03 && (y1-y0)>0.03){
      startCalibrationPreview({x0,y0,x1,y1});
    } else {
      showToast('영역이 너무 작아요. 다시 시도해주세요');
    }
    return;
  }
  if(longPressFired){ longPressFired=false; pointerDownPos=null; return; }
  if(pointerDownPos){
    const dx=e.clientX-pointerDownPos.x, dy=e.clientY-pointerDownPos.y;
    const adx=Math.abs(dx), ady=Math.abs(dy);
    if(Math.hypot(dx,dy) < 8){
      const rect = el('pdf-canvas').getBoundingClientRect();
      const yFrac = (e.clientY-rect.top)/rect.height;
      handleTap(yFrac);
    } else if(adx > 55 && adx > ady*1.5){
      const cs = el('canvas-scroll');
      const atLeftEdge = cs.scrollLeft <= 2;
      const atRightEdge = cs.scrollLeft >= (cs.scrollWidth - cs.clientWidth - 2);
      if(dx < 0 && atRightEdge && currentPage < numPages){ goToPage(currentPage+1); }
      else if(dx > 0 && atLeftEdge && currentPage > 1){ goToPage(currentPage-1); }
    }
  }
  pointerDownPos=null;
});

function getCurrentSegment(){
  const st = getPageState(currentPage);
  if(!st.segments || !st.segments.length) return null;
  const n = st.segments.length;
  const localIdx = Math.min(Math.max(project.rowCount-(st.startIndex||0),0), n-1);
  return st.segments[localIdx];
}
function handleTap(yFrac){
  if(calibActive) return;
  const st = getPageState(currentPage);
  if(!st.segments || !st.segments.length){
    showBanner('이 페이지에서 사용할 영역을 화면에서 드래그로 선택하세요.', true);
    return;
  }
  const seg = getCurrentSegment();
  const center = (seg.top+seg.bottom)/2;
  if(yFrac < center){ changeRow(-1); } else { changeRow(1); }
}
function jumpToPosition(yFrac){
  const st = getPageState(currentPage);
  if(!st.segments || !st.segments.length) return;
  let idx = st.segments.findIndex(s=> yFrac>=s.top && yFrac<=s.bottom);
  if(idx===-1){
    let best=0, bestDist=Infinity;
    st.segments.forEach((s,i)=>{ const c=(s.top+s.bottom)/2; const d=Math.abs(yFrac-c); if(d<bestDist){ bestDist=d; best=i; } });
    idx=best;
  }
  project.rowCount = Math.max(0, (st.startIndex||0) + idx);
  updateRowUI(); saveProject();
  showToast('여기로 이동');
}

/* ---------------- darkness helpers (used only as a starting guess) ---------------- */
function computeRowDarkness(ctx, rx, ry, rw, rh){
  const imgData = ctx.getImageData(rx, ry, rw, rh);
  const data = imgData.data;
  const rowDark = new Float32Array(rh);
  const strideX = Math.max(1, Math.floor(rw/500));
  for(let y=0;y<rh;y++){
    let sum=0,n=0; const rowOffset=y*rw*4;
    for(let x=0;x<rw;x+=strideX){
      const idx=rowOffset+x*4;
      const lum = 0.299*data[idx]+0.587*data[idx+1]+0.114*data[idx+2];
      sum += (255-lum); n++;
    }
    rowDark[y]= n? sum/n : 0;
  }
  return rowDark;
}
function smoothArr(arr, radius){
  const out=new Float32Array(arr.length);
  for(let i=0;i<arr.length;i++){
    let s=0,n=0;
    for(let k=-radius;k<=radius;k++){ const j=i+k; if(j>=0&&j<arr.length){ s+=arr[j]; n++; } }
    out[i]=s/n;
  }
  return out;
}
function guessCount(region){
  try{
    const canvas=el('pdf-canvas'); const ctx=canvas.getContext('2d');
    const rx=Math.floor(region.x0*canvas.width), ry=Math.floor(region.y0*canvas.height);
    const rw=Math.max(1,Math.floor((region.x1-region.x0)*canvas.width));
    const rh=Math.max(1,Math.floor((region.y1-region.y0)*canvas.height));
    const rowDark = computeRowDarkness(ctx,rx,ry,rw,rh);
    let max=0; for(const v of rowDark) if(v>max) max=v;
    if(max<15) return null;
    const sm=smoothArr(rowDark,1); const threshold=max*0.55;
    const peaks=[];
    for(let y=1;y<sm.length-1;y++){ if(sm[y]>threshold && sm[y]>=sm[y-1] && sm[y]>=sm[y+1]) peaks.push(y); }
    const merged=[]; const minGap=Math.max(4, sm.length*0.01);
    for(const p of peaks){ if(merged.length && p-merged[merged.length-1]<minGap) continue; merged.push(p); }
    if(merged.length>=2) return merged.length-1;
    const threshold2=Math.max(4,max*0.12); const maxGap=Math.max(2,Math.round(rowDark.length*0.015));
    const runs=[]; let start=-1,lastInk=-999;
    for(let y=0;y<rowDark.length;y++){
      if(rowDark[y]>threshold2){ if(start===-1) start=y; lastInk=y; }
      else if(start!==-1 && (y-lastInk)>maxGap){ runs.push([start,lastInk]); start=-1; }
    }
    if(start!==-1) runs.push([start,lastInk]);
    const minRunH=Math.max(2,rowDark.length*0.01);
    const filtered=runs.filter(r=>(r[1]-r[0])>=minRunH);
    if(filtered.length>=1) return filtered.length;
  }catch(e){}
  return null;
}

/* ---------------- calibration preview (drag region -> adjust count visually -> confirm) ---------------- */
function startCalibrationPreview(region){
  const st = getPageState(currentPage);
  calibActive = true; calibRegion = region;
  const guess = guessCount(region);
  calibCount = guess || (st.segments ? st.segments.length : 10);
  hideBanner();
  positionFixedOverlays();
  el('calib-bar').style.display='flex';
  syncSliderRange();
  renderCalibLines();
}
function syncSliderRange(){
  const slider = el('calib-slider');
  const max = Math.max(80, calibCount);
  slider.max = max;
  slider.value = calibCount;
}
function renderCalibLines(){
  const wrap = el('calib-lines');
  const region = calibRegion;
  wrap.style.left=(region.x0*100)+'%'; wrap.style.width=((region.x1-region.x0)*100)+'%';
  wrap.style.top=(region.y0*100)+'%'; wrap.style.height=((region.y1-region.y0)*100)+'%';
  wrap.style.display='block';
  wrap.innerHTML='';
  for(let i=0;i<=calibCount;i++){
    const line=document.createElement('div');
    line.className='calib-line';
    line.style.top=((i/calibCount)*100)+'%';
    wrap.appendChild(line);
  }
  el('calib-count-label').textContent = calibCount + '단';
}
function cancelCalib(){
  calibActive=false; el('calib-bar').style.display='none'; el('calib-lines').style.display='none';
}
el('calib-minus').onclick=()=>{ calibCount=Math.max(1,calibCount-1); syncSliderRange(); renderCalibLines(); };
el('calib-plus').onclick=()=>{ calibCount=calibCount+1; syncSliderRange(); renderCalibLines(); };
el('calib-slider').oninput=(e)=>{ calibCount=Math.max(1,parseInt(e.target.value,10)||1); renderCalibLines(); };
el('calib-count-label').onclick=()=>{ el('manual-total').value=calibCount; el('manual-dialog').showModal(); };
el('calib-cancel').onclick=()=>{
  cancelCalib();
  const st=getPageState(currentPage);
  if(!st.segments){ showBanner('이 페이지에서 사용할 영역을 화면에서 드래그로 선택하세요.', true); }
};
el('calib-confirm').onclick=()=>{
  const st=getPageState(currentPage);
  const region=calibRegion;
  const rowH=(region.y1-region.y0)/calibCount;
  const segments=[]; for(let i=0;i<calibCount;i++) segments.push({top:region.y0+i*rowH, bottom:region.y0+(i+1)*rowH});
  st.segments=segments; st.startIndex=project.rowCount; st.region=region;
  const doneLabel = calibCount + '단';
  cancelCalib(); hideBanner();
  renderHighlight(); saveProject();
  showToast(`${doneLabel} 설정 완료`);
};
el('manual-cancel').onclick=()=>el('manual-dialog').close();
el('manual-confirm').onclick=()=>{
  const v=Math.max(1, parseInt(el('manual-total').value,10)||10);
  calibCount=v; syncSliderRange(); renderCalibLines();
  el('manual-dialog').close();
};

/* ---------------- automatic text-line detection (real PDF text layer) ---------------- */
async function tryAutoDetectText(page, viewport){
  const st = getPageState(currentPage);
  try{
    const content = await page.getTextContent();
    const pts = content.items.filter(it=>it.str && it.str.trim().length>0).map(it=>{
      const p = viewport.convertToViewportPoint(it.transform[4], it.transform[5]);
      const fontH = Math.hypot(it.transform[2], it.transform[3]) * viewport.scale || 10;
      return { vy:p[1], fontH };
    });
    if(!pts.length){
      showBanner('이 페이지는 자동으로 인식할 텍스트가 없어요. 영역을 드래그해서 표시해보세요.', true);
      return;
    }
    pts.sort((a,b)=>a.vy-b.vy);
    const lines=[[pts[0]]];
    for(let i=1;i<pts.length;i++){
      const last = lines[lines.length-1];
      const tol = Math.max(3, last[last.length-1].fontH*0.5);
      if(pts[i].vy - last[last.length-1].vy <= tol) last.push(pts[i]);
      else lines.push([pts[i]]);
    }
    const raw = lines.map(group=>{
      const vys=group.map(p=>p.vy); const avgH=group.reduce((a,p)=>a+p.fontH,0)/group.length;
      return { top: Math.min(...vys)-avgH*0.85, bottom: Math.max(...vys)+avgH*0.25 };
    });
    for(let i=0;i<raw.length-1;i++){ const mid=(raw[i].bottom+raw[i+1].top)/2; raw[i].bottom=mid; raw[i+1].top=mid; }
    raw[0].top=0; raw[raw.length-1].bottom=viewport.height;
    st.segments = raw.map(r=>({top:r.top/viewport.height, bottom:r.bottom/viewport.height}));
    st.startIndex = project.rowCount;
    hideBanner();
    showToast(`${st.segments.length}단 인식됨 (필요하면 영역 재지정으로 수정 가능)`);
    saveProject();
  }catch(e){
    showBanner('자동 인식 중 문제가 발생했어요. 영역을 드래그해서 표시해보세요.', true);
  }
}

/* ---------------- highlight rendering ---------------- */
function renderHighlight(){
  const band = el('highlight-band');
  const seg = getCurrentSegment();
  if(!pdfDoc || !seg){ band.style.display='none'; return; }
  band.style.top=(seg.top*100)+'%'; band.style.height=((seg.bottom-seg.top)*100)+'%';
  band.style.display = project.highlightVisible ? 'block' : 'none';
}
window.addEventListener('resize', ()=>{ if(pdfDoc) renderPage(currentPage); });

/* ---------------- init ---------------- */
(async function init(){
  await loadProject();
  updateRowUI(); renderPatternCounters();
  if(project.pdfBase64){
    try{ await openPdfFromArrayBuffer(base64ToArrayBuffer(project.pdfBase64)); }
    catch(e){ console.error('저장된 PDF 복원 실패', e); }
  }
  positionFixedOverlays();
})();
</script>
</body>
</html>
