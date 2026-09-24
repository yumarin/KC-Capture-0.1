# KC Capture

iPhone向け艦これ補助ツール
起動しませんでした
## 機能
- 艦隊情報取得
- 装備情報取得
- データコピー

## 導入方法
## ブックマークレットの作成方法

### iPhoneの場合

1. Safariで適当なページをブックマークに追加します。

2. ブックマーク一覧を開き、作成したブックマークの編集画面を開きます。

3. 名前を分かりやすいものに変更します。

   例：

   ```
   KC Capture
   ```

4. URL欄をすべて削除します。

5. 配布されているブックマークレットコードをURL欄に貼り付けます。
   
   　↓ブックマークレットコード
   
      

   ```
   javascript:(function () { var existingPanel = document.getElementById('kc-capture-panel'); if (window.__KC_BOOKMARKLET__ && existingPanel) { existingPanel.style.display = existingPanel.style.display === 'none' ? 'block' : 'none'; return; } window.__KC_BOOKMARKLET__ = true; var targets = [ { key: 'port', label: '艦隊(port)', path: '/kcsapi/api_port/port' }, { key: 'require_info', label: '装備等(require_info)', path: '/kcsapi/api_get_member/require_info' } ]; var capturedState = {}; var uiState = { opacity: 0.92, collapsed: false, left: null, top: null }; function findTarget(url) { if (!url) return null; for (var i = 0; i < targets.length; i++) { if (url.indexOf(targets[i].path) !== -1) return targets[i]; } return null; } function handlePayload(url, rawText) { var target = findTarget(url); if (!target) return; capturedState[target.key] = { rawText: rawText, capturedAt: new Date() }; render(); } var originalFetch = window.fetch; window.fetch = function () { var args = arguments; return originalFetch.apply(this, args).then(function (res) { var url = typeof args[0] === 'string' ? args[0] : (args[0] && args[0].url); if (res.ok && findTarget(url)) { res.clone().text().then(function (t) { handlePayload(url, t); }); } return res; }); }; var originalOpen = XMLHttpRequest.prototype.open; var originalSend = XMLHttpRequest.prototype.send; XMLHttpRequest.prototype.open = function (m, u) { this._kcUrl = u; return originalOpen.apply(this, arguments); }; XMLHttpRequest.prototype.send = function () { var xhr = this; xhr.addEventListener('load', function () { if (!findTarget(xhr._kcUrl)) return; if (xhr.status < 200 || xhr.status >= 300) return; var text; if (xhr.responseType === '' || xhr.responseType === 'text') { text = xhr.responseText; } else if (xhr.responseType === 'json') { text = JSON.stringify(xhr.response); } else { return; } handlePayload(xhr._kcUrl, text); }, { once: true }); return originalSend.apply(this, arguments); }; function copyToClipboard(text, onDone) { if (navigator.clipboard && navigator.clipboard.writeText) { navigator.clipboard.writeText(text).then(function () { onDone(true); }).catch(function () { fallbackCopy(text, onDone); }); } else { fallbackCopy(text, onDone); } } function fallbackCopy(text, onDone) { var ta = document.createElement('textarea'); ta.value = text; ta.style.position = 'fixed'; ta.style.opacity = '0'; document.body.appendChild(ta); ta.focus(); ta.select(); var ok = false; try { ok = document.execCommand('copy'); } catch (e) { ok = false; } document.body.removeChild(ta); onDone(ok); } var panel, header, toggleBtn, bodyWrap, bodyEl; function updatePanelBg() { if (!panel) return; panel.style.background = 'rgba(20,20,20,' + uiState.opacity + ')'; } function clampToViewport() { if (!panel) return; var maxLeft = Math.max(0, window.innerWidth - panel.offsetWidth); var maxTop = Math.max(0, window.innerHeight - panel.offsetHeight); uiState.left = Math.min(Math.max(0, uiState.left), maxLeft); uiState.top = Math.min(Math.max(0, uiState.top), maxTop); panel.style.left = uiState.left + 'px'; panel.style.top = uiState.top + 'px'; } function ensurePanel() { if (panel) return; panel = document.createElement('div'); panel.id = 'kc-capture-panel'; panel.style.cssText = [ 'position:fixed', 'width:250px', 'color:#fff', 'font-size:12px', 'font-family:sans-serif', 'z-index:2147483647', 'border-radius:8px', 'box-shadow:0 2px 8px rgba(0,0,0,0.4)', 'overflow:hidden', 'user-select:none' ].join(';'); updatePanelBg(); uiState.left = window.innerWidth - 250 - 10; uiState.top = 10; panel.style.left = uiState.left + 'px'; panel.style.top = uiState.top + 'px'; header = document.createElement('div'); header.style.cssText = [ 'padding:8px 10px', 'background:rgba(255,255,255,0.12)', 'font-weight:bold', 'cursor:move', 'display:flex', 'align-items:center', 'justify-content:space-between', 'touch-action:none' ].join(';'); var titleEl = document.createElement('span'); titleEl.textContent = 'KC Capture'; toggleBtn = document.createElement('span'); toggleBtn.textContent = '▼'; toggleBtn.style.cssText = 'cursor:pointer;padding:0 4px;'; header.appendChild(titleEl); header.appendChild(toggleBtn); bodyWrap = document.createElement('div'); bodyWrap.style.cssText = 'padding:8px 10px;'; var opacityRow = document.createElement('div'); opacityRow.style.cssText = 'display:flex;align-items:center;gap:6px;margin-bottom:8px;'; var opLabel = document.createElement('span'); opLabel.textContent = '透明度'; opLabel.style.cssText = 'font-size:10px;opacity:0.7;white-space:nowrap;'; var opSlider = document.createElement('input'); opSlider.type = 'range'; opSlider.min = '20'; opSlider.max = '100'; opSlider.value = String(Math.round(uiState.opacity * 100)); opSlider.style.cssText = 'flex:1;'; opSlider.addEventListener('input', function () { uiState.opacity = parseInt(opSlider.value, 10) / 100; updatePanelBg(); }); opacityRow.appendChild(opLabel); opacityRow.appendChild(opSlider); bodyEl = document.createElement('div'); bodyWrap.appendChild(opacityRow); bodyWrap.appendChild(bodyEl); panel.appendChild(header); panel.appendChild(bodyWrap); document.body.appendChild(panel); var drag = null; header.addEventListener('pointerdown', function (e) { if (e.target === toggleBtn) return; drag = { startX: e.clientX, startY: e.clientY, startLeft: uiState.left, startTop: uiState.top, moved: false }; header.setPointerCapture && header.setPointerCapture(e.pointerId); e.preventDefault(); }); header.addEventListener('pointermove', function (e) { if (!drag) return; var dx = e.clientX - drag.startX, dy = e.clientY - drag.startY; if (Math.abs(dx) > 3 || Math.abs(dy) > 3) drag.moved = true; uiState.left = drag.startLeft + dx; uiState.top = drag.startTop + dy; clampToViewport(); }); function endDrag() { drag = null; } header.addEventListener('pointerup', endDrag); header.addEventListener('pointercancel', endDrag); toggleBtn.addEventListener('click', function () { if (drag && drag.moved) return; uiState.collapsed = !uiState.collapsed; bodyWrap.style.display = uiState.collapsed ? 'none' : 'block'; toggleBtn.textContent = uiState.collapsed ? '▶' : '▼'; }); window.addEventListener('resize', clampToViewport); } function render() { ensurePanel(); bodyEl.innerHTML = ''; targets.forEach(function (target) { var row = document.createElement('div'); row.style.cssText = 'margin-bottom:8px;padding-bottom:8px;border-bottom:1px solid rgba(255,255,255,0.15);'; var entry = capturedState[target.key]; var title = document.createElement('div'); title.textContent = target.label; title.style.cssText = 'margin-bottom:4px;'; row.appendChild(title); var status = document.createElement('div'); status.style.cssText = 'font-size:10px;opacity:0.7;margin-bottom:4px;'; status.textContent = entry ? '取得済み ' + entry.capturedAt.toLocaleTimeString() : '未取得'; row.appendChild(status); var btn = document.createElement('button'); btn.textContent = entry ? 'コピー' : '待機中'; btn.disabled = !entry; btn.style.cssText = [ 'width:100%', 'padding:6px', 'border:none', 'border-radius:4px', 'background:' + (entry ? '#3399ff' : '#555'), 'color:#fff', 'font-size:12px' ].join(';'); btn.addEventListener('click', function () { if (!entry) return; var original = btn.textContent; copyToClipboard(entry.rawText, function (ok) { btn.textContent = ok ? 'コピーしました' : '失敗しました'; setTimeout(function () { btn.textContent = original; }, 1200); }); }); row.appendChild(btn); bodyEl.appendChild(row); }); } render(); console.log('[KC Bookmarklet] injected'); })();

   ```


6.保存します。

7. 艦これをブラウザで開いた状態で、作成したブックマークレットを実行します。

正常に動作すると、画面上にKC Captureのパネルが表示されます。

---

### 注意事項

* 本ツールはBeta版です。
* ブラウザや艦これ側の仕様変更により動作しなくなる可能性があります。
* 実行する際は、必ず艦これを開いた状態で使用してください。
