     <template>
  <div>
    <el-dialog
      title="文档借阅"
      :visible.sync="borrowVisible"
      @close="borrowVisible = false"
      width="90%"
      style="width: 100%"
      :close-on-click-modal="false"
      v-dialogDrag
    >
      <div><BorrowStartUp :workflowObj="workflow" :showUploadFile="true" :workflowFileList="selectedItemList" @closedialog="closeDialog"></BorrowStartUp></div>
    </el-dialog>
    <el-dialog
      :title="$t('application.borrow')"
      :visible.sync="borrowDialogVisible"
      @close="propertyVisible = false"
      width="95%"
      style="width:100%"
      custom-class="customWidthFull"
    >
      <ShowBorrowForm ref="ShowBorrowForm" width="100%" v-bind:borrowForm="borrowForm"></ShowBorrowForm>
    </el-dialog>

    <div slot="footer" class="dialog-footer" style="padding-top:10px">
      <router-link ref="borrowRouteLink" to="/borroworder"></router-link>
      <!-- <el-button  v-if="formId!=''" @click="addToFormFromShopingCart()" style="float:left">添加到表单</el-button> -->
      <!-- <div v-if="formId==''"> -->
        <div v-if="showFooter == true">
        <el-button @click="cleanShopingCart()">清空</el-button>
        <el-button @click="removeShopingCart()">移除所选</el-button>
        <!-- <el-button @click="showDrawingItem()">调晒</el-button> -->
        <el-button @click="borrowItem()">借阅</el-button>
      </div>
    </div>
    <el-form :model="shopingCartForm" style="width:100%">
      <el-row style="width:100%">
        <div v-if="1==1">
          <el-col>
            <el-table :height="tableHeight" :data="tabledata" border v-loading="loading" @selection-change="selectChange">
              <el-table-column type="selection" width="40"></el-table-column>
              <el-table-column type="index" label="#" width="50"></el-table-column>
              <el-table-column prop="id" label="id" v-if="1==2" min-width="15%" sortable></el-table-column>
              <el-table-column width="40">
                <template slot-scope="scope">
                  <img
                    v-if="scope.row.TYPE_NAME=='图册'"
                    :src="'./static/img/drawing.gif'"
                    :title="scope.row.TYPE_NAME"
                    border="0"
                  />
                  <img
                    v-else-if="scope.row.C_ITEM_TYPE=='案卷'"
                    :src="'./static/img/box.gif'"
                    :title="scope.row.TYPE_NAME"
                    border="0"
                  />
                  <img
                    v-else
                    :src="'./static/img/format/f_'+scope.row.FORMAT_NAME+'_16.gif'"
                    :title="scope.row.FORMAT_NAME"
                    border="0"
                  />
                </template>
              </el-table-column>>
              <template v-for="(citem, idx) in gridList">
                <template v-if="citem.visibleType == 1">
              <template v-if="(citem.width + '').indexOf('%') > 0">
                <el-table-column
                  :label="citem.label"
                  :prop="citem.attrName"
                  :min-width="citem.width"
                  :sortable="citem.allowOrderby"
                  :key="idx + '_C'"
                >
                  <template slot-scope="scope">
                    <div v-if="citem.attrName.indexOf('DATE') > 0">
                      <span>{{ dateFormat(scope.row[citem.attrName]) }}</span>
                    </div>
                    <div v-else>
                      <span
                        :class="
                          scope.row['LIFECYCLE_DIR'] == 0 ? 'reject' : 'success'
                        "
                        >{{ scope.row[citem.attrName] }}</span
                      >
                    </div>
                  </template>
                </el-table-column>
              </template>
              <template v-else>
                <el-table-column
                  :label="citem.label"
                  :width="citem.width"
                  :prop="citem.attrName"
                  :sortable="citem.allowOrderby"
                  :key="idx + '_C'"
                >
                  <template slot-scope="scope">
                    <div v-if="citem.attrName.indexOf('DATE') > 0">
                      <span>{{ dateFormat(scope.row[citem.attrName]) }}</span>
                    </div>
                    <div v-else>
                      <span
                        :class="
                          scope.row['LIFECYCLE_DIR'] == 0 ? 'reject' : 'success'
                        "
                        >{{ scope.row[citem.attrName] }}</span
                      >
                    </div>
                  </template>
                </el-table-column>
              </template>
              </template>
              </template>
              <el-table-column align="right" width="80">
                <template slot-scope="scope">
                  <el-button size="mini" @click="viewdoc(scope.row)">查看</el-button>
                </template>
              </el-table-column>
            </el-table>
            <el-pagination
              background
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="currentPage"
              :page-sizes="[10, 20, 50, 100, 200]"
              :page-size="pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="itemCount"
            ></el-pagination>
          </el-col>
        </div>
      </el-row>
    </el-form>
  </div>
</template>


<script type="text/javascript">
import ShowBorrowForm from "@/components/form/Borrow";
import BorrowStartUp from "@/views/workflow/BorrowStartUp.vue"
export default {
  name: "Favorite",
  components: {
    ShowBorrowForm: ShowBorrowForm,
    BorrowStartUp: BorrowStartUp,
  },
  data() {
    return {
      gridviewName: "favoriteGrid",
      gridList: [],
      currentLanguage: "zh-cn",
      tabledata: [],
      loading: false,
      formLabelWidth: "100px",
      shopingCartData: [],
      selectedItemList: [],
      dialogTitle: "借阅",
      shopingCartDialogVisible: false,
      shopingCartForm: {
        taskId: 0,
        result: "在线浏览",
        message: "",
      },
      borrowData: [],
      borrowDialogVisible: false,
      componentName: "shopingCart",
      borrowForm: {
        taskId: 0,
        result: "在线浏览",
        message: "",
      },
      tableHeight: window.innerHeight - 124,

      workflow: {},
      borrowVisible: false,
      currentPage: 1,
      pageSize: 20,
    };
  },
  props: {
    formId: { type: String, default: "" },
    excludeRows: { type: Array, default: () => [] },
    showFooter: { type: Boolean, default: true }
  },
  mounted() {
    let _self = this;

    var psize = localStorage.getItem("docPageSize");
    if (psize) {
      _self.pageSize = parseInt(psize);
    }
  },
  created() {
    let _self = this;
    _self.loadGridView();
  },
  methods: {
    getWorkFlow() {
      let _self = this;

      
    },
    //获取
    openShopingCart() {
      let _self = this;
      var m = new Map();
      _self.loadingTodoData = true;
      m.set("condition", "TYPE_NAME='收藏夹'");
      m.set("pageSize", _self.pageSize);
      m.set("pageIndex", _self.currentPage - 1);
      m.set("userId", sessionStorage.getItem("access-userName"));
      //let i=0;
      axios
        .post("/dc/openShopingCart", JSON.stringify(m))
        .then(function (response) {
          _self.tabledata = response.data.data.filter(function (item) {
            let goodData = true;
            for (let i = 0; i < _self.excludeRows.length; i++) {
              if (item.ID == _self.excludeRows[i].ID) {
                goodData = false;
              }
            }
            return goodData;
          });
          _self.totalCount = response.data.totalCount;
          _self.itemCount = response.data.pager.total;
          _self.loadingTodoData = false;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    dateFormatter(row, column) {
      let datetime = row[column.property];
      return this.datetimeFormat(datetime);
    },
    cancel(b) {
      this.$emit("showOrHiden", b);
    },
    loadGridView() {
      let _self = this;
      var m = new Map();
      m.set("gridName", _self.gridviewName);
      m.set("lang", _self.currentLanguage);
      axios
        .post("/dc/getGridViewInfo", JSON.stringify(m))
        .then(function (response) {
          _self.gridList = response.data.data;
          _self.openShopingCart();
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    selectChange(selection) {
      this.selectedItemList = [];
      if (selection.length > 0) {
        for (var i = 0; i < selection.length; i++) {
          this.selectedItemList.push(selection[i]);
        }
      }
    },
    showDrawingItem() {
      let _self = this;
      let m = new Map();
      let C_ARCHIVE_UNIT = sessionStorage.getItem("access-department");
      var addItemId = [];
      if (_self.selectedItemList.length > 0) {
        for (var i = 0; i < _self.selectedItemList.length; i++) {
          addItemId.push(_self.selectedItemList[i].ID);
          if (
            typeof _self.selectedItemList[i].C_ARCHIVE_UNIT == "undefined" ||
            C_ARCHIVE_UNIT != _self.selectedItemList[i].C_ARCHIVE_UNIT ||
            _self.selectedItemList[i].C_SECURITY_LEVEL != "内部公开"
          ) {
            _self.$message({
              showClose: true,
              message: "只能晒本人所在部门且内部公开的图纸!",
              duration: 5000,
              type: "warning",
            });
            return;
          }
        }
        let showDrawingMap = new Map();
        showDrawingMap.set(
          "C_DRAFTER",
          sessionStorage.getItem("access-userName")
        );
        showDrawingMap.set(
          "C_DESC1",
          sessionStorage.getItem("access-department")
        );
        showDrawingMap.set("STATUS", "待晒图");
        // showDrawingMap.set(C_DRAFTER,sessionStorage.getItem("access-userName"));
        m.set("formData", showDrawingMap);
        m.set("documentIds", addItemId);
        m.set("formId", "");
        _self.loading = true;
        axios
          .post("/dc/SaveShowDrawing", JSON.stringify(m))
          .then(function (response) {
            _self.$message({
              showClose: true,
              message: self.$t('message.ShowDrawSuccess'),
              duration: 2000,
              type: "warning",
            });
            _self.loading = false;
          })
          .catch(function (error) {
            _self.$message({
              showClose: true,
              message: _self.$t('message.ShowDrawFailed'),
              duration: 2000,
              type: "warning",
            });

            _self.loading = false;
            console.log(error);
          });
      } else {
        _self.$message({
          showClose: true,
          message: 
_self.$t('message.PleaseSelectOneDraw'),
          duration: 5000,
          type: "warning",
        });
        return;
      }
    },
    //借阅
    borrowItem() {
      let _self = this;
      
      var m = new Map();
      m.set("processDefinitionKey", "文档借阅流程");

      axios
        .post("/dc/getWorkflow", JSON.stringify(m))
        .then(function (response) {
          _self.workflow = response.data.data[0];
          console.log(_self.workflow)
          _self.borrowVisible = true;
        })
        .catch(function (error) {
          console.log(error);
        });
    },

    closeDialog(val) {
      this.borrowVisible = val;
    },

    cleanShopingCart() {
      let _self = this;
      var m = new Map();
      axios
        .post("/dc/cleanShopingCart", JSON.stringify(m))
        .then(function (response) {
          if (response.data.code == 1) {
            m = new Map();
            axios
              .post("/dc/openShopingCart", JSON.stringify(m))
              .then(function (response) {
                _self.tabledata = response.data.data;

                //_self.tabledata=excludeRows;
                _self.totalCount = response.data.totalCount;
                _self.loadingTodoData = false;
              })
              .catch(function (error) {
                console.log(error);
              });

            _self.$message({
              showClose: true,
              message:_self.$t('message.Clear'),
              duration: 2000,
              type: "success",
            });
          }
          _self.totalCount = response.data.totalCount;
          _self.loadingTodoData = false;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    removeShopingCart() {
      let _self = this;
      var m = new Map();
      var addItemId = [];
      if (_self.selectedItemList.length > 0) {
        for (var i = 0; i < _self.selectedItemList.length; i++) {
          addItemId.push(_self.selectedItemList[i].ID);
        }
      }
      if (addItemId.length == 0) {
        _self.$message({
          showClose: true,
          message:_self.$t('message.PleaseSelectOneFileRemoved'),
          duration: 2000,
          type: "warning",
        });
        return;
      }
      axios
        .post("/dc/removeShopingCart", JSON.stringify(addItemId))
        .then(function (response) {
          if (response.data.code == 1) {
            m = new Map();
            axios
              .post("/dc/openShopingCart", JSON.stringify(m))
              .then(function (response) {
                _self.tabledata = response.data.data;
                _self.totalCount = response.data.totalCount;
                _self.loadingTodoData = false;
              })
              .catch(function (error) {
                console.log(error);
              });

            _self.$message({
              showClose: true,
              message:_self.$t('message.RemoveSuccess'),
              duration: 2000,
              type: "success",
            });
          }
          _self.totalCount = response.data.totalCount;
          _self.loadingTodoData = false;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    addToFormFromShopingCart() {
      let _self = this;
      var m = new Map();
      var addItemId = [];
      if (_self.selectedItemList.length > 0) {
        for (var i = 0; i < _self.selectedItemList.length; i++) {
          addItemId.push(_self.selectedItemList[i].ID);
        }
      } else {
        _self.$message({
          showClose: true,
          message: "请勾选添加的文件!",
          duration: 2000,
          type: "success",
        });
        return;
      }
      m.set("documentIds", addItemId);
      m.set("formId", _self.formId);
      axios
        .post("/dc/addItemToForm", JSON.stringify(m))
        .then(function (response) {
          _self.formId = response.data.data;
          _self.$message({
            showClose: true,
            message:_self.$t('message.AddSuccess'),
            duration: 2000,
            type: "success",
          });
          _self.loadGridView();
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    viewdoc(indata) {
      let condition = indata.ID;
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
    //分页 页数改变
    handleSizeChange(val) {
      let _self = this;
      this.pageSize = val;
      localStorage.setItem("docPageSize", val);
      _self.loadGridInfo(this.currentFolder);
      if (_self.showBox) {
        _self.loadAllGridData(this.currentFolder);
      } else {
        _self.loadGridData(this.currentFolder);
      }
    },
    // 分页 当前页改变
    handleCurrentChange(val) {
      let _self = this;
      this.currentPage = val;
      _self.loadGridInfo(this.currentFolder);
      if (_self.showBox) {
        _self.loadAllGridData(this.currentFolder);
      } else {
        _self.loadGridData(this.currentFolder);
      }
    },
  },
};
</script>