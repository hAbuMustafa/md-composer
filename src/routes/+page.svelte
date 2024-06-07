<script lang="ts">
  import { browser } from '$app/environment';
  import { marked } from 'marked';

  let raw_md = !!browser
    ? localStorage.getItem('md-raw') ?? '# Hello, World!'
    : '# Hello, World!';

  let styles = !!browser
    ? localStorage.getItem('md-styles') ??
      `h1,h2,h3,h4,h5,h6 {
  text-transform: capitalize;
}`
    : `h1,h2,h3,h4,h5,h6 {
  text-transform: capitalize;
}`;

  $: raw = `${raw_md}
  <style>${styles}</style>`;

  function handleLinesShift(e: KeyboardEvent) {
    if (!e.altKey) return;
    if (e.key !== 'ArrowUp' && e.key !== 'ArrowDown') return;

    e.preventDefault();

    const textArea = e.target as HTMLTextAreaElement;

    const lines = textArea.value.replace(/\\r\\n/g, '\n').split('\n');

    const { selectionStart, selectionEnd } = textArea;
    const selectionFirstLineIndex =
      textArea.value.substring(0, selectionStart).split('\n').length - 1;
    const selectionLastLineIndex =
      textArea.value.substring(0, selectionEnd).split('\n').length - 1;

    const charsFromLineStartForSelectionStart =
      selectionStart - lines.slice(0, selectionFirstLineIndex).join('\n').length;

    const charsFromLineStartForSelectionEnd =
      selectionEnd - lines.slice(0, selectionLastLineIndex).join('\n').length;

    if (e.key === 'ArrowUp') {
      if (selectionFirstLineIndex > 0) {
        const movingLines = lines.splice(
          selectionFirstLineIndex,
          selectionLastLineIndex - selectionFirstLineIndex + 1
        );
        lines.splice(selectionFirstLineIndex - 1, 0, ...movingLines);

        textArea.value = lines.join('\n');

        textArea.selectionStart =
          lines.slice(0, selectionFirstLineIndex - 1).join('\n').length +
          charsFromLineStartForSelectionStart;
        textArea.selectionEnd =
          lines.slice(0, selectionLastLineIndex - 1).join('\n').length +
          charsFromLineStartForSelectionEnd;
      }
    } else if (e.key === 'ArrowDown') {
      if (selectionLastLineIndex < lines.length - 1) {
        const movingLines = lines.splice(
          selectionFirstLineIndex,
          selectionLastLineIndex - selectionFirstLineIndex + 1
        );
        lines.splice(selectionFirstLineIndex + 1, 0, ...movingLines);

        textArea.value = lines.join('\n');

        textArea.selectionStart =
          lines.slice(0, selectionFirstLineIndex + 1).join('\n').length +
          charsFromLineStartForSelectionStart;
        textArea.selectionEnd =
          lines.slice(0, selectionLastLineIndex + 1).join('\n').length +
          charsFromLineStartForSelectionEnd;
      }
    }
  }

  function saveAll() {
    localStorage.setItem('md-raw', raw_md);
    localStorage.setItem('md-styles', styles);
  }
</script>

<h1>Markdown Composer</h1>

<textarea
  name="md-raw"
  id="md-raw"
  style:width="70%"
  rows="10"
  class="md"
  bind:value={raw_md}
  on:keydown={handleLinesShift}
  on:input={saveAll}
/>

<textarea
  name="styles-raw"
  id="styles-raw"
  style:width="25%"
  rows="10"
  class="styles"
  bind:value={styles}
  on:input={saveAll}
/>

{#await marked.parse(raw)}
  <p>...parsing</p>
{:then html}
  <iframe
    srcdoc={html}
    frameborder="16"
    title="Markdown Preview"
    class="preview"
    height="550vh"
    width="97.5%"
  />
{/await}

<!-- todo: add file saving to local storage (for both the raw markdown and the parsed html) -->

<!-- todo: add multi-caret support -->

<!-- todo: add syntax highlighting for the raw markdown and styles -->

<!-- fix: shifting lines with alt+arrow up/down doesn't save updates nor update the preview -->

<style>
  textarea {
    font-family: monospace;
    font-size: 1.2em;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 10px;
  }

  iframe {
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 10px;
  }
</style>
