<template>
    <div>
        <el-dialog :title="$t('application.Import')" :append-to-body="true" :visible.sync="importdialogVisible" width="70%">
            <el-row>
                <el-col :span="20">
                    <el-progress :text-inside="true" :stroke-width="24" :percentage="progressNum" status="success"></el-progress>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="20">
                    <el-form size="mini" :label-width="formLabelWidth" v-loading='uploading'>
                        <div style="height:200px;overflow-y:scroll; overflow-x:scroll;">
                        <el-upload
                            :limit="100"
                            :file-list="fileList"
                            action
                            :on-change="handleChange"
                            :auto-upload="false"
                            :multiple="false"
                        >
                            <el-button slot="trigger" size="small" type="primary">{{$t('application.selectFile')}}</el-button>
                        </el-upload>
                        </div>
                    </el-form>
                </el-col>
            </el-row>
            
            <div slot="footer" class="dialog-footer">
                <el-button @click="importdialogVisible = false">{{$t('application.cancel')}}</el-button>
                <el-button type="primary" @click="uploadData()">{{$t('application.start')+$t('application.Import')}}</el-button>
            </div>
        </el-dialog>
        <el-button
            type="primary"
            @click="beforeUploadFile()"
          ><slot>挂载文件</slot></el-button>
    </div>
</template>
<script>
export default {
    data(){
        return{
            importdialogVisible:false,
            formLabelWidth: "120px",
            uploadUrl:'/exc/packUploadZip',
            fileList:[],
            progressNum:0
        }
    },
    methods:{
        handleChange(file, fileList) {
            this.fileList = fileList;
        },
        beforeUploadFile(){
            let _self=this;
            _self.fileList=[];
            _self.importdialogVisible=true;
        
        },
        getFormData() {
            let _self = this;
            let formdata = new FormData();
            var data = {};
            _self.fileList.forEach(function(file) {
                //console.log(file.name);
                formdata.append("uploadFile", file.raw, file.name);
            });
            return formdata;
        },
        //上传文件
        uploadData() {
            let _self = this;
            let formdata = _self.getFormData();

            // _self.uploading=true;
            _self
                .axios({
                headers: {
                    "Content-Type": "application/json;charset=UTF-8"
                },
                 onUploadProgress: progressEvent => {
                    _self.progressNum=(progressEvent.loaded / progressEvent.total * 100).toFixed(0) //调用onProgress方法来显示进度条，需要传递个对象 percent为进度值
                    
                },
                datatype: "json",
                method: "post",
                data: formdata,
                url: _self.uploadUrl
                })
                .then(function(response) {
                _self.importdialogVisible = false;

                let mess = response.data;
                _self.$alert(mess, { dangerouslyUseHTMLString: true });
                    _self.$emit('refresh');
                })
                .catch(function(error) {
                _self.uploading=false;
                console.log(error);
                });
        },
    },
    props:{
        selectedItem:{ type: Array, default: null }
    }
}
</script>
<style scoped>

</style>