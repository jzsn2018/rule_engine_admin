<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('rules:sceneinfo:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <!-- <el-button v-if="isAuth('rules:sceneinfo:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle" style="width: 100%;">
      <!-- <el-table-column type="selection" header-align="center" align="center" width="50">
      </el-table-column> -->
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" class="table-expand">
            <el-form-item label="场景名称">
              <span>{{ props.row.sceneName }}</span>
            </el-form-item>
            <el-form-item label="标识">
              <span>{{ props.row.sceneIdentify }}</span>
            </el-form-item>
            <el-form-item label="描述">
              <span>{{ props.row.sceneDesc }}</span>
            </el-form-item>
            <el-form-item label="关联实体">
              <span>{{ props.row.entityItem }}</span>
            </el-form-item>
            <el-form-item label="备注">
              <span>{{ props.row.remark }}</span>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column prop="sceneId" header-align="center" align="center" label="ID" width="50">
      </el-table-column>
      <el-table-column prop="sceneName" header-align="center" align="center" label="名称">
      </el-table-column>
      <el-table-column prop="sceneDesc" header-align="center" align="center" label="描述">
      </el-table-column>
      <!-- <el-table-column prop="sceneType" header-align="center" align="center" label="类型">
      </el-table-column> -->
      <el-table-column prop="sceneIdentify" header-align="center" align="center" label="标识">
      </el-table-column>
      <el-table-column prop="isEffect" header-align="center" align="center" label="是否有效">
        <template slot-scope="scope">
          <span>{{ displayName(isEffectList,scope.row.isEffect) }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="remark" header-align="center" align="center" label="备注">
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="200" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="ruleInitHandle(scope.row.sceneIdentify)">预览Drl文件</el-button>
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.sceneId)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.sceneId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage" layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <!-- 参数对话框-->
    <el-dialog title="请填写参数" class="dialog-expand" :visible.sync="dialogparamFormVisible">
      <el-form :model="paramForm" >
        <el-form-item :key="index" v-for="item in paramForm.conList" :label="item.fieldName" :label-width="formLabelWidth">
          <el-input :placeholder="item.field" v-model="item.fieldValue" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogparamFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="executeHandle()">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 结果预览对话框-->
    <el-dialog title="结果预览" class="dialog-expand" :visible.sync="dialogFormVisible">
      <el-form :model="form" class="table-expand">
        <el-form-item label="入参" :label-width="formLabelWidth">
          <span v-html="form.request"></span>
        </el-form-item>
        <el-form-item label="计算结果" :label-width="formLabelWidth">
          <span v-html="form.data"></span>
        </el-form-item>
        <el-form-item label="规则串" :label-width="formLabelWidth">
          <span v-html="form.ruleStr"></span>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
      </div>
    </el-dialog>


  </div>
</template>

<script>
  import AddOrUpdate from './sceneinfo-add-or-update'
  export default {
    data() {
      return {
        dataForm: {
          key: ''
        },
        isEffectList: [],
        dataList: [],
        pageIndex: 1,
        pageSize: 20,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        dialogFormVisible: false,
        dialogparamFormVisible: false,
        form: {
          ruleStr: '',
          request: '',
          data: '',
          delivery: false,
          type: [],
          resource: '',
          desc: ''
        },
        paramForm: {
          conList: [],
          scene: ''
        },
        formLabelWidth: '120px'
      }
    },
    components: {
      AddOrUpdate
    },
    mounted() {
      (async () => {
        await this.getisEffectListDict()
        this.getDataList()
      })()
    },
    methods: {
      // 获取数据列表
      getDataList() {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/rules/sceneinfo/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      displayName(dataList, dictValue) {
        let val = ''
        for (var i = 0; i < dataList.length; i++) {
          if (parseInt(dataList[i].code) === dictValue) {
            val = dataList[i].value
            break
          }
        }
        return val
      },
      getisEffectListDict() {
        if (this.isEffectList.length > 0) {
          return this.isEffectList
        }
        this.$http({
          url: this.$http.adornUrl('/sys/dict/qyType'),
          method: 'get',
          params: this.$http.adornParams({
            'key': 'actionType,isEffect'
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.isEffectList = data.dict.isEffect
            //  this.actionTypeList = data.dict.actionType
          }
        })
      },
      // 每页数
      sizeChangeHandle(val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle(val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle(val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle(id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      executeHandle(){
        this.dialogparamFormVisible = false;
         this.$http({
           url: this.$http.adornUrl('/drools/executeDrools'),
           method: 'post',
           data: this.$http.adornData({
             'params': this.paramForm
           })
         }).then(({
           data
         }) => {
           if (data && data.code === 0) {
             this.form.ruleStr = data.data.rulesStr.replace(/\r\n/g, "<br/>")
             this.form.data = data.data.globalMap
             this.form.request = data.data.factObjectList
             this.dialogFormVisible = true;
           } else {
             this.$message.error(data.msg)
           }
         })
      },
      ruleInitHandle(secen) {
        this.$http({
          url: this.$http.adornUrl('/drools/execRuleParam'),
          method: 'get',
          params: this.$http.adornParams({
            'scene': secen
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.paramForm.conList = data.con
            this.paramForm.scene = secen
            this.dialogparamFormVisible = true;
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.sceneId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/rules/sceneinfo/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({
            data
          }) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
<style scoped>
  .table-expand {
    font-size: 0;
  }

  .table-expand label {
    width: 90px;
    color: #99a9bf;
  }

  .table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
    white-space: nowrap;
  }

  .el-dialog__title {
    line-height: 24px;
    font-size: 18px;
    color: #008200;
  }
</style>
