<template>
    <div>
      <el-dialog width="80%" title="列表" :visible.sync="archiveBackupVisible" @close="archiveBackupVisible = false">
        <el-row>
            <el-col>
                <DataGrid ref="orderGrid" key="main" v-bind:itemDataList="childDataList"
                      v-bind:columnList="childColumnList" @pagesizechange="childPageSizeChange"
                      @pagechange="childPageChange" v-bind:itemCount="childItemCount"
                      v-bind:tableHeight="rightTableHeight" :isshowOption="false"
                       ></DataGrid>
            </el-col>

        </el-row>
        <div slot="footer" class="dialog-footer">
          <!-- <el-button @click="saveItem">{{$t('application.save')}}</el-button>  -->
          <el-button @click="archiveBackupVisible = false">{{$t('application.cancel')}}</el-button>
        </div>
      </el-dialog>
        <el-row>
          <el-col :span="4" class="topbar-input">
          <el-input
            v-model="inputkey"
            :placeholder="$t('message.pleaseInput')+$t('application.keyword')"
            @change="searchItem"
            prefix-icon="el-icon-search"
          ></el-input>
          </el-col>
        </el-row>
        <el-row>
            <el-col>
                <DataGrid ref="orderGrid" key="main" 
                      v-bind:tableHeight="rightTableHeight" 
                      :isshowOption="true"
                      :loading="orderLoading"
                      :isshowicon="false"
                      :optionWidth = "1"
                      :isShowChangeList="false"
                      :isShowMoreOption="false"
                      condition="STATUS='已完成'"
                      gridViewName="ArchiveBackup"
                      dataUrl="/dc/getDocuments"
                     ></DataGrid>
            </el-col>

        </el-row>
        
    </div>
</template>
<script type="text/javascript">
import DataGrid from'@/components/DataGrid';
export default {
    name:'archiveBackupNew',
    permit:1,
    data(){
        return{
          archiveBackupVisible:false,
             gridList:[],
             itemDataList:[],
             itemDataListFull:[],
             inputkey:'',
             pageSize: 20,
             currentPage: 1,
             itemCount: 0,
             childCurrentPage:1,
             childPageSize: 20,
             childItemCount:0,
             childDataList:[],
             childColumnList:[],
             orderLoading:false,
             selectedRow:[],
            form: {
              coding:"",
              title:"",
              createDate:"",
              endDate:"",
              size:0,
              director:"",
              condition:""
            },
             rightTableHeight: window.innerHeight - 170
        }
       
    },
    mounted(){
        
        this.loadGridData();
    },
    components:{
        DataGrid:DataGrid
    },
    methods:{
     
        //查询文档
        searchItem() {
          this.loadGridData();
          // this.loadPageInfo();
        },
        // 加载借阅单表格数据
        loadGridData() {
          let _self = this;
          _self.orderLoading=true;
          var key0 = _self.inputkey;
          if (key0 != "") {
            key0 = " (coding like '%" + key0 + "%' or C_DRAFTER like '%" + key0 + "%') and STATUS='已完成' ";
          }else{
              key0=" STATUS='已完成' "
          }
          _self.$refs.orderGrid.condition = key0;
          _self.$refs.orderGrid.currentPage = 1;
          _self.$refs.orderGrid.loadGridInfo();
          _self.$refs.orderGrid.loadGridData();
        },
        
      }

};
</script>
<style scoped>

</style>