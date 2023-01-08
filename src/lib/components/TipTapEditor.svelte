<script>
  import { onMount, onDestroy } from 'svelte';
  import { Editor } from '@tiptap/core';
  import StarterKit from '@tiptap/starter-kit';
  import { textEditorHtml } from '../store';

  let element;
  let editor;

  onMount(() => {
    editor = new Editor({
      element: element,
      extensions: [StarterKit],
      content: 
      `
      `
      ,
      editorProps: {
        attributes: {
          class: 'ProseMirror',
        },
      },
      onTransaction: () => {
        // force re-render so `editor.isActive` works as expected
        editor = editor;
        textEditorHtml.set(editor);
      },
    });
  });

  onDestroy(() => {
    if (editor) {
      editor.destroy();
    }
  });
</script>

{#if editor}
  <div class="spacer">
    <button
      on:click={() => console.log && editor.chain().focus().toggleBold().run()}
      disabled={!editor.can().chain().focus().toggleBold().run()}
      class={editor.isActive('bold') ? 'active' : ''}
    >
      <i class="bi bi-type-bold" />
    </button>
    <button
      on:click={() => editor.chain().focus().toggleItalic().run()}
      disabled={!editor.can().chain().focus().toggleItalic().run()}
      class={editor.isActive('italic') ? 'active' : ''}
    >
      <i class="bi bi-type-italic" />
    </button>
    <!-- <button
      on:click={() => editor.chain().focus().toggleHeading({ level: 1 }).run()}
      class:active={editor.isActive('heading', { level: 1 }) ? 'active' : ''}
    >
      <i class="bi bi-type-h1" />
    </button> -->
    <!-- <button
      on:click={() => editor.chain().focus().toggleHeading({ level: 2 }).run()}
      class:active={editor.isActive('heading', { level: 2 }) ? 'active' : ''}
    >
      <i class="bi bi-type-h1" />
    </button> -->
    <button
      on:click={() => editor.chain().focus().toggleHeading({ level: 3 }).run()}
      class={editor.isActive('heading', { level: 3 }) ? 'active' : ''}
    >
      <i class="bi bi-type-h1" />
    </button>
    <button
      on:click={() => editor.chain().focus().toggleHeading({ level: 4 }).run()}
      class={editor.isActive('heading', { level: 4 }) ? 'active' : ''}
    >
      <i class="bi bi-type-h2" />
    </button>
    <button
      on:click={() => editor.chain().focus().toggleHeading({ level: 5 }).run()}
      class={editor.isActive('heading', { level: 5 }) ? 'active' : ''}
    >
      <i class="bi bi-type-h3" />
    </button>
    <button
      on:click={() => editor.chain().focus().setParagraph().run()}
      class:active={editor.isActive('paragraph')}
    >
      <i class="bi bi-paragraph" />
    </button>
    <button
      on:click={() => editor.chain().focus().toggleBulletList().run()}
      class={editor.isActive('bulletList') ? 'active' : ''}
    >
      <i class="bi bi-list-ul" />
    </button>
    <button
      on:click={() => editor.chain().focus().toggleOrderedList().run()}
      class={editor.isActive('orderedList') ? 'active' : ''}
    >
      <i class="bi bi-list-ol" />
    </button>
  </div>
{/if}

<div bind:this={element} />

<style>
  button{
    border: solid #3366FF;
    background: white;
    color: #3366FF;
    border-radius: 0.3em;
  }
  button.active {
    background: #3366FF;
    color: white;
    border: solid #3366FF;
  }
  .spacer {
    margin-bottom: 10px;
  }
  :global(div.ProseMirror) {
    padding: 0.5em;
    border: 1px solid #D6E0FF;
    outline-color: #3366FF;
    border-radius: 0.5em;
  }
</style>
