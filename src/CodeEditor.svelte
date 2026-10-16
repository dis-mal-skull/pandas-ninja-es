<script lang="ts">
  import { AceEditor } from "svelte-ace";
  import "brace/mode/python";
  import "brace/theme/cobalt";
  import "brace/ext/language_tools";
  import ToolBar from "./ToolBar.svelte";
  import { createEventDispatcher } from "svelte";

  import { shortcut } from "./shortcut.js";

  const dispatch = createEventDispatcher();

  export let code = "";
  export let hint = "";

  let showHint = false;

  function emit_run(event) {
    dispatch("run", {
      code: code,
    });
  }
</script>

<div class="flex flex-col h-full">
  <ToolBar>
    <div class="flex justify-between items-center h-full p-2 gap-2">
      <h2 class="flex text-lg font-semibold">Tu código</h2>

      <div class="flex items-center gap-2">
        {#if hint}
          <button
            class="btn btn-ghost btn-sm"
            title="Mostrar una ayuda para este reto"
            on:click={() => (showHint = !showHint)}
          >
            💡 {showHint ? "Ocultar pista" : "Pista"}
          </button>
        {/if}

        <div class="hidden md:flex gap-1 items-center opacity-70 text-xs">
          <kbd class="kbd kbd-xs">Ctrl</kbd> + <kbd class="kbd kbd-xs">Enter</kbd>
        </div>

        <button
          use:shortcut={{ control: true, code: "Enter" }}
          on:click={emit_run}
          class="btn btn-primary btn-sm"
          >▶ Probar</button
        >
      </div>
    </div>
  </ToolBar>

  {#if showHint && hint}
    <div class="m-2 shrink-0 alert alert-info">
      <span class="text-base">💡 {hint}</span>
    </div>
  {/if}

  <div class="flex-1 min-h-0">
    <AceEditor
      width="100%"
      height="100%"
      options={{
        showPrintMargin: false,
        showLineNumbers: true,
        showGutter: false,
        enableBasicAutocompletion: true,
        highlightActiveLine: false,
        wrap: 200,
        fontSize: 20,
      }}
      lang="python"
      theme="cobalt"
      bind:value={code}
    />
  </div>
</div>

<style>
  :global(.ace_content, .ace_scroller, .ace-gutter-layer, .ace_gutter, ace-cobalt) {
    background-color: hsl(var(--b1));
  }
</style>
