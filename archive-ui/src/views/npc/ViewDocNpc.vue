<template>
  <DataLayout>
    <template v-slot:header>
      <!-- 创建设计文件附件 -->
            <el-dialog :title="$t('application.Import')" :visible.sync="importSubVisible" width="70%" 
            :close-on-click-modal="false" :append-to-body="true">
                <el-form size="mini" :label-width="formLabelWidth" v-loading='uploading'>
                    <div style="height:200px;overflow-y:scroll; overflow-x:scroll;">
                    <el-upload
                        :limit="100"
                        :file-list="fileAttachList"
                        action
                        :on-change="handleChangeAttach"
                        :auto-upload="false"
                        :multiple="true"
                    >
                        <el-button slot="trigger" size="small" type="primary">{{$t('application.selectFile')}}</el-button>
                    </el-upload>
                    </div>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="importSubVisible = false">{{$t('application.cancel')}}</el-button>
                    <el-button type="primary" @click="uploadDataSub()">{{$t('application.start')+$t('application.Import')}}</el-button>
                </div>
            </el-dialog>
               <!-- 添加格式副本 -->
            <el-dialog :title="$t('application.Import')" :visible.sync="importdialogVisible" width="70%"
            :close-on-click-modal="false" :append-to-body="true">
                <el-form size="mini" :label-width="formLabelWidth" v-loading='uploading'>
                  <div style="height:200px;overflow-y:scroll; overflow-x:scroll;">
                    <el-upload
                      :limit="100"
                      :file-list="fileListRedition"
                      action
                      :on-change="handleChangeRedition"
                      :auto-upload="false"
                      :multiple="false"
                    >
                      <el-button slot="trigger" size="small" type="primary">{{$t('application.selectFile')}}</el-button>
                    </el-upload>
                  </div>
                </el-form>
                <div slot="footer" class="dialog-footer">
                  <el-button @click="importdialogVisible = false">{{$t('application.cancel')}}</el-button>
                  <el-button type="primary" @click="uploadDataForRedition()">{{$t('application.start')+$t('application.Import')}}</el-button>
                </div>
            </el-dialog>
    </template>
    <template v-slot:main="{ layout }">
      <!-- <el-card shadow="hover">
        <el-collapse value="模板" accordion>
          <el-collapse-item
            title="模板"
            name="模板"
            id="selectTemplate"
            key="cindex"
          >
            <el-row>
              <el-col :span="3" class="el-form-item__content">模板1</el-col>
              <el-col :span="6" class="el-form-item__content">
                <el-button type="primary">预览</el-button>
                <el-button type="primary">下载</el-button>
              </el-col>
            </el-row>
          </el-collapse-item>
        </el-collapse>
      </el-card> -->
      <el-card shadow="hover">
      <el-row>
        <el-col :span="24">
          <ShowProperty
            v-if="allowEdit"
            ref="ShowProperty"
            @onSaved="onSaved"
            @onSaveSuccess="onPropertiesSaveSuccess"
            width="95%"
            typeName="图纸文件审批单"
            :showUploadFile="false"
            v-bind:itemId="formId"
          ></ShowProperty>
          <ShowPropertyReadOnly
          v-else
          ref="ShowProperty"
          @onSaved="onSaved"
          @onSaveSuccess="onPropertiesSaveSuccess"
          width="95%"
          typeName="图纸文件审批单"
          v-bind:itemId="formId"
        ></ShowPropertyReadOnly>
        </el-col>
      </el-row>
      </el-card>
      <el-card shadow="hover">
      <el-row>
        <el-collapse value="文件列表">
          <el-collapse-item
            title="文件列表"
            name="文件列表"
            id="uploadFile"
            key="cindex"
          >
          <el-row >
            <el-col :span="2">
              <el-button  type="success" @click="showItemContent(formId)">查看主件</el-button>
            </el-col>
            <!---->
             <el-col :span="2">
              <el-button  type="success" @click="showItemContentEdit(formId)">在线编辑</el-button>
            </el-col>
            <el-col :span="2">
                <el-button type="primary" @click="beforeUploadFileRedition('/dc/addRendition')" >格式副本</el-button> 
            </el-col>
            
            <el-col :span="2" v-if="allowEdit">
              <MountFile  
                  :selectedItem="[{'ID':formId}]"
                  >{{ $t("application.ReplaceDoc") }}</MountFile>
            </el-col>
            <el-col :span="2" v-if="allowEdit">
            <el-button type="primary"  
             @click="beforeUploadSubFile('/dc/addAttachment')">
             {{$t('application.Add')+$t('application.Attachment')}}
             </el-button>
            </el-col>
            <el-col :span="2" v-if="allowEdit">
              <el-button type="danger"   @click="onDeleleItem(selectedAttachment,[$refs.attachmentDoc])">{{$t('application.delete')}}</el-button>
            </el-col>
            
            
          </el-row>
          <el-row>
            <!--列表-->
              <DataGrid
                  ref="attachmentDoc"
                  key="attachmentDocKey"
                  dataUrl="/dc/getDocuByRelationParentId"
                  v-bind:tableHeight="(layout.height-startHeight)*(100-topPercent)/100-bottomHeight"
                  v-bind:isshowOption="true" v-bind:isshowSelection ="true"
                  gridViewName="AttachmentGrid"
                  condition=" and a.NAME='附件'"
                  :optionWidth = "2"
                  :isshowCustom="false"
                  :isEditProperty="allowEdit"
                  showOptions="查看内容"
                  :isShowChangeList="false"
                  :parentId="formId"
                  :isInitData="true"
                  @selectchange="attachmentDocSelect"
              ></DataGrid>
          </el-row>
          </el-collapse-item>
          <!-- <el-collapse-item
            title="启动流程"
            name="startupWorkflow"
            id="startupWorkflow"
            key="cindexstartupWorkflow"
          >
          </el-collapse-item> -->
          
        </el-collapse>
      </el-row>
      </el-card>
       <el-card shadow="hover">
        <el-collapse value="选择流程审批人员">
          <el-collapse-item
            title="选择流程审批人员"
            name="选择流程审批人员"
            id="selectApprover"
            key="cindex"
          >
       <el-row>
         <el-form ref="taskForm" :model="taskForm">
            <div v-for="(approver,index)  in approvalUserList" :key="'approver_'+index">
                <!-- <label>{{'approver_'+index}}</label> -->
                <el-form-item :label="approver.activityName.split(':')[0]"
                :rules="[{required:validateValue(approver.activityName),message:$t('application.requiredInput'),trigger:['blur', 'change']}]"
                :label-width="formLabelWidth" style="float:left"
                :prop="approver.performerPolicy">
                <UserSelectInput v-if="isTodoTask"
                    v-model="taskForm[approver.performerPolicy]"
                    v-bind:inputValue="taskForm[approver.performerPolicy]"
                    v-bind:roleName="approver.roleName"
                    v-bind:isRepeat="approver.isMulti"
                ></UserSelectInput>
                <div style="width:80px;" v-else>{{vdata[approver.performerPolicy]}}</div>
                <!-- :buttonType = "formEnableType != 'TodoTask'" -->
                </el-form-item>
            </div>
            </el-form>
        </el-row>
          </el-collapse-item>
        </el-collapse>
       </el-card>
        <el-row>
            
            <!-- <el-col :span="24">
                <el-button type="primary" @click="getData()" style="position:absolute;right:120px;margin:10px;">发起流程</el-button>
                <el-button style="position:absolute;right:50px;margin:10px;" >暂  存</el-button>
            </el-col> -->
        </el-row>
    </template>
  </DataLayout>
</template>
<script>
import DataLayout from "@/components/ecm-data-layout";
import ShowProperty from "@/components/ShowProperty";
import UserSelectInput from "@/components/controls/UserSelectInput";
import DataGrid from "@/components/DataGrid";
import MountFile from "@/components/MountFile.vue";
import ShowPropertyReadOnly from "@/components/ShowPropertyReadOnly.vue";
export default {
  name: "ViewDocNpc",
  data() {
    return {
      file: [],
      fileList: [],
      fileListRedition: [],
      uploadFileModel:"",
      fileAttachList:[],
      approvalUserList:[],
      taskForm:{},
      formLabelWidth: "120px",
      importSubVisible:false,
      importdialogVisible: false,
      uploadUrl:"",
      selectedAttachment:[],
      selectedFileItem:[],
      resultData:{},
      docObj:null,
       doc:{
        id:"",
        format:"",
      },
    };
  },
  model: {
    prop: "vdata",
    event: "change"
  },
  props:{
      formId: {type:String,default:""},
      vdata:{
        type: Object  ,
        default: function () {
            return {}
        }
      },
      allowEdit: { type: Boolean, default: false },
      isTodoTask:{type:Boolean,default:true},
      processDefinitionId: {
        type: String,
        default: ""
      },
      activityName: {
        type: String,
        default: ""
      },
      needAllUser:{
        type:Boolean,
        default:false
      }
  },
  mounted() {
      console.log(this.vdata);
      if(this.needAllUser){
        this.getApprovalAllUserList();
      }else{
        this.getApprovalUserList();
      }
      
      
  },
  components: {
    DataLayout: DataLayout,
    ShowProperty: ShowProperty,
    UserSelectInput:UserSelectInput,
    DataGrid:DataGrid,
    MountFile: MountFile,
    ShowPropertyReadOnly:ShowPropertyReadOnly
  },
  methods: {
    attachmentDocSelect(val){
      this.selectedAttachment=val;
    },
    validateValue(itemData){
      if(itemData.split(':')[1]=='true'){
        return true;
      }
      return false;
    },
    validateApprover1(){
      let result=false;
       this.$refs.taskForm.validate((valid) => {
         
          if (valid) {
            result=true;
            return true;
          } else {
            console.log('error submit!!');
            result=false
            return false;
          }
        });
        return result;
    },
    beforeUploadSubFile(uploadpath){
        let _self=this;
        if(_self.formId==undefined||_self.formId==""){
            // _self.$message('请选择一条文件数据');
            _self.$message({
                    showClose: true,
                    message:  _self.$t('message.pleaseSelectOneDesigndoc'),
                    duration: 2000,
                    type: "warning"
                });
            return;
        }
        _self.uploadUrl=uploadpath;
        _self.fileAttachList=[];
        _self.importSubVisible=true;
        
    },
    getSubFormData() {
        let _self = this;
        let formdata = new FormData();
        var data = {};
        data["parentDocId"] = _self.formId;//_self.selectedInnerItems[0].ID;//_self.selectedFileId;
        data["relationName"]='附件';
        data["TYPE_NAME"]='附件';
        formdata.append("metaData", JSON.stringify(data));
        _self.fileAttachList.forEach(function(file) {
            //console.log(file.name);
            formdata.append("uploadFile", file.raw, file.name);
        });
        return formdata;
    },
    //上传文件
        uploadDataSub() {
            let _self = this;
            let formdata = _self.getSubFormData();
            console.log("UploadData getData");
            console.log(formdata);
            _self.uploading=true;
            _self
                .axios({
                headers: {
                    "Content-Type": "application/json;charset=UTF-8"
                },
                datatype: "json",
                method: "post",
                data: formdata,
                url: _self.uploadUrl
                })
                .then(function(response) {
                _self.importSubVisible = false;
                // _self.refreshData();
                _self.uploading=false;
                // _self.$message(_self.$t('application.Import')+_self.$t('message.success'));
                _self.$message({
                        showClose: true,
                        message: _self.$t('application.Import')+_self.$t('message.success'),
                        duration: 2000,
                        type: 'success'
                    });
                    _self.$refs.attachmentDoc.loadGridData();
                })
                .catch(function(error) {
                _self.uploading=false;
                console.log(error);
                });
            },
      // 查看内容
    showItemContent(id) {
        if(!id){
            return;
        }
      let condition = id;
      let href = this.$router.resolve({
        path: "/viewdoc",
        query: {
          id: condition,
          //token: sessionStorage.getItem('access-token')
        },
      });
      //console.log(href);
      window.open(href.href, "_blank");
    },
      sendData() {
        let _self=this;
          let formData= _self.$refs.ShowProperty.getFormData();
          _self.$refs.taskForm.model=_self.taskForm;
          let jsonStr= formData.get('metaData');
          let m= JSON.parse(jsonStr);
           for(let key in _self.taskForm){
              let p=new Array();
              p.push(key);
              p.push(_self.taskForm[key]);
              m.push(p);
          }
          let x= _self.validateApprover1();
            if(!x){
              _self.$emit("click", null);
              // _self.resultData=null;
            }else{
              _self.$emit("click", new Map(m));
              // _self.resultData=new Map(m);
            }
          
         
      },
      
      saveData(isStartupWorkflow){
          let _self=this;
          let formData= _self.$refs.ShowProperty.getFormData();
          let jsonStr= formData.get('metaData');
          let m= JSON.parse(jsonStr);
           for(let key in _self.taskForm){
              let p=new Array();
              p.push(key);
              p.push(_self.taskForm[key]);
              m.push(p);
          }
          
          _self.$emit("change",new Map(m));
          // m.push(['ID',_self.selectedItemId]);
        axios.post("/dc/saveDocument",JSON.stringify(m))
        .then(function(response) {
          let code = response.data.code;
          //console.log(JSON.stringify(response));
          if(code==1){
            _self.$emit('onSaved','update');
            _self.$emit("onSaveSuccess",m);
            if(isStartupWorkflow){
              _self.$emit("onStartUpworkflow",m);
            }
          }
          else{
             _self.$message(_self.$t('message.saveFailured'));
          }
        })
        .catch(function(error) {
          _self.$message(_self.$t('message.saveFailured'));
          console.log(error);
        });
        
      },
    
// 在线编辑
    showItemContentEdit(id) {
        if(!id){
            return;
        }
      let condition = id;
      // let href = this.$router.resolve({
      //   path: "/viewdocEdit",
      //   query: {
      //     id: condition,
      //     //token: sessionStorage.getItem('access-token')
      //   },
      // });
      // //console.log(href);
      // window.open(href.href, "_blank");
    let _self = this;
    this.token = sessionStorage.getItem("access-token");
    axios.post("/dc/getDocument",id).then(function(response) {
        _self.docObj=response.data.data;
        _self.doc.permit = response.data.permit;
        _self.doc.id=_self.docObj.ID;
        _self.doc.format=_self.docObj.FORMAT_NAME;
        let getfileUrl =  _self.axios.defaults.baseURL+"/dc/getContent?id="+_self.doc.id+"&token="+sessionStorage.getItem('access-token')+"&format="+_self.doc.format;
     
        
            axios.post("/dc/checkOut",id).then(function(response) {
                if (response.data.code == 1) {
                  _self.approvalUserList = response.data.data;
                    var urlStr = "./static/dsoframeviewerh5/dsoframe.html?file="+encodeURIComponent(getfileUrl);
                    var win = window.open("","_blank");
                    win.open(urlStr,"_self");
                } else {
                  _self.$message({
                    showClose: true,
                    message: response.data.message,
                    duration: 2000,
                    type: "warning"
                  });
                  return;
                }
              });

        }).catch(function(error) {
          console.log(error);
      });


    },

    handleChangeAttach(file, fileList) {
      this.fileAttachList = fileList;
      //console.log(file);
      // console.log(fileList);
    },
    getApprovalAllUserList() {
      let _self = this;
      var m = new Map();
      m.set("processDefinitionName", "图纸文件审批流程");
      axios
        .post("/workflow/getApprovalAllUserList", JSON.stringify(m))
        .then(function(response) {
          if (response.data.code == 1) {
            _self.approvalUserList = response.data.data;
          } else {
            _self.$message({
              showClose: true,
              message: response.data.message,
              duration: 2000,
              type: "warning"
            });
            return;
          }
        });
    },
    getApprovalUserList(){
      let _self = this;
      var m = new Map();
      m.set("processDefinitionId", _self.processDefinitionId);
      m.set("activityName", _self.activityName);
      ///workflow/getApprovalUserList
      axios
        .post("/workflow/getApprovalUserListVisible", JSON.stringify(m))
        .then(function(response) {
         _self.approvalUserList = response.data.data;
         let v={};
        _self.approvalUserList.forEach((e)=>{
          v[e.performerPolicy]=_self.vdata[e.performerPolicy];
        });
        let arr=JSON.parse(JSON.stringify(v));
        _self.taskForm=arr;
        });
     
    },

     beforeUploadSubFile(uploadpath){
        let _self=this;
        if(_self.formId==undefined||_self.formId==""){
            // _self.$message('请选择一条文件数据');
            _self.$message({
                    showClose: true,
                    message:  _self.$t('message.pleaseSelectOneDesigndoc'),
                    duration: 2000,
                    type: "warning"
                });
            return;
        }
        _self.uploadUrl=uploadpath;
        _self.fileAttachList=[];
        _self.importSubVisible=true;
        
    },
    //上传格式副本窗体
    beforeUploadFileRedition(uploadpath){
      let _self=this;
        if(_self.formId==undefined||_self.formId==""){
            // _self.$message('请选择一条文件数据');
            _self.$message({
                    showClose: true,
                    message:  _self.$t('message.pleaseSelectOneDesigndoc'),
                    duration: 2000,
                    type: "warning"
                });
            return;
        }
      _self.uploadUrl=uploadpath;
      _self.fileListRedition=[];
      _self.importdialogVisible=true;
      
    },

handleChangeRedition(file, fileList) {
      this.fileListRedition = fileList;
      //console.log(file);
      // console.log(fileList);
    },
      getFormDataForRedition() {
        let _self = this;
        let formdata = new FormData();
        var data = {};
        data["ID"] = _self.formId;//_self.selectedInnerItems[0].ID;//_self.selectedFileId;
        formdata.append("metaData", JSON.stringify(data));
        debugger
        _self.fileListRedition.forEach(function(file) {
            //console.log(file.name);
            formdata.append("uploadFile", file.raw, file.name);
        });
        return formdata;
    },
     //上传文件
    uploadDataForRedition() {
      let _self = this;
      debugger
      let formdata = _self.getFormDataForRedition();;
      console.log("UploadData getData");
      console.log(formdata);
      _self.uploading=true;
      _self
        .axios({
          headers: {
            "Content-Type": "application/json;charset=UTF-8"
          },
          datatype: "json",
          method: "post",
          data: formdata,
          url: _self.uploadUrl
        })
        .then(function(response) {
          _self.importdialogVisible = false;
          // _self.refreshData();
          _self.uploading=false;
          // _self.$message(_self.$t('application.Import')+_self.$t('message.success'));
          _self.$message({
                showClose: true,
                message: _self.$t('application.Import')+_self.$t('message.success'),
                duration: 2000,
                type: 'success'
              });
        })
        .catch(function(error) {
          _self.uploading=false;
          console.log(error);
        });
    },
  }
};
</script>
<style scoped>
</style>
