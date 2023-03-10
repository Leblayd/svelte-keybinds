<script context='module'>
  import { writable } from 'svelte/store'
  import { keys, layout } from './maps.js'

  type KeyBinds = { [key: string]: { fn: () => void, id: string, type?: 'icon' } }
  type Condition = (code: string, e: MouseEvent) => boolean | null

  let keyBinds: KeyBinds | null = null
  export const _keyBinds = writable<Keybinds | null>(null)
  _keyBinds.subscribe((obj) => {
    keyBinds = obj ?? {}
  })

  let condition: Condition | null = null
  export const _condition = writable<Condition>(() => true)
  _condition.subscribe((fn) => {
    if (typeof fn !== 'function') throw new Error('Condition must be a function')
    condition = fn
  })

  window.addEventListener('keydown', runBind)

  async function runBind (code: string, e: MouseEvent) {
    console.log(layout[code]);
    console.log(code);
    if (await condition(e)) keyBinds[layout[code] || code]?.fn?.(e)
  }

  export function loadWithDefaults (defaults: KeyBinds) {
    const def = toIDmap(defaults)
    const saved = JSON.parse(localStorage.getItem('thaunknown/svelte-keybinds'))
    for (const id in saved) {
      saved[id].fn = def[id]?.fn
    }
    _keyBinds.set(toKeyMap({ ...def, ...saved }))
  }

  function toIDmap (target = {}) {
    const obj = {}
    for (const code in target) {
      const bind = target[code]
      obj[bind.id] = { ...bind, code }
    }
    return obj
  }

  function toKeyMap (target = {}) {
    const obj = {}
    for (const code in target) {
      const bind = target[code]
      obj[bind.code] = { ...bind, code }
    }
    return obj
  }

  function save () {
    localStorage.setItem('thaunknown/svelte-keybinds', JSON.stringify(toIDmap(keyBinds)))
  }

  function load() {
    return JSON.parse(localStorage.getItem('thaunknown/svelte-keybinds'))
  }
</script>

<script>
  export let autosave = false

  export let clickable = false

  const kbnref = keyBinds
  let dragged = null
  function draggable (node, code) {
    let drag = false
    node.addEventListener('dragstart', ({ target }) => {
      dragged = target
      target.classList.add('transparent')
      drag = true
    })
    node.addEventListener('dragend', ({ target }) => {
      target.classList.remove('transparent')
      drag = false
    })
    node.addEventListener('dragover', (e) => {
      e.dataTransfer.dropEffect = 'move'
      e.preventDefault()
      if (!drag) e.target.classList.add('transparent')
    })
    node.addEventListener('dragleave', ({ target }) => {
      if (!drag) target.classList.remove('transparent')
    })
    node.addEventListener('drop', ({ target }) => {
      target.style.opacity = null
      const targetcode = dragged.dataset.code
      if (kbnref[code]) {
        const temp = kbnref[targetcode]
        kbnref[targetcode] = kbnref[code]
        kbnref[code] = temp
      } else {
        kbnref[code] = kbnref[targetcode]
        delete kbnref[targetcode]
      }
      _keyBinds.set(kbnref)
      if (autosave === true) {
        save()
      }
    })
  }
</script>

<div class='svelte-keybinds'>
  {#each Object.values(keys) as key}
    {@const { size, dark, name } = key}
    <div
      class:dark
      draggable={!!kbnref[name]}
      data-code={name}
      class='w-{size || 50}'
      title='{kbnref[name]}'
      {...$$restProps}
      use:draggable={name}
      on:click={(e) => clickable && runBind(name, e)}>
      <slot prop={kbnref[name]} />
    </div>
  {/each}
</div>

<style>
  .svelte-keybinds {
    flex-shrink: 0;
    user-select: none;
    display: flex;
    flex-wrap: wrap;
    width: 82em;
    background: rgba(0, 0, 0, 0.8);
    border-radius: 0.4em;
    padding: 1em;
    color: #eee;
  }

  .svelte-keybinds .dark {
    background: #191c20 !important;
  }

  .svelte-keybinds .transparent {
    opacity: 0.2 !important;
  }

  .svelte-keybinds > div {
    height: 4em;
    margin: 0.5em;
    background: #25282c;
    border-radius: 0.4em;
    cursor: pointer;
    transition-property: opacity, transform;
    transition-duration: 0.2s;
    transition-timing-function: ease;
  }
  .svelte-keybinds > div > :global(*) {
    pointer-events: none !important;
  }
  .svelte-keybinds > div:hover {
    transform: scale(0.9);
  }
  .svelte-keybinds .w-50 {
    width: 4em !important;
  }
  .svelte-keybinds .w-75 {
    width: 6.5em !important;
  }
  .svelte-keybinds .w-85 {
    width: 7.5em !important;
  }
  .svelte-keybinds .w-90 {
    width: 8em !important;
  }
  .svelte-keybinds .w-100 {
    width: 9em !important;
  }
  .svelte-keybinds .w-110 {
    width: 10em !important;
  }
  .svelte-keybinds .w-115 {
    width: 10.5em !important;
  }
  .svelte-keybinds .w-300 {
    width: 29em !important;
  }
</style>
