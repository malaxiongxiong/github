{%- capture meta_in -%}
{
{%- for v in product.variants -%}
  "{{ v.id }}": {{ v.metafields.custom.specs_in | metafield_tag | json }}{% unless forloop.last %},{% endunless %}
{%- endfor -%}
}
{%- endcapture -%}

{%- capture meta_cm -%}
{
{%- for v in product.variants -%}
  "{{ v.id }}": {{ v.metafields.custom.specs_cm | metafield_tag | json }}{% unless forloop.last %},{% endunless %}
{%- endfor -%}
}
{%- endcapture -%}

<div class="mz-specs" data-default-unit="cm">
  <div class="mz-specs__head">
    <div class="mz-specs__left">
      <div class="mz-specs__title">Specifications</div>

      <div class="mz-specs__toggle" role="tablist" aria-label="Unit switch">
        <button type="button" class="mz-specs__btn" data-unit="in" aria-pressed="false">in</button>
        <button type="button" class="mz-specs__btn is-active" data-unit="cm" aria-pressed="true">cm</button>
      </div>
    </div>
  </div>

  <div id="mz-specs-box" class="mz-specs__body" data-unit="cm"></div>
</div>

<style>
  .mz-specs { width: 100%; }

  /* 头部：不再 space-between */
  .mz-specs__head{
    display:flex;
    align-items:center;
    justify-content:flex-start;
    margin: 0 0 12px 0;
  }

  /* 标题 + 按钮贴一起 */
  .mz-specs__left{
    display:flex;
    align-items:center;
    gap:14px;              /* 👈 标题和按钮的距离（你想更近就改小，比如 10px） */
  }

  .mz-specs__title{
    font-weight:600;
    line-height:1.2;
  }

  .mz-specs__toggle{
    display:flex;
    gap:10px;
  }

  .mz-specs__btn{
    appearance:none;
    border:1px solid rgba(0,0,0,.18);
    background:#fff;
    padding:8px 14px;
    border-radius:10px;
    cursor:pointer;
    font: inherit;
    line-height:1;
  }
  .mz-specs__btn.is-active{
    background:#111;
    color:#fff;
    border-color:#111;
  }

  .mz-specs__body p{ margin: 0 0 6px 0; }
  .mz-specs__body p:last-child{ margin-bottom:0; }
</style>

<script>
(() => {
  const specsIn = {{ meta_in }};
  const specsCm = {{ meta_cm }};

  const root = document.querySelector('.mz-specs');
  const box  = document.querySelector('#mz-specs-box');
  if (!root || !box) return;

  const buttons = Array.from(root.querySelectorAll('.mz-specs__btn'));
  const defaultUnit = root.getAttribute('data-default-unit') || 'cm';

  function setActiveUnit(unit){
    box.setAttribute('data-unit', unit);
    buttons.forEach(btn => {
      const isActive = btn.getAttribute('data-unit') === unit;
      btn.classList.toggle('is-active', isActive);
      btn.setAttribute('aria-pressed', isActive ? 'true' : 'false');
    });
  }

  function getVariantId() {
    const urlVid = new URLSearchParams(window.location.search).get('variant');
    if (urlVid) return urlVid;

    const el = document.querySelector('[data-current-variant-id],[data-selected-variant-id],[data-variant-id]');
    const dataVid = el?.getAttribute('data-current-variant-id')
      || el?.getAttribute('data-selected-variant-id')
      || el?.getAttribute('data-variant-id');
    if (dataVid) return dataVid;

    const idInput = document.querySelector('form[action*="/cart/add"] [name="id"]');
    if (idInput?.value) return idInput.value;

    return null;
  }

  function render(forceUnit) {
    const vid = getVariantId();
    if (!vid) return false;

    const unit = forceUnit || box.getAttribute('data-unit') || defaultUnit;
    const html = unit === 'in' ? (specsIn[vid] || '') : (specsCm[vid] || '');
    box.innerHTML = html || '';
    return true;
  }

  function bootRender() {
    setActiveUnit(defaultUnit);

    let tries = 0;
    const timer = setInterval(() => {
      tries += 1;
      const ok = render(defaultUnit);
      if (ok || tries >= 40) clearInterval(timer);
    }, 100);
  }

  buttons.forEach(btn => {
    btn.addEventListener('click', () => {
      const unit = btn.getAttribute('data-unit');
      setActiveUnit(unit);
      render(unit);
    });
  });

  const rerenderSoon = () => setTimeout(() => render(), 0);

  document.addEventListener('change', rerenderSoon);
  document.addEventListener('click', rerenderSoon);
  document.addEventListener('variant:change', rerenderSoon);
  document.addEventListener('product:variant-change', rerenderSoon);
  document.addEventListener('shopify:section:load', rerenderSoon);

  const idInput = document.querySelector('form[action*="/cart/add"] [name="id"]');
  if (idInput) {
    new MutationObserver(rerenderSoon).observe(idInput, { attributes: true, attributeFilter: ['value'] });
    idInput.addEventListener('change', rerenderSoon);
  }

  const _ps = history.pushState;
  history.pushState = function() { _ps.apply(this, arguments); rerenderSoon(); };
  window.addEventListener('popstate', rerenderSoon);

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', bootRender);
  } else {
    bootRender();
  }
})();
</script>
