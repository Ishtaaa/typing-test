<script lang="ts">
  import { onMount } from 'svelte';

  type GameState = 'waiting' | 'in-progress' | 'finished';

  interface TypingStats {
    wpm: number;
    accuracy: number;
    totalWords: number;
    correctWords: number;
    totalCharacters: number;
    correctCharacters: number;
    timeElapsed: number;
  }

  // Sample text for typing test
  const sampleTexts = [
    "The quick brown fox jumps over the lazy dog. This pangram contains every letter of the alphabet at least once.",
    "Programming is the art of telling another human being what one wants the computer to do. It requires logical thinking and creativity.",
    "Success is not final, failure is not fatal: it is the courage to continue that counts. Every day brings new opportunities.",
    "The only way to do great work is to love what you do. If you haven't found it yet, keep looking and don't settle.",
    "Innovation distinguishes between a leader and a follower. The future belongs to those who believe in the beauty of their dreams."
  ];

  let gameState: GameState = 'waiting';
  let currentText = sampleTexts[Math.floor(Math.random() * sampleTexts.length)];
  let words = currentText.split(' ');
  let currentWordIndex = 0;
  let currentCharIndex = 0;
  let typedText = '';
  let startTime: number | null = null;
  let endTime: number | null = null;
  let timer: number | null = null;
  let countdownTimer: number | null = null;
  let timeElapsed = 0;
  let timeRemaining = 30; //  countdown
  let stats: TypingStats = {
    wpm: 0,
    accuracy: 0,
    totalWords: words.length,
    correctWords: 0,
    totalCharacters: currentText.length,
    correctCharacters: 0,
    timeElapsed: 0
  };

  let inputElement: HTMLInputElement;

  function startGame() {
    gameState = 'in-progress';
    startTime = Date.now();
    timeRemaining = 30;
    
    // Timer for elapsed time
    timer = setInterval(() => {
      if (startTime) {
        timeElapsed = (Date.now() - startTime) / 1000;
      }
    }, 100);
    
    // Countdown timer
    countdownTimer = setInterval(() => {
      timeRemaining--;
      if (timeRemaining <= 0) {
        endGame();
      }
    }, 1000);
  }

  function endGame() {
    gameState = 'finished';
    endTime = Date.now();
    if (timer) {
      clearInterval(timer);
    }
    if (countdownTimer) {
      clearInterval(countdownTimer);
    }
    calculateStats();
  }

  function calculateStats() {
    if (!startTime || !endTime) return;

    const timeInMinutes = (endTime - startTime) / 60000;
    const totalTypedWords = typedText.trim().split(/\s+/).length;
    
    let correctWords = 0;
    const typedWords = typedText.trim().split(/\s+/);
    const originalWords = currentText.split(' ');
    
    for (let i = 0; i < Math.min(typedWords.length, originalWords.length); i++) {
      if (typedWords[i] === originalWords[i]) {
        correctWords++;
      }
    }

    // Calculate accuracy
    const accuracy = stats.totalCharacters > 0 
      ? Math.round((stats.correctCharacters / stats.totalCharacters) * 100)
      : 0;

    stats = {
      wpm: timeInMinutes > 0 ? Math.round(correctWords / timeInMinutes) : 0,
      accuracy,
      totalWords: words.length,
      correctWords,
      totalCharacters: currentText.length,
      correctCharacters: stats.correctCharacters,
      timeElapsed: timeElapsed
    };
  }

  function handleInput(event: Event) {
    const target = event.target as HTMLInputElement;
    const value = target.value;
    
    if (gameState === 'waiting') {
      startGame();
    }

    typedText = value;
    
    // Update character tracking
    let correctChars = 0;
    let totalTypedChars = 0;
    
    const typedWords = value.trim().split(/\s+/);
    const originalWords = currentText.split(' ');
    
    for (let i = 0; i < Math.min(typedWords.length, originalWords.length); i++) {
      const typedWord = typedWords[i];
      const originalWord = originalWords[i];
      
      for (let j = 0; j < Math.min(typedWord.length, originalWord.length); j++) {
        totalTypedChars++;
        if (typedWord[j] === originalWord[j]) {
          correctChars++;
        }
      }
    }
    
    stats.correctCharacters = correctChars;
  }

  function resetGame() {
    gameState = 'waiting';
    currentText = sampleTexts[Math.floor(Math.random() * sampleTexts.length)];
    words = currentText.split(' ');
    currentWordIndex = 0;
    currentCharIndex = 0;
    typedText = '';
    startTime = null;
    endTime = null;
    timeElapsed = 0;
    timeRemaining = 30;
    stats = {
      wpm: 0,
      accuracy: 0,
      totalWords: words.length,
      correctWords: 0,
      totalCharacters: currentText.length,
      correctCharacters: 0,
      timeElapsed: 0
    };
    
    if (timer) {
      clearInterval(timer);
    }
    if (countdownTimer) {
      clearInterval(countdownTimer);
    }
    
    if (inputElement) {
      inputElement.value = '';
      inputElement.focus();
    }
  }

  function getWordClass(wordIndex: number, typedWords: string[]) {
    if (wordIndex >= typedWords.length) return 'text-gray-600';
    
    const typedWord = typedWords[wordIndex];
    const originalWord = words[wordIndex];
    
    if (typedWord === originalWord) {
      return 'text-green-500';
    } else if (typedWord && originalWord.startsWith(typedWord)) {
      return 'text-yellow-500';
    } else {
      return 'text-red-500';
    }
  }

  function getCharClass(charIndex: number, wordIndex: number, typedWords: string[]) {
    if (wordIndex >= typedWords.length) return 'text-gray-600';
    
    const typedWord = typedWords[wordIndex];
    const originalWord = words[wordIndex];
    
    if (charIndex >= typedWord.length) return 'text-gray-600';
    if (charIndex >= originalWord.length) return 'text-red-500';
    
    return typedWord[charIndex] === originalWord[charIndex] 
      ? 'text-green-500' 
      : 'text-red-500';
  }

  onMount(() => {
    if (inputElement) {
      inputElement.focus();
    }
  });
</script>

<svelte:head>
  <title>Typing Test</title>
</svelte:head>

<div class="min-h-screen bg-gray-900 text-white flex flex-col items-center justify-center p-4">
  <div class="max-w-4xl w-full space-y-8">
    <!-- Header -->
    <div class="text-center">
      <h1 class="text-4xl font-bold text-blue-400 mb-2">Typing Speed Test</h1>
      <p class="text-gray-400">Test your typing speed and accuracy</p>
    </div>

    <!-- Stats Display -->
    {#if gameState === 'in-progress' || gameState === 'finished'}
      <div class="grid grid-cols-2 md:grid-cols-4 gap-4 bg-gray-800 p-4 rounded-lg">
        <div class="text-center">
          {#if gameState === 'in-progress'}
            <div class="text-2xl font-bold text-red-400">{timeRemaining}s</div>
            <div class="text-sm text-gray-400">Time Left</div>
          {:else}
            <div class="text-2xl font-bold text-blue-400">{Math.round(timeElapsed)}s</div>
            <div class="text-sm text-gray-400">Time Used</div>
          {/if}
        </div>
        <div class="text-center">
          <div class="text-2xl font-bold text-green-400">{stats.wpm}</div>
          <div class="text-sm text-gray-400">WPM</div>
        </div>
        <div class="text-center">
          <div class="text-2xl font-bold text-yellow-400">{stats.accuracy}%</div>
          <div class="text-sm text-gray-400">Accuracy</div>
        </div>
        <div class="text-center">
          <div class="text-2xl font-bold text-purple-400">{stats.correctCharacters}/{stats.totalCharacters}</div>
          <div class="text-sm text-gray-400">Characters</div>
        </div>
      </div>
    {/if}

    <!-- Typing Area -->
    <div class="bg-gray-800 p-6 rounded-lg">
      <!-- Input Field -->
      <div class="mb-6">
        <input
          bind:this={inputElement}
          type="text"
          class="w-full p-4 text-lg bg-gray-700 border border-gray-600 rounded-lg focus:outline-none focus:border-blue-500 text-white placeholder-gray-400"
          placeholder="Start typing here..."
          on:input={handleInput}
          disabled={gameState === 'finished'}
        />
      </div>

      <!-- Text Display -->
      <div class="bg-gray-700 p-6 rounded-lg min-h-[200px] leading-relaxed">
        {#if gameState === 'waiting'}
          <p class="text-gray-400 text-center">Click start typing to begin the test</p>
        {:else}
          <div class="text-lg">
            {#each words as word, wordIndex}
              <span class="word inline-block">
                {#each word.split('') as char, charIndex}
                  <span class="char {getCharClass(charIndex, wordIndex, typedText.trim().split(/\s+/))}">
                    {char}
                  </span>
                {/each}
              </span>
              {#if wordIndex < words.length - 1}
                <span class="char text-gray-600">&nbsp;</span>
              {/if}
            {/each}
          </div>
        {/if}
      </div>
    </div>

    <!-- Results Section -->
    {#if gameState === 'finished'}
      <div class="bg-gray-800 p-6 rounded-lg">
        <h2 class="text-2xl font-bold text-center text-green-400 mb-6">Test Complete!</h2>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div class="text-center bg-gray-700 p-4 rounded-lg">
            <div class="text-3xl font-bold text-green-400">{stats.wpm}</div>
            <div class="text-lg text-gray-300">Words Per Minute</div>
          </div>
          <div class="text-center bg-gray-700 p-4 rounded-lg">
            <div class="text-3xl font-bold text-yellow-400">{stats.accuracy}%</div>
            <div class="text-lg text-gray-300">Accuracy</div>
          </div>
          <div class="text-center bg-gray-700 p-4 rounded-lg">
            <div class="text-3xl font-bold text-blue-400">{stats.correctWords}/{stats.totalWords}</div>
            <div class="text-lg text-gray-300">Words Completed</div>
          </div>
        </div>
        <div class="mt-6 text-center">
          <p class="text-gray-400 mb-4">You typed {stats.correctCharacters} correct characters out of {stats.totalCharacters} total characters</p>
          <p class="text-gray-400">Time used: {Math.round(timeElapsed)} seconds</p>
        </div>
      </div>
    {/if}

    <!-- Controls -->
    <div class="flex justify-center space-x-4">
      {#if gameState === 'waiting'}
        <button
          class="px-6 py-3 bg-blue-600 hover:bg-blue-700 rounded-lg font-semibold transition-colors"
          on:click={() => startGame()}
        >
          Start Typing
        </button>
      {:else if gameState === 'finished'}
        <button
          class="px-6 py-3 bg-green-600 hover:bg-green-700 rounded-lg font-semibold transition-colors"
          on:click={resetGame}
        >
          Try Again
        </button>
      {/if}
    </div>

    <!-- Instructions -->
    {#if gameState === 'waiting'}
      <div class="bg-gray-800 p-4 rounded-lg">
        <h3 class="text-lg font-semibold text-blue-400 mb-2">Instructions:</h3>
        <ul class="text-gray-400 space-y-1 text-sm">
          <li>• Type the text exactly as shown above</li>
          <li>• Your speed (WPM) and accuracy will be tracked in real-time</li>
          <li>• Green text indicates correct characters, red indicates mistakes</li>
          <li>• The test ends after 30 seconds automatically</li>
          <li>• Try to type as much as you can within the time limit</li>
        </ul>
      </div>
    {/if}
  </div>
</div>
  