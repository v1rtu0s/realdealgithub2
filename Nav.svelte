<script>
export let activeTab = "limbo";

$: console.log(activeTab);
</script>








<nav>
  
	<a class:active={activeTab === 'limbo'} href="#" on:click={() => activeTab = "limbo"}>Limbo</a>
	  <a class:active={activeTab === 'dice'} href="#" on:click={() => activeTab = "dice"}>Dice</a>
	  <a class:active={activeTab === 'diamonds'} href="#" on:click={() => activeTab = "diamonds"}>Diamonds</a>
	  <a class:active={activeTab === 'wheel'} href="#" on:click={() => activeTab = "wheel"}>Wheel</a>  
	</nav>
	
  <style>
  .active {
    color: #ff3d03;
    font-weight: 700;
  }
</style>