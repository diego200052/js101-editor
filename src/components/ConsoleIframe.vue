<template>
<iframe class="consoleIframe" :src="'/#/execCode/' + filenameToLoad + '/'+ refreshConsole"></iframe>
</template>

<script>
export default {
  props: ['filenameToLoad','refreshConsole'],
  name: 'ConsoleIframe',
  data () {
    return {
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.consoleIframe{
width:100%;
min-height:40vh;
}
iframe{
border:none;
}
</style>
