<template>
<div id="codeMirrorWrap">
  <codemirror :options="{
    mode: 'javascript',
    theme: 'monokai',
    lineNumbers: true,
    tabSize: 4,
    styleActiveLine: false,
    styleSelectedText: false,
    line: true,
    foldGutter: true,
    gutters: ['CodeMirror-linenumbers', 'CodeMirror-foldgutter'],
    highlightSelectionMatches: { showToken: /\w/, annotateScrollbar: true },
    hintOptions:{
    completeSingle: false
    },
    keyMap: 'sublime',
	matchBrackets: true,
	showCursorWhenSelecting: true,
    extraKeys: {'Ctrl-Space': 'autocomplete'}        
  }" @change="preSaveData" ref="myEditor"></codemirror>
  </div>
</template>

<script>

import { codemirror } from 'vue-codemirror-lite'
require('codemirror/mode/javascript/javascript')
require('codemirror/mode/vue/vue')

require('codemirror/addon/hint/show-hint.js')
require('codemirror/addon/hint/show-hint.css')
require('codemirror/addon/hint/javascript-hint.js')
require('codemirror/theme/monokai.css')
require('codemirror/addon/selection/active-line.js')
require('codemirror/addon/selection/mark-selection.js')
require('codemirror/addon/search/searchcursor.js')
require('codemirror/addon/scroll/annotatescrollbar.js')
require('codemirror/addon/search/matchesonscrollbar.js')
require('codemirror/addon/search/searchcursor.js')
require('codemirror/addon/search/match-highlighter.js')
require('codemirror/mode/clike/clike.js')
require('codemirror/addon/edit/matchbrackets.js')
require('codemirror/addon/comment/comment.js')
require('codemirror/addon/dialog/dialog.js')
require('codemirror/addon/dialog/dialog.css')
require('codemirror/addon/search/searchcursor.js')
require('codemirror/addon/search/search.js')
require('codemirror/keymap/sublime.js')
require('codemirror/addon/fold/foldgutter.css')
require('codemirror/addon/fold/brace-fold.js')
require('codemirror/addon/fold/comment-fold.js')
require('codemirror/addon/fold/foldcode.js')
require('codemirror/addon/fold/foldgutter.js')
require('codemirror/addon/fold/indent-fold.js')
require('codemirror/addon/fold/markdown-fold.js')
require('codemirror/addon/fold/xml-fold.js')
import'codemirror/addon/selection/active-line.js'
import'codemirror/addon/hint/show-hint.js'
import'codemirror/addon/hint/show-hint.css'
import'codemirror/addon/hint/javascript-hint.js'

export default {
  data () {
    return {
      code: 'const str = "hello world"',
      fileObj : this.$q.localStorage.get.item(this.$route.path)
    }
  },
  name: 'CodeMirrorTab',
  components: { codemirror },
  methods:{
  preSaveData (){
  this.fileObj.code = this.$refs.myEditor.editor.getValue();
  this.$q.localStorage.set(this.$route.path, this.fileObj);
  }
  },
  computed: {
    editor() {
      // get current editor object
      return this.$refs.myEditor.editor
    }
  },
  mounted() {
    // use editor object...
    this.editor.focus();
    this.fileObj = this.$q.localStorage.get.item(this.$route.path);
    this.editor.setValue(this.fileObj.code);
  }
}
</script>

<style>
#codeMirrorWrap{
clear: both;
}
.CodeMirror {
height:40vh;
}
</style>
