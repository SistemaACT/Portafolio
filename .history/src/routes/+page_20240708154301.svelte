<script lang="ts">
    import { onMount, afterUpdate } from "svelte";
    import { createChart } from 'lightweight-charts';
    let coins:any;
    let coin_ohlc_data:{[key:number]:[][]}={};
    let coin_data:any =[]
    let open_data:boolean[] = []
    let purchase_date:string[]=[];
    let purchase_price:number[]=[];
    let purchase_ammount:number[]=[];
    let portafolio:{
      id:string,
      name:string,
      precio_compra:number,
      precio_actual:number,
      cantidad_de_compra:number
      fecha_de_compra:Date
    }[] = []
    let costo_de_adquisicion=0;
    let profit_loss=0;
    let holdings=0;
    let roi=0;
    let now:Date = new Date();

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
    let counter:number=0;
    function add_to_selected_coins(e:Event){
      e.preventDefault();
      const selectedCoin = coins.find((coin: any) => coin.id === selection);
      if (selectedCoin) {
        selected_coins = [...selected_coins, selectedCoin];
      } else {
        console.warn("Coin not found in the coins array:", selection);
      }

      get_coin_ohlc(counter)
      get_coin_data(counter)
      purchase_ammount.push(0)
      purchase_date.push("")
      counter++
    }

    async function get_coin_data(graficar:number){
      const selectedCoin = coins.find((coin: any) => coin.id === selected_coins[graficar as number].id);
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
      coin_data.push(await data.json())
      open_data.push(false)
    }
    
    function grafica_moneda(moneda:number){
      graficar=moneda;
    }

    async function get_coin_ohlc(graficar:number):Promise<any>{
      const selectedCoin = coins.find((coin: any) => coin.id === selected_coins[graficar as number].id);
      let info;
      if(!Object.keys(coin_ohlc_data).includes(String(graficar))){
        const data = await fetch("https://private-coingecko-api.vercel.app/API", {
            method: "POST",
            headers: {
              'x-api-key': "YOUR_VALID_API_KEY",
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              func: "get_btc_ohlc",
              params: [selectedCoin.id],
            }),
          });
           info = await data.json()
          coin_ohlc_data[graficar as number]=info
      }else{
          info = coin_ohlc_data[graficar as number]
      }
        //console.log(info)
        return info
    }

    afterUpdate(async ()=>{
      if(graficar!=undefined && document.getElementById('container')!=null){
        const chart = createChart(document.getElementById('container') as HTMLElement)
        const candles = chart.addCandlestickSeries()
        
        let info = await get_coin_ohlc(graficar)
        let clean_data:any[]=[]
        info.forEach((day:any)=>{
          clean_data.push({time:new Date(day[0]).toISOString().split('T')[0],open:day[1],high:day[2],low:day[3],close:day[4]})
        })
        const sortedData = clean_data.sort((a, b) => a.time - b.time);
          // Now use the sorted data
        candles.setData(sortedData);
        chart.timeScale().fitContent()
      }
    });


    const average = (array:number[]) => array.reduce((a, b) => a + b) / array.length;
    function SampleCovariance(array1:number[], array2:number[]) {
    if (array1.length !== array2.length) {
        throw new Error("Arrays must have the same length.");
    }

    const n = array1.length;
    const mean1 = array1.reduce((acc, value) => acc + value, 0) / n;
    const mean2 = array2.reduce((acc, value) => acc + value, 0) / n;

    let covarianceSum = 0;
    for (let i = 0; i < n; i++) {
        const diff1 = array1[i] - mean1;
        const diff2 = array2[i] - mean2;
        covarianceSum += diff1 * diff2;
    }

    return covarianceSum / (n - 1);
    }


    let Retornos_diarios: any[][] = [];
    let Retornos_promedio: any[][]=[];
    let varCovar: number[][]=[];

    $: selected_coins
    $: graficar
    $: {
      Retornos_diarios=[]
      Retornos_promedio=[]
      varCovar=[]
      Object.values(coin_ohlc_data).forEach((table: any[][], i: number) => {
        let coinReturns: number[] = []; // Initialize an empty array for each coin
        let averageReturns:number[]=[];
        table.forEach((row, j: number) => {
          if (j !== 0) {
            const value = row[4] / coin_ohlc_data[i][j - 1][4]! - 1;
            coinReturns.push(value); // Push daily return value to the coin-specific array
            if(j!==1){
              averageReturns.push(average(coinReturns))
            }

          }
        });
        Retornos_promedio.push(averageReturns)
        Retornos_diarios.push(coinReturns); // Push the coin-specific array to the main array
      });

      //Aqui creamos la matriz de varianza covarianza
      
      for(let i=0;i<Retornos_diarios.length;i++){
        let row:number[] = []
        for(let j=0;j<Retornos_diarios.length;j++){
          row.push(SampleCovariance(Retornos_diarios[i],Retornos_diarios[j])*365)
        }
        varCovar.push(row)
      }

      console.log(varCovar)

}

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
    <p class="text-justify p-2">
        Esta pagina no persiste informacion, por lo cual si en algun punto del proceso agrega informacion y desea borrarla tendra que reiniciar la pagina.
        Esto ocacionara que toda la infomacion sea borrada y tendra que volver a empezar el programa. A continuacion dejo las instrucciones de uso:
    </p>
    <p class="p-2">Paso 1: Agregar las monedas que quiera a el portafolio, puede seleccionar las que quiera mientras no exceda 10 peticiones por minuto (Es decir si espera puede agregar aun mas)</p>
    <p class="p-2">Paso 2: Ver el grafico de la moneda que va a comprar</p>
    <p class="p-2">Paso 3: Regrese y presione ver datos, aqui notara un formulario solicitando la fecha de compra y cantidad comprada</p>
    <p class="p-2">Paso 4: Seleccione la fecha donde desea realizar la compra asegurese que los calculos inferiores sean numeros puesto que se puede elegir fechas invalidas. (Tiene hasta 1 año de datos diarios) </p>
    <p class="p-2">Paso 5: Agrega tu compra al portafolio, cierra la informacion presionando ver datos y observa los resultados</p>
</div>

{#if coins}
<form on:submit={add_to_selected_coins} class="m-4 p-4 bg-gray-200 rounded flex justify-start space-x-5 items-center" >
<h2 class="font-bold">Crea tu portafolio</h2>
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
  <h2 class="w-full text-center text-2xl font-bold m-2">Arma Tu portafolio</h2>
  {#if selected_coins.length==0}
    <p class="w-full text-center">Aun no has seleccionado ninguna moneda para crear tu portafolio</p>
  {:else if graficar==undefined}
    {#each selected_coins as coin,i}
    <div class="bg-white rounded w-full">
      <div class="bg-white rounded w-full flex justify-between items-center p-4 my-2 font-bold text-lg">
        <p>Id:{coin.id}</p>
        <p>Name:{coin.name}</p>
        <p>Symbol:{coin.symbol}</p>
        <button on:click={()=>grafica_moneda(i)} class="bg-blue-500 p-2 rounded text-white">Ver Grafico</button>
        <button on:click={()=>open_data[i]=!open_data[i]} class="bg-green-500 p-2 rounded text-white">Ver Datos</button>
      </div>
      {#if open_data[i]}
        <div class="flex p-4">
          <img src={coin_data[i].image.small} alt="img" />
            <p class="p-2">{coin_data[i].description.en.slice(0,500)+"..."}</p>
        </div>
        <div>
          <div>
            <p class="p-4">Precio Actual: {coin_data[i].market_data.current_price.usd}</p>

          </div>
          <div class="p-4">
            <div class="flex flex-col">
              <label for="Cantidad">Cantidad a comprar</label>
              <input bind:value={purchase_ammount[i]} class="bg-white border border-black rounded" placeholder="Cantidad a comprar" type="number" id="Cantidad"/>
            </div>
            <div class="flex flex-col">
              <label for="Compra">Fecha de comprar</label>
              <input  on:change={async (event)=>{
                if (event.target instanceof HTMLInputElement) {
                  purchase_date[i]= new Date(event.target.value).toISOString().split('T')[0]
                }
                let selected_row;
                let table = await get_coin_ohlc(i)
                selected_row = table.find((row)=>new Date(row[0]).toISOString().split("T")[0]===purchase_date[i])

                purchase_price[i]=selected_row[4]
   
              
              }} class="bg-white border border-black rounded" placeholder="Fecha de comprar" type="date" id="Compra"/>
            </div>
            <div class=" flex justify-between p-4">
              <div>
                <p>Precio de Compra</p>
                <p>${purchase_price[i]}</p>
              </div>
              <div>
                <p>Monto de compra</p>
                <p>${purchase_price[i]*purchase_ammount[i]}</p>
              </div>
              <button class="bg-green-500 text-white p-2 rounded"
                on:click={()=>{
                  costo_de_adquisicion+=purchase_price[i]*purchase_ammount[i]
                  holdings+=coin_data[i].market_data.current_price.usd*purchase_ammount[i]
                  profit_loss+=coin_data[i].market_data.current_price.usd*purchase_ammount[i]-purchase_price[i]*purchase_ammount[i]
                  roi+=(coin_data[i].market_data.current_price.usd*purchase_ammount[i]-purchase_price[i]*purchase_ammount[i])/(purchase_price[i]*purchase_ammount[i])
                  portafolio = [...portafolio,{id:coin.id,name:coin.name,precio_compra:purchase_price[i],precio_actual:coin_data[i].market_data.current_price.usd,cantidad_de_compra:purchase_ammount[i],fecha_de_compra:new Date(purchase_date[i])}]
                }}
              >Agregar al protafolio</button>
            </div>
          </div>
        </div>
      {/if}
    </div>
    {/each}
  {:else}
      <div>
        <button on:click={()=>{graficar=undefined}} class="bg-red-500 m-2 text-white p-2 rounded">Regresar</button>
        <div id="container" class="h-64 w-full"></div>
      </div>
  {/if}
</div>



{#if portafolio.length!=0}
  <div class="bg-gray-200 m-4 rounded p-4">
    <h2 class="w-full text-center text-2xl font-bold m-2">Resultados de tu Protafolio</h2>
    <div class="bg-white rounded p-4 my-2">
      <h3>Sobre Tu Portafolio</h3>
      <p>Costo de Adquisicion: ${costo_de_adquisicion}</p>
      <p>Valor del Portafolio: ${holdings}</p>
      <p>Profit/Loss: ${profit_loss}</p>
      <p>ROI:{roi}</p>
    </div>
    <h2>Resultados Individuales</h2>
    {#each portafolio as coin}
    <div class="flex justify-evenly bg-white p-2 rounded">
      <div class="border-r border-black flex flex-col p-2">
        <p>Id: {coin.id}</p>
        <p>Name: {coin.name}</p>
        <p>Fecha de Compra: {coin.fecha_de_compra.toISOString().split("T")[0]}</p>
        <p>Cantidad Comprada: {coin.cantidad_de_compra}</p>
        <p>Precio de Compra: {coin.precio_compra}</p>
        <p>Precio Actual: {coin.precio_actual}</p>
      </div>
      <div class="flex flex-col p-2">
        <p>Inversion Total: {coin.precio_compra * coin.cantidad_de_compra}</p>
        <p>Valor actual del portafolio: {coin.precio_actual * coin.cantidad_de_compra}</p>
        <p>Profit/Loss: {coin.precio_actual * coin.cantidad_de_compra - coin.precio_compra * coin.cantidad_de_compra}</p>
        <p>ROI: {(coin.precio_actual * coin.cantidad_de_compra - coin.precio_compra * coin.cantidad_de_compra) / (coin.precio_compra * coin.cantidad_de_compra)}</p>
        <p>Dias desde compra:{now.getDate()-coin.fecha_de_compra.getDate()}</p>
        <p>Retorno Diario:{((coin.precio_actual * coin.cantidad_de_compra - coin.precio_compra * coin.cantidad_de_compra) / (coin.precio_compra * coin.cantidad_de_compra))/(now.getDate()-coin.fecha_de_compra.getDate())}%</p>
      </div>
    </div>
    
      
    {/each}
  </div>

  <div>
    <h2 class="w-full text-center text-2xl font-bold m-2">Estimaciones</h2>
    <p class="m-2">Por motivos de simpicidad aqui se realizaran los calculos considerando exactametne un año de informacion </p>
    <div class="w-full flex justify-center">
      <table class="m-2 border border-black rounded">
        <thead>
          <th class="text-center">Fecha</th>
          {#each selected_coins as coin}
            <th class="border-x border-black">
              {coin.name}
            </th>
         {/each} 
        </thead>
        <tbody>
          {#each Object.values(coin_ohlc_data)[0] as rows,i}
            <tr class="border-y border-black">
              <td>{new Date(Number(rows[0])).toISOString().split('T')[0]}</td>
              {#each Object.values(coin_ohlc_data) as data }
                <td class="border-x border-black">{data[i][4]}</td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>

    <div>
      <h2>Rendimientos</h2>
      <table>
        <thead>
         {#each selected_coins as coin}
          <th class="border-x border-black">
            {coin.name}
          </th>
         {/each} 
        </thead>
        <tbody>
          {#each Retornos_diarios[0] as _,i}
            <tr class="border-y border-black">
              {#each Retornos_diarios as _,j}
                <td class="border-x border-black">
                  {Retornos_diarios[i][j]}
                </td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>


    <div>
      <h2>Retornos Promedio</h2>
      <table>
        <thead>
         {#each selected_coins as coin}
          <th class="border-x border-black">
            {coin.name}
          </th>
         {/each} 
        </thead>
        <tbody>
          {#each Retornos_promedio[0] as _,i}
            <tr class="border-y border-black">
              {#each Retornos_promedio as _,j}
                <td class="border-x border-black">
                  {Retornos_promedio[i][j]}
                </td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>

  </div>





  {/if}

