<template>
<div class="bodyIframe">
Archivo: {{ filename }}<br>
Hora: {{refreshConsole | formatDate}}<br>
<br>
<div v-if = "consoleResult.length>0" v-for="argLine in consoleResult">
      <span v-if = "argLine.length>0" v-for="arg in argLine">
          {{ arg }}
      </span>
  </div>
</div>
</template>

<script>

import moment from 'moment'


var console = {}; 
var globalReturn = []; 
console.log = function(){ globalReturn.push(arguments); };
var iframe;

function f(c) {
   'use strict';
    eval(c);
}

export default {
  name: 'ConsoleExec',
  props: ['filename','refreshConsole'],
  data () {
    return {
        consoleResult : [],
        lastRefreshConsole : 0
    }
  },
  filters: {

	formatDate: function(value) {
		if (value) {
    return moment().format('MM/DD/YYYY hh:mm')
		}
	}
  },
  methods: {
   processSyntaxError () {
  		     var errorLines = window.localStorage.getItem('syntaxError').split("\n");
  		     errorLines[1] = "at "+this.filename+" (line: "+(parseInt(errorLines[1].split(":")[1])-1)+":"+(parseInt(errorLines[1].split(":")[2])-1)+")";
  		     errorLines[2] = "";
			 errorLines = errorLines.map(i => [i]);
			 this.consoleResult = errorLines;             
		}
  },
  updated() {
  if(this.lastRefreshConsole!=this.refreshConsole){	
    this.consoleResult = [];
    globalReturn = [];    
    
    try {
    
		f(this.$q.localStorage.get.item('/editor/'+this.filename).code);
		this.consoleResult = globalReturn;
	} catch (e) {
         var errorLines = e.stack.split("\n"); 
         errorLines = errorLines.map(i => [i]);    
         var replaceStr;
         if(e instanceof ReferenceError){
         replaceStr = 'at eval (eval at f (webpack-internal:///./node_modules/babel-loader/lib/index.js?!./node_modules/vue-loader/lib/index.js?!./src/components/ConsoleExec.vue?vue&type=script&lang=js&), <anonymous>';
         }else if(e instanceof SyntaxError){
         //we execute the code again in a isolated enviorment from vue. the reason for this, is that the line number reported by the first eval, is not correct.
         window.localStorage.setItem('syntaxError',this.$q.localStorage.get.item('/editor/'+this.filename).code);
         if(iframe!=undefined){
         document.body.removeChild(iframe);
         iframe=undefined;
         }
         iframe = document.createElement('iframe');
         iframe.src = '/statics/js-exec.html';
         iframe.style.width = "640px";
         iframe.style.height = "480px";
         iframe.style.display = 'none';
         document.body.appendChild(iframe);
         setTimeout(this.processSyntaxError,500);
                  
         replaceStr = false;
         }else if(e instanceof TypeError){
         replaceStr = 'at eval (eval at f (webpack-internal:///./node_modules/babel-loader/lib/index.js?!./node_modules/vue-loader/lib/index.js?!./src/components/ConsoleExec.vue?vue&type=script&lang=js&), <anonymous>';
         }
         
         if(replaceStr!=false){
         errorLines[1][0] = errorLines[1][0].replace(replaceStr,'at '+ this.filename + '(line ');         
         this.consoleResult = errorLines;    
         }
	}
	this.lastRefreshConsole = this.refreshConsole;
	}
	
  }   
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.bodyIframe{
background-color: #272822 !important;
height:100vh;
color:white;
padding-left:10px;
padding-top:10px;
}
</style>
