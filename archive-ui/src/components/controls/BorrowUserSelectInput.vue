<template>
  <el-container >
    <el-dialog
      v-dialogDrag
      :visible.sync="visible"
      :append-to-body="true"
      :close-on-click-modal="false"
      @open="refreshData"
      @close="closeDialog"
      :title="$t('application.selectUser')"
      width="70%"
    >
      <div>
        <el-header>
          <el-input  :placeholder="$t('application.placeholderSearch')" @keyup.enter.native="refreshData" v-model="findValue"></el-input>
        </el-header>
        <el-main>
          <el-row>
            <el-col :span="11">
              <el-table
                height="250"
                :data="dataList"
                ref="leftTable"
                stripe
                border
                size="mini"
                 @row-dblclick="leftDbClick"
                 @row-click="leftClick"
                @selection-change="handleSelectionChange"
              >
                <el-table-column type="selection" width="60"></el-table-column>
                <el-table-column prop="name" :label="$t('application.userName')" width="120"></el-table-column>
                <el-table-column prop="departmentName" :label="$t('application.group')" width="120"></el-table-column>
                <el-table-column prop="email" :label="$t('application.email')"></el-table-column>
              </el-table>
            </el-col>
            <el-col :span="2">
              <div style="text-align:center;padding-top: 40px;">
                <el-button
                  size="mini"
                  type="infor"
                  @click="addToRight()"
                  :disabled="(!isRepeat&&rightList.length==1)||(isRepeat&&rightList.length>=maxCount)"
                >
                  <i class="el-icon-arrow-right"></i>
                </el-button>
                <br />
                <br />
                <br />
                <el-button size="mini" type="infor" @click="addToLeft()">
                  <i class="el-icon-arrow-left"></i>
                </el-button>
              </div>
            </el-col>
            <el-col :span="11">
              <el-table
                height="250"
                :data="rightList"
                ref="rightTable"
                stripe
                border
                size="mini"
                @row-click="rightClick"
                @row-dblclick="rightDbClick"
                @selection-change="handleRightSelectionChange"
              >
                <el-table-column type="selection" width="60"></el-table-column>
                <el-table-column prop="name" :label="$t('application.userName')" width="120"></el-table-column>
                <el-table-column prop="departmentName" :label="$t('application.group')" width="120"></el-table-column>
                <el-table-column prop="email" :label="$t('application.email')"></el-table-column>
              </el-table>
            </el-col>
          </el-row>
        </el-main>
        <el-footer>
          <el-button
            style="height: 35px;width: 70px;float: right;"
            type="primary"
            @click="addToFather"
          >{{$t('application.ok')}}</el-button>
        </el-footer>
      </div>
    </el-dialog>
    <el-col >
      <el-input style="width:200px" type="text" :placeholder="$t('application.selectUser')" readonly="readonly" v-model="inputValue"></el-input>
      <input value="value1" type="hidden" />
    </el-col>
    <el-col >
      <el-button :disabled="buttonType" icon="el-icon-user-solid" @click="clickShowDialog">{{$t('application.select')}}</el-button>
    </el-col>
  </el-container>
</template>
<script>
export default {
  data() {
    return {
      visible: false,
      findValue: "",
      dataList: [],
      rightList: [],
      tranList2: [],
      tranList: [],
      rightNameList: "",
      rightListId: []
    };
  },
  model: {
    prop: "value1",
    event: "change"
  },
  mounted() {},
  props: {
    //输入框默认显示值
    inputValue: {
      type: String,
      default: ""
    },
    isRepeat: {
      type: Boolean,
      default: false
    },
    maxCount: {
      type: Number,
      default: 50
    },
    roleName: {
      type: String,
      default: ""
    },
    noGroup: {
      type: String,
      noGroup: "0"
    },
    buttonType:{
      type: Boolean,
      default:false
    }
  },
  // computed:{
  // 	getSelectUsers(){
  // 		var showValue = "";
  // 		for(var i=0;i<this.inputValue.length;i++){
  // 			if(showValue.length>0){
  // 				showValue+=";"
  // 			}
  // 			showValue+=this.inputValue[i];
  // 		}
  // 		return showValue;
  // 	}
  // },
  methods: {
  
    clickShowDialog() {
      this.visible = true;
    },
    refreshData() {
      let _self = this;
      for (var i = 0; i < _self.rightList.length; i++) {
        _self.rightListId[i] = _self.rightList[i].id;
      }
      var m = new Map();
      m.set("noGroup", _self.noGroup);
      m.set("condition", "name like '%" + this.findValue + "%' or login_name like '%" + this.findValue + "%'");
      m.set("pageIndex", 0);
      m.set("pageSize", 50);
      m.set("roleName", _self.roleName);
      axios.post("/admin/getUsersByGroupName",m)
        .then(function(response) {
          _self.dataList = response.data.data;
          for (var i = 0; i < _self.rightListId.length; i++) {
            var item = _self.rightListId[i];
            _self.dataList.forEach(function(val, index, arr) {
              if (val.id == item) {
                arr.splice(index, 1);
              }
            });
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    addToFather() {
      let _self = this;
      _self.rightList.forEach(function(val, index, arr) {
        _self.rightNameList += val.name + ";";
      });
      _self.rightNameList = _self.rightNameList.substring(
        0,
        _self.rightNameList.length - 1
      );
      _self.$emit("change", _self.rightNameList);
      _self.rightNameList = "";
      _self.visible = false;
    },
    handleSelectionChange(selection) {
      this.tranList = [];
      if (selection.length > 0) {
        for (var i = 0; i < selection.length; i++) {
          this.tranList.push(selection[i]);
        }
      }
    },
    handleRightSelectionChange(selection) {
      this.tranList2 = [];
      if (selection.length > 0) {
        for (var i = 0; i < selection.length; i++) {
          this.tranList2.push(selection[i]);
        }
      }
    },
    leftClick(row){
      this.$refs.leftTable.toggleRowSelection(row);
    },
    rightClick(row){
      this.$refs.rightTable.toggleRowSelection(row);
    },
    leftDbClick(row){
      this.tranList = [];
      this.tranList.push(row);
      this.addToRight();
    },
    rightDbClick(row){
      this.tranList2 = [];
      this.tranList2.push(row);
      this.addToLeft();
      this.refreshData();
    },
    addToRight() {
      if(!this.isRepeat && this.rightList.length>0){
        return;
      }
      for (var i = 0; i < this.tranList.length; i++) {
        this.rightList.push(this.tranList[i]);
        if(!this.isRepeat){
          break;
        }
      }
      for (var i = 0; i < this.tranList.length; i++) {
        var item = this.tranList[i];
        this.dataList.forEach(function(val, index, arr) {
          if (item.id == val.id) {
            arr.splice(index, 1);
          }
        });
         if(!this.isRepeat){
          break;
        }
      }
    },
    addToLeft() {
      
      for (var i = 0; i < this.tranList2.length; i++) {
        this.dataList.push(this.tranList2[i]);
      }
      for (var i = 0; i < this.tranList2.length; i++) {
        var item = this.tranList2[i];
        this.rightList.forEach(function(val, index, arr) {
          if (item.id == val.id) {
            arr.splice(index, 1);
          }
        });
      }
    },
    closeDialog() {
      this.visible = false;
      this.findValue = "";
      this.rightList = [];
    }
  }
};
</script>
<style scoped>
.el-header {
  background-color: white;
  text-align: left;
}
.el-main {
  background-color: white;
}
.el-footer {
  background-color: white;
}
</style>
