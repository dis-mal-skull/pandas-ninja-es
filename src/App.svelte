<script lang="ts">
  import Header from "./Header.svelte";
  import DataFrame from "./DataFrame.svelte";

  import CodeEditor from "./CodeEditor.svelte";
  import Grid from "./Grid.svelte";
  import Console from "./Console.svelte";
  import ChallengeList from "./ChallengeList.svelte";
  import Drawer from "./Drawer.svelte";
  import VictoryDialog from "./VictoryDialog.svelte";
  import LoadingDialog from "./LoadingDialog.svelte";
  import HelpDialog from "./HelpDialog.svelte";

  // Versión de la app inyectada por Vite (ver vite.config.ts)
  declare const __APP_VERSION__: string;
  const APP_VERSION =
    typeof __APP_VERSION__ !== "undefined" ? __APP_VERSION__ : "dev";

  /** ======================
   *      Interface
   *========================
   */

  // Challenge from from Challenge List
  interface Challenge {
    title: string;
    file: string;
    level: string;
    rating: number;
  }

  // Challenge object
  interface ChallengeDetail {
    title: string;
    author: string;
    placeholder?: string;
    hint?: string;
    input: Array<Object>;
    expected: Array<Object>;
  }

  /** ======================
   *      Variables
   *========================
   */

  // List of challenges
  let challenges: Challenge[] = [];

  // current challenge
  let challenge: Challenge | undefined = undefined;

  // current challenge details
  let challengeDetail: ChallengeDetail | undefined = undefined;

  // Index of current challenge (undefined until the list is loaded)
  let challengeIndex: number | undefined = undefined;

  // Reload the challenge detail whenever the index changes and the list is ready
  $: if (challengeIndex !== undefined && challenges.length > 0) {
    loadChallengeDetail(challengeIndex);
  }

  // Python handle
  let pyodide: any = null;

  // console output
  let consoleOutput: string = "";

  // codeEditor content
  let code: string = "";

  // Display Loading modal box
  let loading: boolean = true;

  // Loading message
  let loadingMessage: string = "Preparando todo... 🐼";

  // display Win modal box
  let won: boolean = false;

  // Win score
  let score: number = 0;

  // expression number
  let expressionCount: number;

  // Drawer pen
  let show_drawer: boolean = false;

  // ¿Está abierta la ventana de ayuda "¿Cómo se juega?"
  let helpOpen: boolean = false;

  // Computed dataframe
  let computed_input: any = [];

  // Computed dataframe as string
  let computed_input_str: any = [];

  /** ======================
   *      Functions
   *========================
   */

  // ------------------------------------------------------
  // Append text to console
  // ------------------------------------------------------
  function appendConsole(text: string): void {
    consoleOutput = consoleOutput + (consoleOutput ? "\n" : "") + text;
  }

  // ------------------------------------------------------
  // Clear console
  // ------------------------------------------------------
  function clearConsole() {
    consoleOutput = "";
  }

  // ------------------------------------------------------
  // Load all challenges from challenges.json
  // ------------------------------------------------------
  async function loadChallenges() {
    loadingMessage = "Cargando los retos";

    challenges = [];
    try {
      let response = await fetch("challenges.json", { cache: "no-store" });
      let el = await response.json();
      challenges = el["data"] ?? [];

      // get All rating
      challenges.map((x) => (x.rating = getRating(x.file)));
    } catch (error) {
      console.error("No se pudo cargar challenges.json");
    }
  }

  // ------------------------------------------------------
  // Get a Challenge from index
  // ------------------------------------------------------
  function getChallenge(index: number): Challenge | undefined {
    if (index >= 0 && index < challenges.length) {
      return challenges[index];
    }
    return undefined;
  }
  // ------------------------------------------------------
  // set a Challenge from index
  // ------------------------------------------------------
  function setChallenge(index: number): void {
    challengeIndex = index;
  }
  // ------------------------------------------------------
  // Load a Challenge detail from specific index
  // ------------------------------------------------------
  async function loadChallengeDetail(index: number) {
    challenge = getChallenge(index);
    let file = challenge?.file;

    if (!file) {
      // Todavía no hay un reto seleccionado: no hacemos nada.
      return;
    }

    try {
      let response = await fetch(`challenges/${file}`, { cache: "no-store" });

      challengeDetail = await response.json();

      code = challengeDetail.placeholder ?? "";
      runCode();
    } catch (error) {
      console.error("No se pudo cargar el reto", file);
    }
  }

  // ------------------------------------------------------
  // get Rating from local Storage
  // ------------------------------------------------------
  function getRating(file: string): number {
    let value = localStorage.getItem(file) ?? "-1";
    return parseInt(value);
  }
  // ------------------------------------------------------
  // Load Python
  // ------------------------------------------------------
  async function loadPython() {
    loading = true;

    loadingMessage = "Cargando Python y pandas... 🐼 (la primera vez tarda un poco)";

    pyodide = await loadPyodide({
      stdout: (text: string) => appendConsole(text),
      stderr: (text: string) => appendConsole(text),
    });
    // Pyodide is now ready to use...
    let python_version = pyodide.runPython(`
    import sys
    sys.version
    `);

    appendConsole(python_version);

    // Load Pandas
    loadingMessage = "Cargando pandas... 🐼";
    await pyodide.loadPackage(["pandas"]);

    // import pandas.. It takes times
    pyodide.runPython(`import pandas as pd `);
  }

  // ------------------------------------------------------
  // Init function at startup
  // ------------------------------------------------------

  async function main() {
    loading = true;
    await loadPython();
    await loadChallenges();

    challengeIndex = 0;
    loading = false;

    // La primera vez mostramos la ayuda para saber cómo se juega.
    if (!localStorage.getItem("pandas_ninja_help_seen")) {
      helpOpen = true;
      localStorage.setItem("pandas_ninja_help_seen", "1");
    }
  }

  // ------------------------------------------------------
  // run Code
  // ------------------------------------------------------
  function runCode() {
    try {
      // Move JS code to Python
      clearConsole();
      won = false;
      score = 0;

      if (!challengeDetail) {
        return;
      }

      pyodide.globals.set("raw_input", challengeDetail.input);

      // build pre-code and post-code
      const start_code = `import js\nimport ast\ndf = pd.DataFrame(raw_input.to_py())`;
      const end_code = `df_str = df.astype(str).to_dict(orient="records")\ndf = df.to_dict(orient="records")`;
      const intro_code = `line_number=len(ast.parse("""${code}""").body)`;

      // build all code
      let all_code =
        start_code + "\n" + code + "\n" + end_code + "\n" + intro_code;

      // avoid forbidden words

      if (code.includes("DataFrame(") || code.includes("Series(")) {
        appendConsole(
          "Para que sea más fácil, no hace falta crear una tabla desde cero.\nIntenta transformar la tabla que ya tienes con métodos, por ejemplo: df.head(3)"
        );
        return;
      }

      // run all_code
      pyodide.runPython(all_code);

      // get results
      let json_result = pyodide.globals.get("df");
      let json_result_str = pyodide.globals.get("df_str");
      expressionCount = pyodide.globals.get("line_number");

      if (expressionCount == 0 || expressionCount > 3) score = 0;
      else score = 4 - expressionCount;

      computed_input = json_result.toJs({ dict_converter: Object.fromEntries });
      computed_input_str = json_result_str.toJs({
        dict_converter: Object.fromEntries,
      });

      // check victories
      checkVictory();
    } catch (err) {
      let message = (err as Error).message;
      appendConsole(message);
    }
  }

  // ------------------------------------------------------
  // Next Level
  // ------------------------------------------------------

  function nextLevel() {
    if (challengeIndex !== undefined && challengeIndex < challenges.length - 1) {
      // El cambio de índice dispara la recarga del detalle (ver $: arriba)
      challengeIndex = challengeIndex + 1;
    }
  }

  // ------------------------------------------------------
  // Previous Level
  // ------------------------------------------------------

  function previousLevel() {
    if (challengeIndex !== undefined && challengeIndex > 0) {
      challengeIndex = challengeIndex - 1;
    }
  }

  // ------------------------------------------------------
  // Check victory
  // ------------------------------------------------------
  function checkVictory() {
    if (!challengeDetail) {
      return;
    }

    if (
      JSON.stringify(computed_input) == JSON.stringify(challengeDetail.expected)
    ) {
      if (challenge) {
        localStorage.setItem(challenge.file, score.toString());
        challenge.rating = score;
      }
      won = true;
    }
  }
</script>

<svelte:head>
  <script
    src="https://cdn.jsdelivr.net/pyodide/v0.21.3/full/pyodide.js"
    on:load={main}
  ></script>
</svelte:head>

<!-- Modal Dialog  -->
<LoadingDialog {loading} message={loadingMessage} />
<VictoryDialog show={won} {score} {expressionCount} on:next={nextLevel} />
<HelpDialog bind:show={helpOpen} />

<Drawer show={show_drawer}>
  <svelte:fragment slot="content">
    <Header
      on:left={previousLevel}
      on:right={nextLevel}
      on:menu={() => (show_drawer = !show_drawer)}
      on:help={() => (helpOpen = true)}
      title={challenge?.title}
      level={challenge?.level}
      rating={challenge?.rating}
      index={challengeIndex ?? 0}
    />

    <Grid>
      <CodeEditor
        slot="a"
        on:run={runCode}
        bind:code
        hint={challengeDetail?.hint ?? ""}
      />
      <Console bind:code={consoleOutput} slot="b" />
      <DataFrame title="Tu tabla" slot="c" data={computed_input_str} />
      <DataFrame
        title="Tabla objetivo"
        slot="d"
        data={challengeDetail?.expected}
      />
    </Grid>

    <p
      class="font-mono text-xs text-right bg-base-300 pt-1  border-t-2 border-base-300 px-2"
    >
      versión {APP_VERSION}
    </p>
  </svelte:fragment>

  <svelte:fragment slot="sidebar">
    <ChallengeList bind:current_index={challengeIndex} {challenges} />
  </svelte:fragment>
</Drawer>

<style>
</style>
