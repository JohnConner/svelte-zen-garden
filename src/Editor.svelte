<script>
  import { 
    // onMount, 
    onDestroy } from 'svelte'
  import { userCSS, showEditor } from './store'
  // import { editor as MonacoEditor } from 'monaco-editor';
  import { pannable } from './pannable.js';
  let editor, modelChangeSub
  let x = 0
  let y = 0
  // onMount(mountEditor)
  function mountEditor() {
    setTimeout(() => {
      import("monaco-editor").then(monaco => {
        editor = monaco.editor.create(document.getElementById('monaco-container'), {
          value: $userCSS.split(`
`).join('\n'),
          language: 'css',
          roundedSelection: false,
          scrollBeyondLastLine: false,
          readOnly: false,
          tabSize: 2,
          theme: "vs-dark",
          minimap: {
            enabled: false
            // renderCharacters: false
          }
        });
        modelChangeSub = editor.getModel().onDidChangeContent(v => {
          userCSS.set(editor.getModel().getValue())
        })
      })
    }, 200)
  }
  userCSS.subscribe(v => {
    if (!editor || !editor.getModel()) return
    const curVal = editor.getModel().getValue()
    if (curVal !== v) {
      editor.setValue(v)
    }
  })
  onDestroy(() => {
    if (editor) {
      editor.dispose();
      const model = editor.getModel();
      if (model) model.dispose();
    }
    if (modelChangeSub) modelChangeSub.dispose()
  })



  function handlePanMove(event) {
    x = x + event.detail.dx
    y = y + event.detail.dy
  }
  function toggleEditor() {
    $showEditor = !$showEditor
    if ($showEditor) mountEditor()
  }
</script>

<style>
  #editorWithButton {
    height: 40vh;
    width: 40vw;
    min-width: 600px;
    position: fixed;
    z-index: 999;
    bottom: 0;
    left: 0;
  }
  #monaco-container {
    height: 100%;
    width: 100%;
  }
  .handButton {
    position: fixed;
    width: 60px;
    height: 60px;
    z-index: 9999;
    top: -30px;
    right: -30px;
    border-radius: 50px;
    text-align: center;
    box-shadow: 2px 2px 3px #999;
    font-size: 3rem;
  }
  .open {
    background-color: #0C9;
  }
  .close {
    background-color: #F55;
  }
</style>
<div use:pannable
  on:panmove={handlePanMove}
  style="transform:
  translate({x}px,{y}px)"
  id="editorWithButton"
>
  {#if $showEditor}
  <button class="handButton close" on:click={toggleEditor}>
    ✖️    
  </button>
  {:else}
  <button class="handButton open" on:click={toggleEditor}>
    ✍️
  </button>
  {/if}
  {#if $showEditor}
  <div id="monaco-container"></div>
  {/if}
</div>
