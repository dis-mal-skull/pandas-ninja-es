<script lang="ts">
  import { AceEditor } from "svelte-ace";
  import "brace/theme/cobalt";
  import ToolBar from "./ToolBar.svelte";

  export let code = "";
</script>

<ToolBar>
  <div class="flex items-center h-full flex-row justify-between p-2">
    <h2>Consola</h2>
    <button class="btn  btn-sm" on:click={() => (code = "")}> Limpiar </button>
  </div>
</ToolBar>

<AceEditor
  lang="text"
  width="100%"
  height="100%"
  options={{
    showPrintMargin: false,
    showLineNumbers: false,
    showGutter: false,
    readOnly: true,
    highlightActiveLine: false,
    fontSize: 20,
  }}
  theme="cobalt"
  bind:value={code}
/>
