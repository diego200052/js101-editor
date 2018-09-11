<template>
  <q-layout view="lHh Lpr lFf">
    <q-layout-header>
		 <q-tabs v-model="selectedTab">
		  <template v-for="(child, index) in childrenTabs">
		  <q-tab :name="child.route" class="routeTabStyle" slot="title">
		  <q-btn @click="showTab(child.route)" class="filenameButton" flat dense> {{ child.label }} </q-btn>    
		  <q-btn @click="closeFile(child.route)" class="closeButton" flat round dense>X</q-btn>
		  </q-tab>
		  </template>
		  <q-tab slot="title" name="adding" @click="add()" label="+"/>
		 </q-tabs>

    </q-layout-header>

    <q-layout-drawer
      v-model="leftDrawerOpen"
      :content-class="$q.theme === 'mat' ? 'bg-grey-2' : null"
    >
      <q-list
        no-border
        inset-delimiter
      >
        <q-list-header>Archivos</q-list-header>
        
        <template v-for="(child, index) in filesList">
        <q-item>
          <q-item-side right>
            <q-btn flat round dense icon="delete" @click="deleteFile(child.route)"></q-btn>
          </q-item-side>
          <q-item-main :label="child.label" />
          <q-item-side link right>
              <router-link tag="div" square @click.native="openFile(child.route)" class="shadow-2 openSquare bg-secondary primary" :to="child.route">
				Abrir
			  </router-link>
		   </q-item-side>
        </q-item>
		</template>
		<q-btn v-if="filesList.length>0" @click="exportFiles()" class="bg-secondary exportFileButton"><span>Bajar Archivos</span> <q-icon name="cloud_download" /></q-btn>

        
         </q-list>
    </q-layout-drawer>

    <q-page-container>
      <router-view :key="$route.fullPath"/>
      <q-btn v-if="$route.path != '/'" id="execBtn" label="Ejecutar" color="secondary" @click="execCode()"> <q-icon name="send" /></q-btn>
     <ConsoleIframe v-if="$route.path != '/'" :filenameToLoad="fileToRunConsole" :refreshConsole="refreshConsole" ref="consoleIframeInst"></ConsoleIframe>
    </q-page-container>
  </q-layout>
</template>

<script>
import { openURL } from 'quasar'
import ConsoleIframe from '.././components/ConsoleIframe.vue'
import JSZip from 'jszip'


export default {
  name: 'MyLayout',
  components: { ConsoleIframe },
  data () {
    return {
      refreshConsole: new Date().getTime() / 1000,
      fileToRunConsole: "console.js", 
      leftDrawerOpen: this.$q.platform.is.desktop,
      filesList: this.getFileSavedList(),
      childrenTabs: [],
      selectedTab: ""
    }
  },
  methods: {
    execCode (){
     this.fileToRunConsole = this.$route.path.replace("/editor/","");
     this.refreshConsole = new Date().getTime() / 1000;
    },
    openFile (route){
    var findTab = this.childrenTabs.filter(file=>file.route == route);
    if(findTab.length>0){
    }else{ 
	this.childrenTabs.push(this.filesList.filter(file=>file.route == route)[0]);	       
    }
    
    this.selectedTab = route;    
    },
    showTab (route){
    this.$router.push({ path: route });
    },
	add () {      
this.$q.dialog({
  title: 'Nuevo Archivo',
  message: 'Nombre del archivo',
  prompt: {
    model: '',
    type: 'text' // optional
  },
  cancel: true,
  color: 'secondary'
}).then(data => {	
    if(data==""){
    this.$q.notify('El nombre del archivo no puede estar vacio!');
    }else{
	var filenameStr = data.replace(" ","_");
	if(this.filesList.filter(file=>file.filename == filenameStr).length>0){
	this.$q.notify('El nombre del archivo ya existe! Elije otro!');
	}else{
	var newObjFile = {route:"/editor/"+filenameStr,label:data,filename:filenameStr,code:""};
    this.$q.localStorage.set(newObjFile.route,newObjFile);       
	this.childrenTabs.push(newObjFile);	
	this.filesList.push(newObjFile);
	this.$router.push({ path: newObjFile.route });   
	this.selectedTab = newObjFile.route;
 
	
	}
	}
	
}).catch(() => {
})

    },
    getFileSavedList () {
       if(this.$q.localStorage.isEmpty()){       
       this.$q.localStorage.set("/editor/console.js",{route:"/editor/console.js",label:"console.js",filename:"console.js",code:"console.log(101)"});       
       }
       
       var listFiles = [];
       var storedFiles = this.$q.localStorage.get.all();
       for(var key in storedFiles){
       if(key!='syntaxError'){       
       listFiles.push(storedFiles[key]);
       }
       }
       return listFiles;
    },
    deleteFile (route){
     
    this.$q.dialog({
  title: 'Borrar Archivo',
  message: 'Estas seguro que quieres borrar el archivo '+this.filesList.filter(file=>file.route == route)[0].filename,
  ok: 'Ok',
  cancel: 'Cancelar'
}).then(() => {
  this.$q.localStorage.remove(route);
  this.filesList = this.filesList.filter(file=>file.route != route);
  this.childrenTabs = this.childrenTabs.filter(file=>file.route != route);
  if(this.$route.path ==  route){
  if(this.childrenTabs.length>0){
  this.$router.push({ path: this.childrenTabs[0].route });
  this.selectedTab = this.childrenTabs[0].route;    
  }else{
  this.$router.push({ path: "/" });      
  }
  }
  this.$q.notify('Archivo borrado!');
}).catch(() => {
});
    
      
    }, 
    closeFile (route){
    this.childrenTabs = this.childrenTabs.filter(file=>file.route != route);
	if(this.$route.path ==  route){
    if(this.childrenTabs.length>0){
	this.$router.push({ path: this.childrenTabs[0].route });
	this.selectedTab = this.childrenTabs[0].route;    
	}else{
    console.log("change tab root");
	this.$router.push({ path: "/" });      
	}  
    }
    }, 
    exportFiles (){
    
    var zip = new JSZip();
    this.filesList.forEach(function(item,key){
    zip.file(item.filename, item.code);    
    });
    
    zip.generateAsync({type:"base64"}).then(function (base64) {
        window.location = "data:application/zip;base64," + base64;
    }, function (err) {
            this.$q.notify('No se pudo crear el archivo zip!');
    });
	zip.generateAsync({type:"blob"})
	.then(function(content) {
    saveAs(content, "js101-archivos.zip");
	});
    
    },
    openURL
  },
  mounted() {
       var findTab = this.filesList.filter(file=>file.route == this.$route.path);
       if(findTab.length>0){
       this.childrenTabs.push(findTab[0]);
       this.selectedTab = findTab[0].route;
       }else{
       this.$router.push({ path: "/" });    
       }
  }
}
</script>

<style>
.modal-header{
font-weight:400;
}

#execBtn{
width:100%;
}
.closeButton{
display:inline-block;
}
.openSquare{
padding:10px;
cursor: pointer;
color: white;
}
.q-tabs-head{
background-color: #141618 !important;
padding-left:0px !important;
}
.q-tab{
background-color: #272822 !important;
}
.q-layout{
background-color: #141618 !important;
}
.exportFileButton{
width:100%;
color:white;
margin-top:10px;
}
.exportFileButton span{
margin-right:10px;
display:inline-block;
}
.routeTabStyle{
flex-direction: row;
padding-left:0px;
padding-top:0px;
padding-bottom:0px;
padding-right:10px;
}
.filenameButton{
padding-left:25px;
padding-top:8px;
padding-bottom:8px;
height:100%;
}


.filenameButton:hover q-focus-helper, .filenameButton:focus q-focus-helper, .filenameButton.q-hoverable, .filenameButton .q-focus-helper{
background-color:transparent !important;
}
</style>
