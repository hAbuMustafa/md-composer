<script lang="ts">
  import { marked } from 'marked';

  let raw_md = '# Hello, World!';

  let styles = `h1,h2,h3,h4,h5,h6 {
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
</script>

<h1>Markdown Composer</h1>

<textarea
  name="md-raw"
  id="md-raw"
  style:width="73%"
  rows="10"
  bind:value={raw_md}
  on:keydown={handleLinesShift}
/>
<textarea
  name="styles-raw"
  id="styles-raw"
  style:width="25%"
  rows="10"
  bind:value={styles}
/>

{#await marked.parse(raw)}
  <p>...parsing</p>
{:then html}
  <iframe
    srcdoc={html}
    frameborder="16"
    title="Markdown Preview"
    height="550vh"
    width="98%"
  />
{/await}
