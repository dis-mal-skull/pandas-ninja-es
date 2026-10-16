<script lang="ts">
  export let show = false;

  function close() {
    show = false;
  }
</script>

<div class="modal modal-bottom sm:modal-middle {show ? 'modal-open' : ''}">
  <div class="modal-box max-w-2xl">
    <h2 class="text-3xl font-bold mb-1">¿Cómo se juega? 🥷</h2>
    <p class="opacity-70 mb-4">Tranquilo/a: aquí nadie se equivoca. ¡Se puede probar las veces que quieras!</p>

    <ol class="space-y-3 text-lg leading-relaxed">
      <li>
        <b>1. Mira las tablas.</b> 🐼 En <b>Tu tabla</b> ves lo que tienes en cada momento.
        En <b>Tabla objetivo</b> está lo que tienes que conseguir.
      </li>
      <li>
        <b>2. Escribe tu código</b> en el panel <b>Tu código</b>. Tu tabla se llama <code class="kbd kbd-sm">df</code>.
      </li>
      <li>
        <b>3. Pulsa ▶ Probar</b> (o <kbd class="kbd kbd-sm">Ctrl</kbd> + <kbd class="kbd kbd-sm">Enter</kbd>)
        para ver si tu tabla ya se parece a la objetivo.
      </li>
      <li>
        <b>4. ¿No sabes qué escribir?</b> 💡 Pulsa el botón <b>Pista</b> arriba del editor.
      </li>
      <li>
        <b>5. Gana estrellas ⭐.</b> Cuantas menos líneas uses, más estrellas consigues.
      </li>
    </ol>

    <div class="mt-5 rounded-box bg-base-200 p-4 text-base">
      <p class="font-semibold mb-1">Consejos</p>
      <ul class="list-disc list-inside space-y-1">
        <li>Si te sale un mensaje en rojo abajo, es normal: solo dice que algo hay que corregir.</li>
        <li>Puedes borrar y volver a escribir con <b>Limpiar</b>.</li>
        <li>Cambia de reto con los botones ◀ y ▶, o desde el menú <b>Reto</b>.</li>
      </ul>
    </div>

    <div class="modal-action">
      <button class="btn btn-primary btn-lg" on:click={close}>¡Entendido!</button>
    </div>
  </div>
</div>
