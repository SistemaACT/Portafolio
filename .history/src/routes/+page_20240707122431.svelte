<script lang="ts">
    import { onMount } from "svelte";
    let coins:any;
    onMount(async () => {
      try {
        let res = await fetch("https://private-coingecko-api.vercel.app/API", {
          method: "POST",
          headers: {
            'x-api-key': "YOUR_VALID_API_KEY",
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            func: "get_coin_list",
            params: [],
          }),
        });
        coins = await res.json()
        console.log(coins);

      } catch (error) {
        console.error(error);
      }
    });
  </script>
  




<div class="w-full bg-blue-500 text-white p-5 text-2xl font-bold flex items-center  space-x-4 ">
    <h1>
        Proyecto Final Topicos Financieros
    </h1>
    <p class="text-lg">Autor: Jesus Gómez-Braña González</p>
    <p class="text-lg">Profesor: Dr. Roberto Hernández Martínez</p>

</div>

<div class="bg-gray-200 rounded p-4 flex flex-col items-center m-4 shadow-md">
    <h2 class="w-full bg-white rounded text-xl font-bold text-center">Acerca del modelo y del proyecto</h2>
    <p class="text-justify p-2">
        Para comenzar hay que explicar que es lo que se esta presentando. Actualmente
        esta observando una aplicacion realizada con sveltekit y tailwindcss. Este framework permite
        realizar paginas dinamicas y reactivas con mucha facilidad utilizando html.
    </p>
    <p class="text-justify p-2">
        Esto se realizo para facilitar la visualizacion de la informacion relevante al proyecto final.
        Puesto que al utilizar Google sheets se podria ver limitado por la funcionalidad que tiene este.
        De esta manera se lograra explorar mas a fondo lo realizado en clase y brindara ideas nuevas al modelo
        propuesto en clase
    </p>
    <h3 class="w-full bg-white rounded text-xl font-bold text-center">Sobre el modelo</h3>
    <p class="text-justify p-2">
        Para este modelo se esta realizando un crypto portafolio, en el cual se analizaran datos reales para
        determinar si el portafolio seria rentable al igual que medir sus riesgos. Una de la manera en la cual se 
        se realiza esto es mediante la implementacion del modelo value at risk. El cual resulta te da una idea de cuanto podría salir mal un proyecto de inversion.
    </p>
</div>

{#if coins}
<form class="w-full m-4 p-4 bg-gray-200 rounded flex justify-start space-x-5" >
<h2 class="font-bold">Crea tu portafolio Maximo 7 monedas</h2>
<input type="text" class="rounded border border-black bg-white" list="Selected_coin">
<datalist id="Selected_coin" >
  {#each coins as coin }
    <option value={coin.id}>{coin.name}</option>
  {/each}
</datalist>
<button type="submit">Agregar moneda</button>
</form>
{/if}