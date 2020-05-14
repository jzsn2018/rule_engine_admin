<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('rules:entityinfo:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <!-- <el-button v-if="isAuth('rules:entityinfo:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading"  style="width: 100%;">
    <!--  <el-table-column type="selection" header-align="center" align="center" width="50">
      </el-table-column> -->
	  <el-table-column type="expand">
		  <template slot-scope="props">
		          <el-form label-position="left"  class="table-expand">
		            <el-form-item label="实体名称">
		              <span>{{ props.row.entityName }}</span>
		            </el-form-item>
		            <el-form-item label="所属场景">
		              <span>{{ props.row.scene }}</span>
		            </el-form-item>
		            <el-form-item label="包路径">
		              <span>{{ props.row.pkgName }}</span>
		            </el-form-item>
		            <el-form-item label="标识">
		              <span>{{ props.row.entityIdentify }}</span>
		            </el-form-item>
		            <el-form-item label="描述">
		              <span>{{ props.row.entityDesc }}</span>
		            </el-form-item>
		            <el-form-item label="关联字段">
                  <el-tooltip :key="index"  v-for="(item,index) in props.row.item" class="item" effect="dark"  placement="bottom">
                  <div slot="content"> 实体ID:&nbsp;{{item.id}} <br/> 实体描述:&nbsp;{{ item.itemName }} </div>
		              <span >{{ item.itemIdentify }}&nbsp;</span>
                  </el-tooltip>
		            </el-form-item>
                <el-form-item label="备注">
                  <span>{{ props.row.remark }}</span>
                </el-form-item>
		          </el-form>
		        </template>
	  </el-table-column>
      <el-table-column prop="id" header-align="center" align="center" label="ID" width="50">
      </el-table-column>
      <el-table-column prop="entityName" header-align="center" align="center" label="名称">
      </el-table-column>
      <el-table-column prop="scene" header-align="center" align="center" label="所属场景">
      </el-table-column>
      <el-table-column prop="entityDesc" header-align="center" align="center" label="描述">
      </el-table-column>
      <el-table-column prop="entityIdentify" header-align="center" align="center" label="标识">
      </el-table-column>
      <!-- <el-table-column prop="pkgName" header-align="center" align="center" label="包路径">
      </el-table-column> -->
      <el-table-column prop="isEffect" header-align="center" align="center" label="是否有效">
        <template slot-scope="scope">
          <span>{{ displayName(isEffectList,scope.row.isEffect) }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="remark" header-align="center" align="center" label="备注">
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
  import AddOrUpdate from './entityinfo-add-or-update'
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
        addOrUpdateVisible: false
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
          url: this.$http.adornUrl('/rules/entityinfo/list'),
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
          if (parseInt(dataList[i].code) == dictValue) {
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
            this.actionTypeList = data.dict.actionType
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
      disableHandle(id) {
        console.log(id, 'id')
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.entityId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/rules/entityinfo/delete'),
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
  .box {
    width: 400px;

    .top {
      text-align: center;
    }

    .left {
      float: left;
      width: 60px;
    }

    .right {
      float: right;
      width: 60px;
    }

    .bottom {
      clear: both;
      text-align: center;
    }

    .item {
      margin: 4px;
    }

    .left .el-tooltip__popper,
    .right .el-tooltip__popper {
      padding: 8px 10px;
    }
  }
</style>
