<template>
  <v-app>
    <Login @loginTeknisi="loginTeknisi" v-if="!isLogin"/>
    <Hello @logout="logout" v-if="isLogin"/>
  </v-app>
</template>

<script>
import Login from './components/Login'
import Hello from './components/Hello'

export default {
  name: 'App',
  components: {
    Login,
    Hello
  },
  created(){
    if(localStorage.getItem('@idTeknisi')){
      this.isLogin = true
      this.idAdmin = localStorage.getItem('@idTeknisi')
    }
  },
  data: ()=>({
    isLogin:false,
    idAdmin:''
  }),
  methods:{
    logout(){
      this.isLogin = false
    },
    loginTeknisi(idTeknisi){
      console.log('masuk')
      this.isLogin = true
      this.idTeknisi = idTeknisi
    }
  }
}
</script>
<style>
*{
  margin: 0px;
  padding: 0px;
}
p{
  margin:1px;
}
*{
  -webkit-tap-highlight-color: rgba(0,0,0,0);
  
}/* width */
::-webkit-scrollbar {
  width: 5px;
}

/* Track */
::-webkit-scrollbar-track {
  background: #f1f1f1; 
}
 
/* Handle */
::-webkit-scrollbar-thumb {
  background: #888; 
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  background: #555; 
  width: 10px;

}
</style>