<template>
    <iframe key="#1" :src="dsoUrl" frameborder="0" width="100%" :height="itemHeight" ></iframe>
</template>

<script type="text/javascript">
import 'url-search-params-polyfill'

export default {
  name: "DSOframerHtml5",
  data() {
    return {
      dsoUrl:"",
      itemHeight: window.innerHeight - 50
    };
  },
  props: {
    id:{type:String},
    format:{type:String}
  },
  created() {
    if(this.id==null && this.$route.query.id){
      this.id = this.$route.query.id;
    }
    if(this.format==null && this.$route.query.format){
      this.format = this.$route.query.format;
    }
    this.loadUrl();
    this.writeAudit(this.id);
  },
  methods: {
    loadUrl() {
      let _self = this;
      _self.loading = true;
      let key = "OwaPdfUrl";
      if(_self.format=="doc" || _self.format=="docx"){
        key = "OwaWordUrl";
      }else if(_self.format=="xlsx" || _self.format=="xls"){
        key = "OwaExcelUrl";
      }else if(_self.format=="ppt" || _self.format=="pptx"){
        key = "OwaPowerpointUrl";
      }

     //let getfileUrl =  _self.axios.defaults.baseURL+"/dc/getContent?id="+_self.id+"&token="+sessionStorage.getItem('access-token');
    let getfileUrl =  _self.axios.defaults.baseURL+"/dc/getContent?id="+_self.id+"&token="+sessionStorage.getItem('access-token')+"&format="+_self.format;
     _self.dsoUrl="./static/dsoframeviewerh5/dsoframe.html?file="+encodeURIComponent(getfileUrl)+"&."+_self.format;
    },
    writeAudit(docId){
      var m = new Map();
      m.set("docId",docId)
      m.set("actionName","ecm_read")
      m.set("appName","portal")
      axios
        .post("/audit/addAudit", JSON.stringify(m))
        .then(function(response){
          
        })
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
