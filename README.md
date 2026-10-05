(function() {
  const DELAY = 30;

  function getTargetInput() {
    return document.querySelector('input[type="text"], input:not([type]), textarea');
  }

  function simulateKey(el, char) {
    const nativeInputSetter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
    el.focus();
    ['keydown','keypress'].forEach(type => {
      el.dispatchEvent(new KeyboardEvent(type, { key: char, char, keyCode: char.charCodeAt(0), bubbles: true, cancelable: true }));
    });
    const cur = el.value;
    nativeInputSetter.call(el, cur + char);
    el.dispatchEvent(new Event('input', { bubbles: true }));
    el.dispatchEvent(new Event('change', { bubbles: true }));
    el.dispatchEvent(new KeyboardEvent('keyup', { key: char, char, keyCode: char.charCodeAt(0), bubbles: true }));
  }

  function getWordFromPage() {
    const selectors = ['.word','#word','.current-word','[class*="word"]','.trainer__word','.lesson-word','span.target'];
    for (const sel of selectors) {
      const el = document.querySelector(sel);
      if (el && el.textContent.trim()) return el.textContent.trim().split(' ')[0];
    }
    const spans = [...document.querySelectorAll('span, div, p')];
    for (const s of spans) {
      const txt = s.textContent.trim();
      if (txt && /^[a-z]+$/.test(txt) && txt.length > 3 && txt.length < 25) return txt;
    }
    return null;
  }

  let stopFlag = false;

  async function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

  async function typeWord(el, word) {
    for (const ch of word) {
      if (stopFlag) return;
      simulateKey(el, ch);
      await sleep(DELAY);
    }
  }

  async function autoRun() {
    stopFlag = false;
    console.log('▶️ Запущен! Для остановки: stopFlag = true');
    let prevWord = '';
    while (!stopFlag) {
      const word = getWordFromPage();
      if (!word || word === prevWord) { await sleep(100); continue; }
      prevWord = word;
      const input = getTargetInput();
      if (!input) { await sleep(200); continue; }
      console.log('✍️', word);
      input.value = '';
      input.dispatchEvent(new Event('input', { bubbles: true }));
      await typeWord(input, word);
      await sleep(50);
      input.dispatchEvent(new KeyboardEvent('keydown', { key: ' ', keyCode: 32, bubbles: true }));
      input.value += ' ';
      input.dispatchEvent(new Event('input', { bubbles: true }));
      await sleep(200);
    }
  }

  autoRun();
})();
