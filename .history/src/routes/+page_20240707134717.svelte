<script lang="ts">
    import { onMount, afterUpdate } from "svelte";
    import { createChart } from 'lightweight-charts';
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

      } catch (error) {
        console.error(error);
      }
    });
    let graficar:number|undefined=undefined;
    let selected_coins:{id:string,name:string,symbol:string}[] = []
   
    let selection:any;
    function add_to_selected_coins(e:Event){
      e.preventDefault();
      const selectedCoin = coins.find((coin: any) => coin.id === selection);
      if (selectedCoin) {
        selected_coins = [...selected_coins, selectedCoin];
      } else {
        console.warn("Coin not found in the coins array:", selection);
      }
    }
    function grafica_moneda(moneda:number){
      graficar=moneda;
    }

    

    afterUpdate(async ()=>{
      if(graficar!=undefined && document.getElementById('container')!=null){
        const chart = createChart(document.getElementById('container') as HTMLElement)
        const candles = chart.addCandlestickSeries()
        console.log(selected_coins[graficar as number])
        const selectedCoin = coins.find((coin: any) => coin.id === selected_coins[graficar as number]);
        const data = await fetch("https://private-coingecko-api.vercel.app/API", {
          method: "POST",
          headers: {
            'x-api-key': "YOUR_VALID_API_KEY",
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            func: "get_coin",
            params: [selectedCoin.id],
          }),
        });
        console.log(await data.json())
      }
    });

    $: selected_coins
    $:  graficar
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
<form on:submit={add_to_selected_coins} class="m-4 p-4 bg-gray-200 rounded flex justify-start space-x-5 items-center" >
<h2 class="font-bold">Crea tu portafolio Maximo 5 monedas</h2>
<input bind:value={selection} type="text" class="rounded border border-black bg-white" list="Selected_coin">
<datalist id="Selected_coin" >
  {#each coins as coin }
    <option value={coin.id}>{coin.name}</option>
  {/each}
</datalist>
<button class="p-2 bg-blue-500 text-white rounded" type="submit">Agregar moneda</button>
</form>
{/if}

<div class="m-4 p-4 bg-gray-200 rounded">
  <h2 class="w-full text-center text-2xl font-bold m-2">Tu portafolio</h2>
  {#if selected_coins.length==0}
    <p class="w-full text-center">Aun no has seleccionado ninguna moneda para crear tu portafolio</p>
  {:else if graficar==undefined}
    {#each selected_coins as coin,i}
      <div class="bg-white rounded w-full flex justify-between items-center p-4 font-bold text-lg">
        <p>Id:{coin.id}</p>
        <p>Name:{coin.name}</p>
        <p>Symbol:{coin.symbol}</p>
        <button on:click={()=>grafica_moneda(i)} class="bg-blue-500 p-2 rounded text-white">Ver Grafico</button>
      </div>
    {/each}
  {:else}
      <div>
        <button on:click={()=>{graficar=undefined, console.log(graficar)}} class="bg-red-500 text-white p-2 rounded">Regresar</button>
        <div id="container"></div>
      </div>
  {/if}
</div>