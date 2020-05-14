<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('rules:actioninfo:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
       <!-- <el-button v-if="isAuth('rules:actioninfo:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle" style="width: 100%;">
      <el-table-column type="expand">
        <template slot-scope="props">
                <el-form label-position="left"  class="table-expand">
                  <el-form-item label="动作名称">
                    <span>{{ props.row.actionName }}</span>
                  </el-form-item>
                  <el-form-item label="动作类型">
                    <span>{{ displayName(actionTypeList,props.row.actionType) }}</span>
                  </el-form-item>
                  <el-form-item label="动作实现类">
                    <span>{{ props.row.actionClass }}</span>
                  </el-form-item>
                  <el-form-item label="动作描述">
                    <span>{{ props.row.actionDesc }}</span>
                  </el-form-item>
                  <el-form-item label="动作参数">
                    <el-tooltip v-for="(item,index) in props.row.actionParamItem" class="item" effect="dark"  placement="bottom">
                      <div slot="content"> 参数名称:&nbsp;{{item.actionParamName}} <br/> 参数值:&nbsp;{{ item.paramValueStr }} </div>
                    <span >{{ item.paramIdentify }}&nbsp;</span>
                    </el-tooltip>
                  </el-form-item>
                  <el-form-item label="备注">
                    <span>{{ props.row.remark }}</span>
                  </el-form-item>
                </el-form>
              </template>
      </el-table-column>

      <el-table-column prop="actionId" header-align="center" align="center" width="50" label="ID">
      </el-table-column>
      <el-table-column prop="actionName" header-align="center" align="center" label="动作名称">
      </el-table-column>
      <el-table-column prop="actionType" header-align="center" align="center" label="动作类型">
        <template slot-scope="scope">
          <span>{{ displayName(actionTypeList,scope.row.actionType) }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="actionClass" header-align="center" align="center" label="动作实现类">
      </el-table-column>
      <el-table-column prop="actionDesc" header-align="center" align="center" label="动作描述">
      </el-table-column>
      <el-table-column prop="isEffect" header-align="center" align="center" label="是否有效">
        <template slot-scope="scope">
          <span>{{ displayName(isEffectList,scope.row.isEffect) }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="remark" header-align="center" align="center" label="备注">
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.actionId)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.actionId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage" layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './actioninfo-add-or-update'

  export default {
    data() {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        isEffectList: [],
        actionTypeList: [],
        pageIndex: 1,
        pageSize: 20,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    filters: {
      formatTypeList: function(value) {
        return value
      }
    },
    mounted() {
      (async () => {
        await this.getisEffectListDict()
        this.getDataList()
      })()
    },
    components: {
      AddOrUpdate
    },
    methods: {
      // 获取数据列表
      getDataList() {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/rules/actioninfo/list'),
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
      getDict(dictName) {
        if (this.actionTypeList.length > 0) {
          return this.actionTypeList
        }
        this.$http({
          url: this.$http.adornUrl('/sys/dict/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': 0,
            'limit': 9999,
            'key': dictName
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            // dicts = data.page.list
            this.actionTypeList = data.page.list
          }
        })
      },
      getisEffectListDict() {
        if (this.isEffectList.length > 0) {
          return this.isEffectList
        }
        this.$http({
          url: this.$http.adornUrl('/sys/dict/qyType'),
          method: 'get',
          params: this.$http.adornParams({
            'page': 0,
            'limit': 9999,
            'key': 'actionType,isEffect'
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            // dicts = data.page.list
            this.isEffectList = data.dict.isEffect
            this.actionTypeList = data.dict.actionType
          }
        })
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.actionId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/rules/actioninfo/delete'),
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
<style>
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
  }
</style>
