<script>
  import { spring } from 'svelte/motion';
  import { fade, fly, slide } from 'svelte/transition';

  let ticketCount = 10;
  let isDrawing = false;
  let drawnNumbers = []; // Add array to store drawn numbers
  let displayNumber = spring(0, {
    stiffness: 0.1,
    damping: 0.4,
    precision: 1  // Increase precision to avoid floating points
  });
  let finalNumber = 0;
  let showFireworks = false; // New state for fireworks
  let isFinalFlash = false; // Add new state variable
  let startRange = 1000; // New variable for starting range
  let showNumber = false; // Add new state variable
  
  async function generateResult() {
    if (drawnNumbers.length >= ticketCount) {
      alert('All numbers have been drawn! Click Reset to start over.');
      return;
    }

    isDrawing = true;
    showFireworks = false;
    isFinalFlash = false;
    showNumber = true; // Show the number when drawing starts
    
    // Recalculate available numbers each time
    let availableNumbers = [...Array(ticketCount)].map((_, i) => startRange + i + 1)
    .filter(num => !drawnNumbers.includes(num));
    
    // Animate through random numbers (excluding drawn ones)
    for (let i = 0; i < 20; i++) {
      $displayNumber = availableNumbers[Math.floor(Math.random() * availableNumbers.length)];
      await new Promise(r => setTimeout(r, 100));
    }
    
    // Final result (excluding drawn numbers)
    finalNumber = availableNumbers[Math.floor(Math.random() * availableNumbers.length)];
    $displayNumber = finalNumber; // Show the final number after flashing
    
    // Update drawn numbers immediately after final number is drawn
    drawnNumbers = [...drawnNumbers, finalNumber];
    
    await new Promise(r => setTimeout(r, 500));
    isDrawing = false;
    showFireworks = true; // Trigger fireworks
    isFinalFlash = true;
    
    // Wait for fireworks animation before resetting final flash
    await new Promise(r => setTimeout(r, 2000));
    isFinalFlash = false;
    showNumber = false; // Hide the number after animation
  }

  function resetDrawn() {
    drawnNumbers = [];
    finalNumber = 0;
    $displayNumber = 0;
    showFireworks = false;
    isFinalFlash = false;
    showNumber = false;
  }
</script>

<main>
  <div class="container">
    <h1>Raffle Draw</h1>
    <div class="input-group-grid">
      <div class="input-group">
        <label for="startRange">Start:</label>
        <input 
          type="number" 
          id="startRange" 
          min="1" 
          bind:value={startRange}
          disabled={isDrawing && !isFinalFlash}
        >
      </div>

      <div class="input-group">
        <label for="tickets">Tickets:</label>
        <input 
          type="number" 
          id="tickets" 
          min="1" 
          bind:value={ticketCount}
          disabled={isDrawing && !isFinalFlash}
        >
      </div>
    </div>

    <div class="result" transition:fly>
      <div class="circle-gradient" class:spinning={showNumber}></div> <!-- Add this line -->
      {#if (!isDrawing && !showNumber)}
      <button 
        on:click={generateResult} 
        disabled={isDrawing || ticketCount < 1 || drawnNumbers.length >= ticketCount}
      >
        Draw Number
      </button>
      {/if}
      {#if (isDrawing || finalNumber > 0)}
      
        {#if showNumber}
        <div class="number" class:final-flash={isFinalFlash} class:drawing={isDrawing} transition:fade>
          {#if showFireworks}
            <div class="fireworks-container">
              {#each Array(12) as _, i}
                <div 
                  class="particle" 
                  style="--angle: {i * 30}deg; --delay: {i * 100}ms"
                  transition:fly={{ 
                    y: 100,
                    duration: 1500,
                    delay: i * 100,
                    opacity: 0.2
                  }}
                ></div>
              {/each}
            </div>
          {/if}
          #{Math.round($displayNumber)}
        </div>
      {/if}
      {/if}
      </div>

    {#if drawnNumbers.length > 0}
      <div class="drawn-numbers" transition:fade>
        <div class="number-list">
          {#each drawnNumbers as number, index (number)}
            <span 
              class="drawn-number {index === 0 ? 'gold' : index === 1 ? 'silver' : index === 2 ? 'bronze' : 'transparent'}"
              transition:fade
            >
              #{number}
            </span>
          {/each}
        </div>
      </div>
    {/if}
  </div>
</main>

<style>
  .container {
    max-width: 600px;
    margin: 2rem auto;
    text-align: center;
  }

  .input-group-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
    align-items: start; /* Align items to the start */
    background: #ffffff11;
    margin-bottom: 1rem;
  }

  .input-group {
    margin: 1.5rem 0;
  }

  label {
    font-weight: bold;
    font-size: 1.5rem;
  }

  input {
    padding: 0.5rem;
    font-size: 1.2rem;
  }

  button {
    padding: 1rem 2rem;
    font-size: 1.5rem;
    background-color: #54338f;
    color: white;
    border: none;
    border-radius: 40px;
    cursor: pointer;
    transition: background-color 0.3s;
    text-wrap: nowrap;
    align-self: center; /* Center buttons vertically */
  }
  
  button:hover {
    background-color: #63419f;
  }

  button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
    color: #555555;
  }

  .result {
    margin-top: 0;
    height: 300px;
    position: relative;
    display: flex; /* Add this line */
    justify-content: center; /* Add this line */
    align-items: center; /* Add this line */
  }

  .result button {
    z-index:100;
    position: absolute;
    box-shadow: 0px 2px 12px #000000cc;
  }

  .result button:hover {
    box-shadow: 0px 2px 10px #000000cc;

  }

  .circle-gradient {
    position: absolute;
    top: 0;
    width: 300px; /* Adjust size as needed */
    height: 300px; /* Adjust size as needed */
    border-radius: 50%;
    background-size: 100% 100%;
    background-position: 0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px,0px 0px;
    background-image: radial-gradient(18% 28% at 24% 50%, #CEFAFFFF 7%, #073AFF00 100%),radial-gradient(18% 28% at 18% 71%, #FFFFFF59 6%, #073AFF00 100%),radial-gradient(70% 53% at 36% 76%, #73F2FFFF 0%, #073AFF00 100%),radial-gradient(42% 53% at 15% 94%, #FFFFFFFF 7%, #073AFF00 100%),radial-gradient(42% 53% at 34% 72%, #FFFFFFFF 7%, #073AFF00 100%),radial-gradient(18% 28% at 35% 87%, #FFFFFFFF 7%, #073AFF00 100%),radial-gradient(31% 43% at 7% 98%, #FFFFFFFF 24%, #073AFF00 100%),radial-gradient(21% 37% at 72% 23%, #D3FF6D9C 24%, #073AFF00 100%),radial-gradient(35% 56% at 91% 74%, #8A4FFFF5 9%, #073AFF00 100%),radial-gradient(74% 86% at 67% 38%, #6DFFAEF5 24%, #073AFF00 100%),linear-gradient(125deg, #4EB5FFFF 1%, #4C00FCFF 100%);
    animation: rotate 20s linear infinite;
    z-index: 1; /* Ensure it is behind the number */
  }


  @keyframes rotate {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }

  .number {
    font-size: 5rem;
    font-weight: bold;
    color: #00000078;
    text-shadow: 2px 2px 1px #FFFFFF, 0px 0px 10px #000000;
    position: relative;
    z-index: 2;
  }

  .number.drawing {
    color: #322e34be;
  }

  .number.final-flash {
    color: #ffc940;
    animation: flash 0.5s infinite;
    position: relative; /* Add this line */
  }

  @keyframes flash {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.3; }
  }

  .fireworks-container {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 300px;
    height: 300px;
    transform: translate(-50%, -50%);
    pointer-events: none;
    margin-top: -20px;
  }

  .particle {
    position: absolute;
    width: 8px;  /* Increased from 4px */
    height: 40px; /* Increased from 20px */
    background: linear-gradient(to bottom, #ffffff, #4EB5FFFF);
    top: 50%;
    left: 50%;
    transform-origin: center;
    animation: shoot 2s ease-out forwards; /* Increased duration from 1.5s */
    opacity: 0;
  }

  @keyframes shoot {
    0% {
      transform: rotate(var(--angle)) translateY(0) scale(1);
      opacity: 1;
    }
    100% {
      transform: rotate(var(--angle)) translateY(200px) scale(0); /* Increased from 100px */
      opacity: 0;
    }
  }

  .drawn-numbers {
    margin-top: 2rem;
  }

  .number-list {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    justify-content: center;
    margin-top: 1rem;
  }

  .drawn-number {
    padding: 0.5rem 1rem;
    border-radius: 4px;
    font-size: 1.5rem;
    font-weight: bold;
    border: 2px solid rgba(255, 255, 255, 0.2);
    box-shadow: 4px 4px 1px black;
    text-shadow: 1px 1px rgba(255, 255, 255, 0.4);
  }

  @keyframes gradient {
    0% {
      background-position: 0% 50%;
    }

    50% {
      background-position: 100% 50%;
    }

    100% {
      background-position: 0% 50%;
    }
  }

  .drawn-number.gold {
    background-color: gold;
    color: black; /* Updated text color */
    background: linear-gradient(-45deg, goldenrod 37%, gold 50%, goldenrod 60%);

    background-size: 400% 400%;
    animation: gradient 6s ease infinite;
  }

  .drawn-number.silver {
    background-color: #b2becf;
    color: black; /* Updated text color */
    background: linear-gradient(-45deg, #b2becf 37%, #dae5f3 50%, #b2becf 60%);

    background-size: 400% 400%;
    animation: gradient 6s ease infinite;
  }

  .drawn-number.bronze {
    background-color: #cd7f32;
    color: black; /* Updated text color */
    background: linear-gradient(-45deg, #cd7f32 37%, #efa255 50%, #cd7f32 60%);

    background-size: 400% 400%;
    animation: gradient 6s ease infinite;
  }

  .drawn-number.transparent {
    background-color: transparent;
    text-shadow: 1px 1px rgba(0, 0, 0, 0.4);
  }

  .button-group {
    display: flex;
    gap: 1rem;
    justify-content: center;
    flex-wrap: nowrap;
    grid-column: span 2; /* Span both columns */
  }

  button + button {
    margin-left: 0; /* Remove existing margin since we're using gap */
  }

  input[type="number"] {
    field-sizing: content;
    min-width: 2em;
    padding: .5rem;
    font-family: 'Courier New', Courier, monospace;
    font-weight:bold;
    font-size: 2em
  }


input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button
{
  -webkit-appearance: none;
  margin: 0;
}
</style>