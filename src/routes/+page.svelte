<script lang="ts">
	import { onMount } from 'svelte';

	let pyodide: any;
	let loadedPyodide = false;
	let files: FileList;
	let pdfContent: string = '';

	onMount(async () => {
		// @ts-ignore
		pyodide = await loadPyodide();
		await pyodide.loadPackage('micropip');
		const micropip = pyodide.pyimport('micropip');
		await micropip.install('pdfminer.six');
		loadedPyodide = true;
	});

	function handleClick() {
		if (!files) return;
		const reader = new FileReader();
		reader.onload = async function (event) {
			if (!event?.target?.result) return;
			const rawData = event.target.result as ArrayBuffer;

			const pdfData = new Uint8Array(rawData);
			pyodide.FS.writeFile('/file.pdf', pdfData, { encoding: 'utf8' });
			const extractedText = pyodide.runPython(`
                from pdfminer.high_level import extract_text
                extract_text("/file.pdf")
            `);

			pdfContent = extractedText;
		};
		reader.readAsArrayBuffer(files[0]);
	}
</script>

<h1>Eloquent Examinator</h1>

{#if loadedPyodide}
	<form>
		<label for="pdf-file">Upload slides PDF:</label>
		<input type="file" bind:files id="pdf-file" accept="application/pdf" />
	</form>
	<button on:click={handleClick}>Extract Content</button>
	<div>
		<pre>{pdfContent}</pre>
	</div>
{:else}
	<p>Initalizing...</p>
{/if}
