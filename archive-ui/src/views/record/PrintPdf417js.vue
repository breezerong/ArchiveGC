<template>
  <div>
      <!-- <div style="width:30%;display:inline-block;position: absolute; left:30px;">
        <el-select
            name="selectName"
            v-model="selectedPx"
            placeholder="'请选择条码尺寸'"
            style="display:block;"
            @change="refresh(archiveObjects,selectedPx)"
            >
            <el-option label="1条码" value="1" key='2'></el-option>
            <el-option label="2条码" value="2" key='2'></el-option>
            <el-option label="3条码" value="3" key='3'></el-option>
            <el-option label="4条码" value="4" key='4'></el-option>
            <el-option label="5条码" value="5" key='5'></el-option>
            <el-option label="6条码" value="6" key='6'></el-option>
          </el-select>
          
      </div> -->
      <div style="display:inline-block;position: absolute;top:40px;left:480px;">
        <el-row>
          <el-select v-model="printType" @change="onPrintTypeChange" >
            <template v-for="item in printTypeList">
                <el-option :label="item" :value="item" :key="item"></el-option>
            </template>
          </el-select>
        </el-row>
        <el-row>
          <button @click="printCode" v-print="'#print'">打印</button>
        </el-row>
      </div>
      <div id='print' ref='print' :style="'position: absolute; top:0px;'">
        <div v-for="(item,keys) in printObjects" :key="'divk'+keys" style="width:400px">
          <el-row>
            <el-col :span="12" style="color: #000000;text-align: left;font-size:18px;padding:4px;">{{item.typeName}}</el-col>
            <el-col :span="6" style="color: #000000;text-align: left;font-size:18px;padding:4px;">{{item.itemType}}</el-col>
            <el-col :span="6" style="color: #000000;text-align: left;font-size:18px;padding:4px;">{{printType}}</el-col>
          </el-row>
          <el-row>
            <el-col :span="24" style="color: #000000;text-align: left;font-size:28px;padding:4px;">{{item.coding}}&nbsp;</el-col>
          </el-row>
          <el-row>
            <el-col :span="12" style="color: #000000;text-align: left;font-size:18px;padding:4px;">密级：{{item.securityLevel}}</el-col>
            <el-col v-if="item.revision" :span="12" style="color: #000000;text-align: left;font-size:18px;padding:4px;">版本：{{item.revision}}</el-col>
          </el-row>
          <el-row>
            <el-col :span="12" style="color: #000000;text-align: left;font-size:18px;padding:4px;">保管期限：{{item.retention}}</el-col>
            <el-col :span="12" style="color: #000000;text-align: left;font-size:18px;padding:4px;">工程号：{{item.projectCode}}</el-col>
          </el-row>
          <el-row>
            <el-col :span="24" style="color: #000000;text-align: left;font-size:18px;padding:4px;">归档日期：{{item.archiveDate}}</el-col>
          </el-row>
          <el-row style="padding-bottom:15px;">
            <el-col :span="10" style="padding-top:10px;">
              <el-row style="color: #000000;text-align: left;font-size:18px;padding:2px;">{{item.volString}}</el-row>
              <el-row style="color: #000000;text-align: center;font-size:46px;padding-top:10px;">{{item.storeCoding}}</el-row>
            </el-col>
            <el-col :span="14"><canvas :ref="'canvas'+keys" :style="'display:'+noneStr"></canvas><img :ref="'image'+keys" /></el-col>
          </el-row>
        </div>
        <!-- <div v-if="isQRCode"  ref='qrCodeUrl2'></div> -->
  　　</div>
  </div>
</template>

<script type="text/javascript">
import Print from '@/plugins/print'
import PDF417 from '@/plugins/pdf417'
import Vue from 'vue';
import QRCode from 'qrcodejs2'// 引入qrcode
import JsBarcode from 'jsbarcode'
Vue.use(Print)
Vue.use(PDF417);
export default {
   name: 'printPDF147Code',
    
  // name: "printPage",
  data() {
    return {
      archiveTitle:"",
      archiveCode:"",
      innerDataList:[],
      dialogQrcodeVisible: true,
      currentLanguage: "zh-cn",
      volumeTitle:"",
      ridgeData:[],
      selectedPx:'112',
      noneStr:"block",
      barCodeWidth:2,
      barCodeHeight:40,
      printType:"原件",
      printTypeListGeneral:["原件","复制件"],
      printTypeListBussiness:["正本","副本","复制件"],
      printTypeList:[],
      printObjects:[],
    };
  },
  mounted() {
    let _self = this;
    _self.printTypeList = _self.printTypeListGeneral;
    _self.currentLanguage = localStorage.getItem("localeLanguage") || "zh-cn";
    _self.getConfigParam("PrintArchiveCodeConfig");
    
  },
  props: {
    archiveObjects:{type:Array,default:() => []},
    divWidth:{type:String,default:'400px'},
    divHeight:{type:String,default:'50px'},
    divMargin:{type:String,default:'10px'},
  },
  methods: {
    loadData(data){
      let _self = this;
      if(data){
        _self.archiveObjects = data;
      }
      for(let i=0;i< _self.archiveObjects.length;i++){
        let obj = _self.archiveObjects[i];
        if(obj["C_ARC_CLASSIC"] && obj["C_ARC_CLASSIC"]=="商务管理"){
          _self.printTypeList = _self.printTypeListBussiness;
          _self.printType ="正本";
          break;
        }
      }
      setTimeout(() => {
        _self.getPrintObjects();
        }, 100
      );
    },
      getConfigParam(keyName) {
        let _self = this;
        axios
          .post("/dc/getJsonParamMap", keyName)
          .then(function(response) {
            _self.divWidth = response.data.data.divWidth;
            _self.divHeight = response.data.data.divHeight;
            _self.divMargin = response.data.data.divMargin;
          })
          .catch(function(error) {
            console.log(error);
          });
      },
      onPrintTypeChange(val){
        this.printType = val;
        this.getPrintObjects();
      },
      refresh(){
        let _self=this;
        setTimeout(() => {
          for(let i=0; i < _self.printObjects.length;i++){
            let obj = _self.printObjects[i];
            let barcode = obj.id +";" +obj.archiveCoding + ";"+obj.coding+";"+obj.revision+";";//+";"+obj.typeName+";"+obj.printType;
            _self.generate(barcode,_self.$refs['canvas'+i][0]);
          }
        },100);
        
      },

    getPrintObjects(){
      let _self=this;
      var m = new Map();
      _self.loading = true;
      let ids = [];
      for(let i=0;i< _self.archiveObjects.length;i++){
        let obj = _self.archiveObjects[i];
        ids.push(obj["ID"]);
      }
      m.set('ids',ids);//ID
      m.set('printType',_self.printType);
      _self.axios.post("/record/print/getPrintData", JSON.stringify(m))
        .then(function(response) {
          _self.printObjects = response.data.data;
          _self.refresh();
          _self.loading = false;
        })
        .catch(function(error) {
          console.log(error);
          _self.loading = false;
        });
    },
    printCode(){
      this.noneStr = "none";
      if(this.printObjects && this.printObjects.length>0){
        for(var i=0; i<this.printObjects.length; i++){
          this.$refs["image"+i][0].src = this.$refs['canvas'+i][0].toDataURL();
        }
      }
      setTimeout(() => {
      this.$print(this.$refs.print);
    }, 200);
    },
    
    generate(content,showCanvas) {
        this.PDF417.init(content);             

        let barcode = this.PDF417.getBarcodeArray();

        // block sizes (width and height) in pixels
        let bw = 2;
        let bh = 2;

        // create canvas element based on number of columns and rows in barcode
        

        let canvas = showCanvas;
        canvas.width = bw * barcode['num_cols'];
        canvas.height = bh * barcode['num_rows'];
        
        let ctx = canvas.getContext('2d');                    

        // graph barcode elements
        let y = 0;
        // for each row
        for (let r = 0; r < barcode['num_rows']; ++r) {
            let x = 0;
            // for each column
            for (let c = 0; c < barcode['num_cols']; ++c) {
                if (barcode['bcode'][r][c] == 1) {                        
                    ctx.fillRect(x, y, bw, bh);
                }
                x += bw;
            }
            y += bh;
        }       
    }
    
    
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style media="printContent" scoped>
@page {
size: auto; /* auto is the initial value /
margin: 3mm; / this affects the margin in the printer settings */
}

html {
background-color: #ffffff;
margin: 0px; /* this affects the margin on the html before sending to printer */
}

body {
border: solid 1px rgba(255,255,255,0);
/* margin: 10mm 15mm 10mm 15mm; margin you want for the content */
}
</style>
