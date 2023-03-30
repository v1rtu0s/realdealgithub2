<script>
  import crypto from "crypto";
  import Gear from "./dia_green.svg?inline";
  import { quintOut } from "svelte/easing";
  import { fade, draw, fly } from "svelte/transition";

  var ServerSeed = "123";
  var ClientSeed = "123";
  var nonce = 1;
  var cursor = 0;
  var count = 8;
  var activeTab = "dice";
  var isFieldsFilled = false;
  const GEMS = [green, purple, yellow, red, cyan, orange, blue];
  const green = "#1ce61d";
  const purple = "#864bfa";
  const yellow = "#fde22e";
  const red = "#ff1c44";
  const cyan = "#6bf9ff";
  const orange = "#ff91d2";
  const blue = "#1e6eef";
  var endresultgems = [];
  var gem1color = "#ffffff";
  var gem2color = "#ffffff";
  var gem3color = "#ffffff";
  var gem4color = "#ffffff";
  var gem5color = "#ffffff";

  var BetObject = {
    ServerSeed: ServerSeed,
    ClientSeed: ClientSeed,
    nonce: nonce,
    cursor: cursor,
    count: count
  };

  var verify = generateFloats(BetObject);
  var resultat = game_evaluation();

  function checkFields() {
    if (ClientSeed && ServerSeed && nonce) {
      isFieldsFilled = true;
      console.log(isFieldsFilled);
      return isFieldsFilled;
    } else {
      isFieldsFilled = false;
      console.log(isFieldsFilled);
      return isFieldsFilled;
    }
  }

  function game_evaluation_handler() {
    checkFields();
    game_evaluation();
    console.log("handler working");
  }
  $: resultat = game_evaluation(BetObject);

  $: console.log(resultat);
  $: Clientseed = checkFields();
  $: verify = generateFloats(BetObject);
  $: console.log(BetObject);
  $: BetObject.ServerSeed = ServerSeed;
  $: BetObject.ClientSeed = ClientSeed;
  $: BetObject.nonce = nonce;
  $: console.log(activeTab);
  $: gem1color;
  gem2color;
  gem3color;
  gem4color;
  gem5color;

  function generateBytes({ ServerSeed, ClientSeed, nonce, cursor, count }) {
    // Setup cursor variables
    let currentRound = Math.floor(cursor / 32);
    let currentRoundCursor = cursor;
    currentRoundCursor -= currentRound * 32;
    // Generate bytes until count is fulfilled
    const bytes = [];
    while (bytes.length < count) {
      // HMAC function used to output provided inputs into bytes
      const hmac = crypto.createHmac("sha256", ServerSeed);
      hmac.update(`${ClientSeed}:${nonce}:${currentRound}`);
      const buffer = hmac.digest();
      // Update cursor for next iteration of loop
      while (currentRoundCursor < 32 && bytes.length < count) {
        bytes.push(buffer[currentRoundCursor]);
        currentRoundCursor += 1;
      }
      currentRoundCursor = 0;
      currentRound += 1;
    }
    console.log("bytes:", bytes);
    return bytes;
  }

  function generateFloats({ ServerSeed, ClientSeed, nonce, cursor, count }) {
    const bytes = generateBytes({
      ServerSeed,
      ClientSeed,
      nonce,
      cursor,
      count: count * 4
    });
    // Convert bytes to floats
    const floats = [];
    for (let i = 0; i < bytes.length; i += 4) {
      let partial = 0;
      for (let j = 0; j < 4; j++) {
        partial += bytes[i + j] / 256 ** (j + 1);
      }
      floats.push(partial);
    }
    console.log("floats:", floats);
    return floats;
  }

  function game_evaluation() {
    if (activeTab === "limbo") {
      var game = "limbo";
      console.log("limbopath", game);
      let zwischenresult = verify[0];
      let endresult = zwischenresult * 16777216;
      console.log("trunc", Math.trunc(endresult));
      let endresult2 = (16777216 / (Math.trunc(endresult) + 1)) * (1 - 0.01);
      console.log("result", endresult2.toFixed(2));
      return endresult2.toFixed(2);
    }
    if (activeTab === "dice") {
      var game = "dice";
      console.log("dicepath", game);
      let endresult = verify[0] * 10001;
      let endresult2 = Math.trunc(endresult) / 100;
      console.log(endresult2);
      return endresult2;
    }
    if (activeTab === "diamonds") {
      var game = "diamonds";
      endresultgems = [];
      console.log("diamondspath", game);
      count = 5;
      for (let i = 0; i < count; i++) {
        let gem = GEMS[Math.floor(verify[i] * 7)];

        endresultgems.push(gem);
      }
      console.log("loop vorbei", endresultgems);
      gem1color = endresultgems[0];
      gem2color = endresultgems[1];
      gem3color = endresultgems[2];
      gem4color = endresultgems[3];
      gem5color = endresultgems[4];
      console.log(gem1color);

      return endresultgems;
    }
  }
</script>

<main>

<div class="parent">
<div class="div1"> </div>
<div class="div2">

<nav>
  	<a class:active={activeTab === 'limbo'} href="#" on:click={() => activeTab = "limbo"}>Limbo</a>
	  <a class:active={activeTab === 'dice'} href="#" on:click={() => activeTab = "dice"}>Dice</a>
	  <a class:active={activeTab === 'diamonds'} href="#" on:click={() => activeTab = "diamonds"}>Diamonds</a>
	  <a class:active={activeTab === 'wheel'} href="#" on:click={() => activeTab = "wheel"}>Wheel</a>  
</nav></div>
<div class="div3"> </div>


<div class="div4"> </div>
<div class="div5">
<div class="center">
    <input title="ServerSeed" bind:value={ServerSeed} placeholder="ServerSeed"/>
	<br>
	  <input title="ClientSeed" bind:value={ClientSeed} placeholder="ClientSeed"/>
	<br>
	  <input title="nonce" bind:value={nonce} placeholder="Nonce"/>
	<br>
</div>
  <div class="center">
	  <button on:click={game_evaluation_handler}>SHOW</button>
</div> </div> </div>
	<h1>RESULT</h1>
  
 

{#if isFieldsFilled && activeTab === "dice"} 
{resultat}
<div class="center">
<input type="range" class="slider" min="0.00" max="100.00" bind:value={resultat}/>
</div>
{:else if isFieldsFilled && activeTab === "diamonds"}
<!-- -----------------------------1st---------------------- -->
{#each game_evaluation(BetObject) as char, i}
			<span
				in:fade="{{delay: 1000 + i * 150, duration: 800}}"
			>{char},</span>
		{/each}
<div class="container">

 <div class="gear center container gem1">
 <svg transition:fade="{{duration: 2000 }}" height="45px" width="45px" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 viewBox="0 0 464 464" xml:space="preserve">
<g style="fill:{gem1color}">
	<polygon style="fill:#82C8DC;" points="0,156 232,428 128.889,156 	"/>
	<polygon style="fill:#B4E6F0;" points="335.111,156 232,428 464,156 	"/>
	<polygon style="fill:#D9FFFF;" points="232,156 335.111,156 232,36 128.889,156 	"/>
	<polygon style="fill:#A0DCE6;" points="232,156 128.889,156 232,428 335.111,156 	"/>
	<polygon style="fill:#A0DCE6;" points="104,36 0,156 128.889,156 	"/>
	<polygon style="fill:#EBFFFF;" points="464,156 360,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="360,36 232,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="232,36 104,36 128.889,156 	"/>
</g>
 </div>
 
<!-- -----------------------------2nd---------------------- -->


 <div class="gear center container gem2">
<svg height="45px" width="45px" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 viewBox="0 0 464 464" xml:space="preserve">
<g>
	<polygon style="fill:#82C8DC;" points="0,156 232,428 128.889,156 	"/>
	<polygon style="fill:#B4E6F0;" points="335.111,156 232,428 464,156 	"/>
	<polygon style="fill:#D9FFFF;" points="232,156 335.111,156 232,36 128.889,156 	"/>
	<polygon style="fill:#A0DCE6;" points="232,156 128.889,156 232,428 335.111,156 	"/>
	<polygon style="fill:#A0DCE6;" points="104,36 0,156 128.889,156 	"/>
	<polygon style="fill:#EBFFFF;" points="464,156 360,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="360,36 232,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="232,36 104,36 128.889,156 	"/>
</g>

</svg>
 </div>
 <!-- -----------------------------3rd---------------------- -->
<div class="gear center container gem3">
 
<svg height="45px" width="45px" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 viewBox="0 0 464 464" xml:space="preserve">
<g>
	<polygon style="fill:#82C8DC;" points="0,156 232,428 128.889,156 	"/>
	<polygon style="fill:#B4E6F0;" points="335.111,156 232,428 464,156 	"/>
	<polygon style="fill:#D9FFFF;" points="232,156 335.111,156 232,36 128.889,156 	"/>
	<polygon style="fill:#A0DCE6;" points="232,156 128.889,156 232,428 335.111,156 	"/>
	<polygon style="fill:#A0DCE6;" points="104,36 0,156 128.889,156 	"/>
	<polygon style="fill:#EBFFFF;" points="464,156 360,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="360,36 232,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="232,36 104,36 128.889,156 	"/>
</g>
 </div>
<!-- -----------------------------4th---------------------- -->
<div class="gear center container gem4">
 <svg height="45px" width="45px" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 viewBox="0 0 464 464" xml:space="preserve">
<g>
	<polygon style="fill:#82C8DC;" points="0,156 232,428 128.889,156 	"/>
	<polygon style="fill:#B4E6F0;" points="335.111,156 232,428 464,156 	"/>
	<polygon style="fill:#D9FFFF;" points="232,156 335.111,156 232,36 128.889,156 	"/>
	<polygon style="fill:#A0DCE6;" points="232,156 128.889,156 232,428 335.111,156 	"/>
	<polygon style="fill:#A0DCE6;" points="104,36 0,156 128.889,156 	"/>
	<polygon style="fill:#EBFFFF;" points="464,156 360,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="360,36 232,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="232,36 104,36 128.889,156 	"/>
</g>
 </div>
<!-- -----------------------------5th---------------------- -->
<div class="gear center container gem5">
 <svg height="45px" width="45px" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 viewBox="0 0 464 464" xml:space="preserve">
<g>
	<polygon style="fill:#82C8DC;" points="0,156 232,428 128.889,156 	"/>
	<polygon style="fill:#B4E6F0;" points="335.111,156 232,428 464,156 	"/>
	<polygon style="fill:#D9FFFF;" points="232,156 335.111,156 232,36 128.889,156 	"/>
	<polygon style="fill:#A0DCE6;" points="232,156 128.889,156 232,428 335.111,156 	"/>
	<polygon style="fill:#A0DCE6;" points="104,36 0,156 128.889,156 	"/>
	<polygon style="fill:#EBFFFF;" points="464,156 360,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="360,36 232,36 335.111,156 	"/>
	<polygon style="fill:#B4E6F0;" points="232,36 104,36 128.889,156 	"/>
</g>
 </div>
</div>
{:else}
test
{/if}
    
{#if isFieldsFilled}
 <table>
  <thead>
    <tr>
      <th>FLOATS</th>
    </tr>
  </thead>
  <tbody>
    {#each verify as item}
      <tr>
        <td>{item}</td>
      </tr>
    {/each}
  </tbody>
</table>
 
{/if}
  
<div class="div6"> </div>
<div class="div7"> </div>


	

	  
	

	
 
  


  
</main>


<style>
  /* @import "https://unpkg.com/carbon-components-svelte/css/g80.css"; */
  @import "https://unpkg.com/chota@latest";
  nav {
    margin-bottom: 1rem;
  }

  button {
    margin: 1rem;
  }
  main {
    max-height: 100%;
    text-align: center;
    padding: 1em;

    margin: 0 auto;
    background-color: #1a2c38;
    color: #ffffff;
  }
  h1 {
    color: #ff3e00;

    font-size: 2em;
    font-weight: 250;
  }
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }

  div {
    display: flex;
    justify-content: center;
    margin: 8px;
  }

  .box {
    display: flex;
    align-items: left;
    justify-content: left;
    width: 500px;
    height: 25px;
    overflow-wrap: break-word;
    display: inline-block;
  }
  .active {
    color: #ff3d03;
    font-weight: 700;
  }
  input {
    max-width: 555px;
    max-height: 45px;
    margin: 4px;
  }
  .center {
    margin: 0 auto;
  }
  input[type="range"].slider {
    width: 100%;
    margin: 4.35px 0;
    background-color: transparent;
    -webkit-appearance: none;
  }
  input[type="range"].slider:focus {
    outline: none;
  }
  input[type="range"].slider::-webkit-slider-runnable-track {
    background: #00e701;
    border: 7.8px solid #0f2114;
    border-radius: 10.3px;
    width: 100%;
    height: 22.3px;
    cursor: pointer;
  }
  input[type="range"].slider::-webkit-slider-thumb {
    margin-top: -12.15px;
    width: 30px;
    height: 31px;
    background: #4391e7;
    border: 1.5px solid #4365e8;
    border-radius: 3px;
    cursor: pointer;
    -webkit-appearance: none;
  }
  input[type="range"].slider:focus::-webkit-slider-runnable-track {
    background: #00ec01;
  }
  input[type="range"].slider::-moz-range-track {
    background: #00e701;
    border: 7.8px solid #0f2114;
    border-radius: 10.3px;
    width: 100%;
    height: 22.3px;
    cursor: pointer;
  }
  input[type="range"].slider::-moz-range-thumb {
    width: 30px;
    height: 31px;
    background: #4391e7;
    border: 1.5px solid #4365e8;
    border-radius: 3px;
    cursor: pointer;
  }
  input[type="range"].slider::-ms-track {
    background: transparent;
    border-color: transparent;
    border-width: 5.55px 0;
    color: transparent;
    width: 100%;
    height: 22.3px;
    cursor: pointer;
  }
  input[type="range"].slider::-ms-fill-lower {
    background: #00e201;
    border: 7.8px solid #0f2114;
    border-radius: 20.6px;
  }
  input[type="range"].slider::-ms-fill-upper {
    background: #00e701;
    border: 7.8px solid #0f2114;
    border-radius: 20.6px;
  }
  input[type="range"].slider::-ms-thumb {
    width: 30px;
    height: 31px;
    background: #4391e7;
    border: 1.5px solid #4365e8;
    border-radius: 3px;
    cursor: pointer;
    margin-top: 0px;
    /*Needed to keep the Edge thumb centred*/
  }
  input[type="range"].slider:focus::-ms-fill-lower {
    background: #00e701;
  }
  input[type="range"].slider:focus::-ms-fill-upper {
    background: #00ec01;
  }
  /*TODO: Use one of the selectors from https://stackoverflow.com/a/20541859/7077589 and figure out
                                        how to remove the virtical space around the range input in IE*/
  @supports (-ms-ime-align: auto) {
    /* Pre-Chromium Edge only styles, selector taken from hhttps://stackoverflow.com/a/32202953/7077589 */
    input[type="range"].slider {
      margin: 0;
      /*Edge starts the margin from the thumb, not the track as other browsers do*/
    }
  }

  .parent {
    display: grid;
    grid-template-columns: 1fr 2fr 1fr;
    grid-template-rows: 0.2fr 1fr 0.2fr;
    grid-column-gap: 5px;
    grid-row-gap: 5px;
  }

  .div1 {
    grid-area: 1 / 1 / 2 / 2;
  }
  .div2 {
    grid-area: 1 / 2 / 2 / 3;
  }
  .div3 {
    grid-area: 1 / 3 / 2 / 4;
  }
  .div4 {
    grid-area: 2 / 1 / 3 / 2;
  }
  .div5 {
    grid-area: 2 / 2 / 3 / 3;
  }
  .div6 {
    grid-area: 2 / 3 / 3 / 4;
  }
  .div7 {
    grid-area: 3 / 1 / 4 / 4;
  }

  .my-svg {
    color: currentColor;
  }

  .gear svg path {
  }

  .container {
    display: inline-flex;
    max-width: 350px;
  }

  .gem1 svg path {
    color: gem1color;
    fill: gem1color;
  }
</style>
