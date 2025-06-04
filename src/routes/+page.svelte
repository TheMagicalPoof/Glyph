<script lang="ts">
  import { onMount } from 'svelte';
  import { charMap } from '$lib/charMap';

  onMount(() => {
    const check = () => {
      isMobile = window.innerWidth <= 768;
    };
    check();
    window.addEventListener('resize', check);
    return () => window.removeEventListener('resize', check);
  });

  export function clickOutside(node: HTMLElement, callback: () => void) {
    const handleClick = (event: MouseEvent) => {
      if (!node.contains(event.target as Node)) { callback() };
    };

    document.addEventListener('click', handleClick, true);

    return {
      update(newCallback: () => void) {
        callback = newCallback;
      },
      destroy() {
        document.removeEventListener('click', handleClick, true);
      }
    };
  };
  let prevInput = '';
  let scrollContainer: HTMLElement | null; // контейнер с DOM элементом иероглифов
  let isMobile = false; // мобилка или нет(по ширине экрана)
  let showNotification = false;
  let notificationText = '';
  let input = '';
	let cachedChars: string[] = [];
	let transformed = '';

	let editingIndex: number | null = null;
	let pickerRef;

    function getRandomCharFor(char: string): string {
		const options = charMap[char.toUpperCase()];
		if (!options || options.length === 0) return char;
		const i = Math.floor(Math.random() * options.length);
		return options[i];
	}

  // Функция рандомного выбора иероглифа из вариантов
  function randomVariant(char: string) {
    const variants = charMap[char.toUpperCase()];
    if (!variants) return char;  // Если вариантов нет — возвращаем исходный символ
    return variants[Math.floor(Math.random() * variants.length)];
  }

	function rerandomize() {
		cachedChars = input.split('').map(getRandomCharFor);
		transformed = cachedChars.join('');
	}

	function selectChar(index: number, value: string) {
    
		cachedChars[index] = value;
		transformed = cachedChars.join('');
		editingIndex = null;
	}

  function copyToClipboard(text: string) {
      navigator.clipboard.writeText(text).then(() => {
          notificationText = 'Скопировано!';
          showNotification = true;
      });
  }

  $: if (input !== prevInput) {
  updateTransformed();
}
function updateTransformed() {
  const newChars = input.split('');
  const oldChars = cachedChars; // берем прошлую трансформацию, а не prevInput

  const newCached: string[] = [];

  for (let i = 0; i < newChars.length; i++) {
    const newChar = newChars[i];
    if (oldChars[i] && charMap[newChar.toUpperCase()] && oldChars[i]) {
      // если символ не изменился - берем старую трансформацию
      if (input[i] === prevInput[i]) {
        newCached.push(oldChars[i]);
      } else {
        newCached.push(randomVariant(newChar));
      }
    } else {
      newCached.push(randomVariant(newChar));
    }
  }

  cachedChars = newCached;
  transformed = cachedChars.join('');
  prevInput = input;
}

</script>

<!-- svelte-ignore css_unused_selector -->
<style>
  @keyframes fadeSlideIn {
    0% {
      opacity: 0;
      transform: translateY(0.5rem);
    }
    20% {
      opacity: 1;
    }
    60% {
      opacity: 1;
    }
    100% {
      opacity: 0;
      transform: translateY(-0.5rem);
    }
  }

  .animate-fadeSlideIn {
    animation: fadeSlideIn 1s ease-out;
  }

	.hoverable:hover {
		background-color: rgba(255, 255, 255, 0.1);
		cursor: pointer;
		transition: background-color 0.2s;
	}

    :global(body) {
        background: var(--color-gray-900);
    }

	textarea {
		background: #1e1e2f;
		color: #ddd;
		border: 1px solid #444;
		font-family: monospace;
		resize: none;
	}

	.transformed-box {
		background: var(--color-gray-900);
		color: #eee;
		border: 1px solid #444;
		font-family: monospace;
		position: relative;
		overflow-wrap: break-word;
	}

  /* Webkit-скроллбар */
  ::-webkit-scrollbar {
    width: 8px;
    height: 8px;
  }

  ::-webkit-scrollbar-thumb {
    background-color: #4b5563; /* gray-700 */
    border-radius: 4px;
    cursor: pointer;
  }

  ::-webkit-scrollbar-track {
    background-color: #1f2937; /* gray-900 */
  }
  
  .seo-text {
    font-size: 0.75rem;
    color: var(--color-gray-700);
    margin-top: 2rem;
    max-width: 600px;
    line-height: 1.2;
    user-select: none;
  }

  textarea.clean {
    all: unset;
    white-space: pre-wrap;
    word-break: break-word;
    width: 100%;
    height: 100%;
    padding: 0;
    background-color: none; /* Пример: gray-800 */
    color: #e5e7eb;            /* Пример: gray-200 */
    box-sizing: border-box;
    resize: none;
    line-height: 1.5;
  }
</style>

<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<link rel="canonical" href="https://red.dim/hieroglyphicizer" />
<meta name="description" content="Онлайн инструмент для преобразования русского и латинского текста в стилизованные иероглифы. Введите слово и получите уникальную иероглифическую транслитерацию.">
<meta name="keywords" content="иероглифы, транслитерация, онлайн конвертер, иероглификатор, текст в иероглифы">
<meta name="robots" content="index, follow">
<div class="min-h-screen bg-gray-900 text-gray-300 p-6 flex flex-col max-w-4xl mx-auto space-y-6">
  <h1 class="select-none text-3xl font-semibold text-gray-500">ИЕРОГЛИФИКАТОР: Текст в Иероглифы</h1>

  <div class="flex flex-col md:flex-row  items-stretch gap-4 h-[400px]">
    <div class="flex-1 bg-gray-800 rounded-lg p-4 h-full box-border text-gray-300">
      <textarea
      bind:value={input}
      placeholder="Введите текст..."
      class="clean">
      
    </textarea>
  </div>
    

    <div bind:this={scrollContainer} class="flex rotate-90 md:rotate-0 items-center justify-center text-gray-500 text-3xl select-none px-2">
      &gt;
    </div>

    <div class="flex-1 whitespace-pre-wrap  bg-gray-800 rounded-lg p-4 font-mono overflow-auto overflow-y-auto h-[400px]">
      {#each cachedChars as ch, i}
        <!-- svelte-ignore a11y_click_events_have_key_events -->
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <div class="relative inline-block">
          <span
          class="inline cursor-pointer hover:bg-gray-700 rounded text-gray-300"
          class:bg-blue-600={editingIndex === i}
          on:click={() => {
            const c = input[i];
            if (charMap[c?.toUpperCase()]) editingIndex = i;
          }}
        >{ch}</span><!--
          -->{#if editingIndex === i && !isMobile}
        <div bind:this={pickerRef} use:clickOutside={() => (editingIndex = null)} class="absolute top-full mt-1 left-0 bg-gray-700  rounded-md p-2 z-50">
          {#each charMap[input[i]?.toUpperCase()] ?? [] as variant}
            <div class="px-2 py-1 rounded cursor-pointer hover:bg-blue-600 hover:text-white" on:click={() => selectChar(i, variant)}>{variant}</div>
          {/each}
        </div>
        {/if}
      </div>

        
      {/each}

      
    </div>
  </div>

  {#if isMobile} 
  <div class="flex items-center overflow-x-auto space-x-3 rounded shadow-inner min-h-[48px] min-w-[48px]">
    {#if editingIndex !== null}
  
      <!-- Кнопки вариантов -->
      <!-- svelte-ignore a11y_no_static_element_interactions -->
      {#each charMap[input[editingIndex]?.toUpperCase()] ?? [] as variant}
        <!-- svelte-ignore a11y_click_events_have_key_events -->
        <div
          class="min-w-[48px] px-3 py-2 bg-gray-800 rounded cursor-pointer text-center hover:bg-blue-600 hover:text-white select-none"
          on:click={() => selectChar(editingIndex!, variant)}
        >
          {variant}
        </div>
      {/each}


  
      {#if (charMap[input[editingIndex]?.toUpperCase()] ?? []).length === 0}
        <div class="min-w-[48px] px-3 py-2 select-none" aria-hidden="true">&nbsp;</div>
      {/if}
    {:else}
      <div class="min-w-[48px] px-3 py-2 select-none opacity-0" aria-hidden="true">&nbsp;</div>
    {/if}
  </div>
  {/if}

  <div class="flex h-full sw:h-1/2 justify-center md:justify-end space-x-6 md:space-x-12">
    <button
      class="w-1/2 md:w-1/5 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-lg px-4 py-2 transition-colors  cursor-pointer
      "
      class:opacity-50={!transformed}
      class:cursor-not-allowed={!transformed}
      class:active:scale-105={transformed}
      class:transition-transform={transformed}
      class:duration-150={transformed}
      class:ease-out={transformed}
      disabled={!transformed}
      title="Случайные иероглифы"
      on:click={rerandomize}
    >
      🎲 Рандом
    </button>
    <button
      class="w-1/2 md:w-1/5 bg-green-600 hover:bg-green-700 text-white font-semibold rounded-lg px-4 py-2 transition-colors relative inline-block cursor-pointer"

      class:opacity-50={!transformed}
      class:cursor-not-allowed={!transformed}
      class:active:scale-105={transformed}
      class:transition-transform={transformed}
      class:duration-150={transformed}
      class:ease-out={transformed}
      disabled={!transformed}
      title="Скопировать результат"
      on:click={() => copyToClipboard(transformed)}
    >
      📋 Копировать

      {#if showNotification}
      <div
      on:animationend={() => {showNotification = false}}
      class="absolute bottom-full right-0 mb-2 px-3 py-1 rounded bg-gray-700 text-white text-sm shadow-lg z-50
             animate-fadeSlideIn ">
        {notificationText}
      </div>
      {/if}
    </button>
  </div>



  <section class="seo-text">
      <h1>Иероглификатор — онлайн конвертер текста в иероглифы</h1>
      <p>Введите любой текст на русском или английском языке, и наш инструмент преобразует его в стилизованные иероглифы, создавая уникальную визуальную запись.</p>
      <p>Идеально подходит для создания необычных никнеймов, арт-надписей и оформления сообщений.</p>
    </section>
  <title>Иероглификатор — транслит текста в иероглифы онлайн</title>
</div>