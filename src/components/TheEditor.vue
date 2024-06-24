<script setup lang="ts">
import { onMounted, ref } from 'vue'
import * as monaco from 'monaco-editor'
import editorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker'
import jsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker'
import cssWorker from 'monaco-editor/esm/vs/language/css/css.worker?worker'
import htmlWorker from 'monaco-editor/esm/vs/language/html/html.worker?worker'
import tsWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker'
import { completeCode } from '../services/completion'

// Import the workers according to <https://github.com/vitejs/vite/discussions/1791#discussioncomment-321046>
self.MonacoEnvironment = {
  getWorker(_, label) {
    if (label === 'json') {
      return new jsonWorker()
    }
    if (label === 'css' || label === 'scss' || label === 'less') {
      return new cssWorker()
    }
    if (label === 'html' || label === 'handlebars' || label === 'razor') {
      return new htmlWorker()
    }
    if (label === 'typescript' || label === 'javascript') {
      return new tsWorker()
    }
    return new editorWorker()
  }
}

const container = ref<HTMLDivElement | null>(null)

const value = `function hello() {
  alert('Hello world!')
}`

const language = 'javascript'

monaco.languages.registerInlineCompletionsProvider(language, {
  provideInlineCompletions: async (
    model: monaco.editor.ITextModel,
    position: monaco.Position,
  ) => {
    const currentLine = model.getLineContent(position.lineNumber)
    const offset = model.getOffsetAt(position)
    
    // The code before the current line.
    const textBeforeCursor = model.getValue().substring(0, offset - currentLine.length)

    // The code on the current line before the cursor.
    const textBeforeCursorOnCurrentLine = currentLine.substring(0, position.column - 1)
    
    const completion = await completeCode(
      textBeforeCursor,
      textBeforeCursorOnCurrentLine,
      language,
    )

    const startLineNumber = position.lineNumber
    const startColumn = position.column
    const endLineNumber = position.lineNumber
    const endColumn = position.column + completion.length
    return Promise.resolve({
      items: [
        {
          insertText: completion,
          range: new monaco.Range(startLineNumber, startColumn, endLineNumber, endColumn),
        },
      ]
    })
  },
  freeInlineCompletions: () => {},
})

onMounted(() => {
  if (container.value === null)  return
  monaco.editor.create(container.value, {
    value,
    language,
    automaticLayout: true,
    wordBasedSuggestions: 'off',
    inlineSuggest: {
      enabled: true,
      showToolbar: 'onHover',
      mode: 'subword',
      suppressSuggestions: false,
    },
  })
})
</script>

<template>
  <div
    ref="container"
    style="height: 100%; width: 100%;"
  />
</template>
