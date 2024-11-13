<script>
  import { spring } from 'svelte/motion';
  import { fade, fly } from 'svelte/transition';

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
  
  async function generateResult() {
    if (drawnNumbers.length >= ticketCount) {
      alert('All numbers have been drawn! Click Reset to start over.');
      return;
    }

    isDrawing = true;
    showFireworks = false;
    isFinalFlash = false;
    let availableNumbers = [...Array(ticketCount)].map((_, i) => i + 1)
    .filter(num => !drawnNumbers.includes(num));
    
    // Animate through random numbers (excluding drawn ones)
    for (let i = 0; i < 20; i++) {
      $displayNumber = availableNumbers[Math.floor(Math.random() * availableNumbers.length)];
      await new Promise(r => setTimeout(r, 100));
    }
    
    // Final result (excluding drawn numbers)
    finalNumber = availableNumbers[Math.floor(Math.random() * availableNumbers.length)];
    $displayNumber = finalNumber; // Show the final number after flashing
    
    await new Promise(r => setTimeout(r, 500));
    isDrawing = false;
    showFireworks = true; // Trigger fireworks
    isFinalFlash = true;
    
    // Wait for fireworks animation before updating drawn numbers
    await new Promise(r => setTimeout(r, 2000));
    isFinalFlash = false;
    drawnNumbers = [...drawnNumbers, finalNumber]; // Moved this here
  }

  function resetDrawn() {
    drawnNumbers = [];
    finalNumber = 0;
    $displayNumber = 0;
    showFireworks = false;
    isFinalFlash = false;
  }
</script>

<main>
  <div class="container">
    <h1>Raffle Picker</h1>
    
    <div class="input-group">
      <label for="tickets">Number of tickets:</label>
      <input 
        type="number" 
        id="tickets" 
        min="1" 
        bind:value={ticketCount}
        disabled={isDrawing && !isFinalFlash}
      >
    </div>

    <div class="button-group">
      <button 
        on:click={generateResult} 
        disabled={isDrawing || ticketCount < 1 || drawnNumbers.length >= ticketCount}
      >
        Draw Number
      </button>

      <button 
        on:click={resetDrawn}
        disabled={isDrawing || drawnNumbers.length === 0}
      >
        Reset
      </button>
    </div>

    {#if isDrawing || finalNumber > 0}
      <div class="result" transition:fade>
        <h2>Result:</h2>
        
        <div class="number" class:final-flash={isFinalFlash} class:drawing={isDrawing}>
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
      </div>
    {/if}

    {#if drawnNumbers.length > 0}
      <div class="drawn-numbers" transition:fade>
        <h3>Winners:</h3>
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

  .input-group {
    margin: 2rem 0;
  }

  input {
    margin-left: 1rem;
    padding: 0.5rem;
    font-size: 1.2rem;
  }

  button {
    padding: 1rem 2rem;
    font-size: 1.5rem;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
    text-wrap: nowrap;
  }

  button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
  }

  .result {
    margin-top: 2rem;
    position: relative;
  }

  .number {
    font-size: 4rem;
    font-weight: bold;
    color: #2196F3;
    position: relative;
    z-index: 2;
  }

  .number.drawing {
    color: #2196F3;
  }

  .number.final-flash {
    color: #ff4081;
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
    width: 100%;
    height: 100%;
    transform: translate(-50%, -50%);
    pointer-events: none;
    margin-top: -0.1em;
  }

  .particle {
    position: absolute;
    width: 4px;
    height: 20px;
    background: linear-gradient(to bottom, #ff4081, #ffeb3b);
    top: 50%;
    left: 50%;
    transform-origin: center;
    animation: shoot 1.5s ease-out forwards;
    opacity: 0;
  }

  @keyframes shoot {
    0% {
      transform: rotate(var(--angle)) translateY(0) scale(1);
      opacity: 1;
    }
    100% {
      transform: rotate(var(--angle)) translateY(100px) scale(0);
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

  .drawn-number.gold {
    background-color: gold;
    color: black; /* Updated text color */
  }

  .drawn-number.silver {
    background-color: #b2becf;
    color: black; /* Updated text color */
  }

  .drawn-number.bronze {
    background-color: #cd7f32;
    color: black; /* Updated text color */
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
  }

  button + button {
    margin-left: 0; /* Remove existing margin since we're using gap */
  }
</style>