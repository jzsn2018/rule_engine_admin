<template>
  <el-dialog :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">

      <el-form-item label="动作类型" prop="actionType">
        <el-select v-model="dataForm.actionType" placeholder="动作类型">
          <el-option v-for="item in actionTypeList" :key="item.value" :label="item.value" :value="item.code">
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="动作名称" prop="actionName">
        <el-input v-model="dataForm.actionName" placeholder="动作名称"></el-input>
      </el-form-item>
      <el-form-item label="实现类" prop="actionClass">
        <el-input v-model="dataForm.actionClass" placeholder="动作实现类"></el-input>
      </el-form-item>
      <el-form-item label="动作描述" prop="actionDesc">
        <el-input v-model="dataForm.actionDesc" placeholder="动作描述"></el-input>
      </el-form-item>
     <!-- <el-form-item label="是否有效" prop="isEffect">
        <el-select v-model="dataForm.isEffect" placeholder="是否有效">
          <el-option v-for="item in isEffectList" :key="item.value" :label="item.value" :value="item.code">
          </el-option>
        </el-select>
      </el-form-item> -->

      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
      </el-form-item>
      <!-- 参数-->
      <el-form-item label="参数" class="flex-100" prop="actionParamItem">
              <el-button type="primary" @click="addParamList">添加参数</el-button>
              <el-row v-for="(item,index) in paramList" :key="index" :gutter="20">
                <el-col :span="14">
                  <el-input placeholder="请输入内容" v-model="item.actionParamName">
                    <template slot="prepend">参数名</template>
                  </el-input>
                  <el-input placeholder="请输入内容" v-model="item.paramIdentify">
                    <template slot="prepend">标识</template>
                  </el-input>
                  <el-input placeholder="请输入内容" v-model="item.actionParamDesc">
                    <template slot="prepend">描述</template>
                  </el-input>
                  <el-input placeholder="请输入内容" v-model="item.remark">
                    <template slot="prepend">备注</template>
                  </el-input>
                  <el-button type="primary" @click="addParamValueList(index)">添加参数值</el-button>
                  <el-row  v-for="(item_val,index_val) in item.actionParamValueItem" :key="index_val" :gutter="20">
                    <el-col :span="14">
                      <el-input placeholder="请输入内容" v-model="item_val.paramValue">
                        <template slot="prepend">参数值</template>
                      </el-input>
                      <el-input placeholder="请输入内容" v-model="item_val.remark">
                        <template slot="prepend">备注</template>
                      </el-input>
                    </el-col>
                    <el-col :span="4">
                      <el-button type="primary" @click="delParamValueList(index,index_val)">删除</el-button>
                    </el-col>
                  </el-row>
                </el-col>
                <el-col :span="4">
                  <el-button type="primary" @click="delParamList(index)">删除</el-button>
                </el-col>
              </el-row>
            </el-form-item>



    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data() {
      return {
        visible: false,
        isEffectList: [],
        actionTypeList: [],
        paramList: [], // 参数
        //paramValueList: [], // 参数值
        dataForm: {
          actionId: 0,
          actionType: '',
          actionName: '',
          actionDesc: '',
          actionClass: '',
          isEffect: '',
          creUserId: '',
          creTime: '',
          remark: ''
        },
        dataRule: {
          actionType: [{
            required: true,
            message: '动作类型(1实现2自身)不能为空',
            trigger: 'blur'
          }],
          actionName: [{
            required: true,
            message: '动作名称不能为空',
            trigger: 'blur'
          }],
          actionDesc: [{
            required: true,
            message: '动作描述不能为空',
            trigger: 'blur'
          }],
          actionClass: [{
            required: true,
            message: '动作实现类(包路径)不能为空',
            trigger: 'blur'
          }],
          isEffect: [{
            required: true,
            message: '是否有效不能为空',
            trigger: 'blur'
          }],
          creUserId: [{
            required: true,
            message: '创建人不能为空',
            trigger: 'blur'
          }],
          creTime: [{
            required: true,
            message: '创建时间不能为空',
            trigger: 'blur'
          }],
          remark: [{
            required: true,
            message: '备注不能为空',
            trigger: 'blur'
          }]
        }
      }
    },
    mounted() {
      (async () => {
        await this.getisEffectListDict()
      })()
    },
    methods: {
      init(id) {
        this.dataForm.actionId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.actionId) {
            this.$http({
              url: this.$http.adornUrl(`/rules/actioninfo/info/${this.dataForm.actionId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({
              data
            }) => {
              if (data && data.code === 0) {
                this.dataForm.actionType = data.actionInfo.actionType.toString()
                this.dataForm.actionName = data.actionInfo.actionName
                this.dataForm.actionDesc = data.actionInfo.actionDesc
                this.dataForm.actionClass = data.actionInfo.actionClass
                this.dataForm.isEffect = data.actionInfo.isEffect.toString()
                this.dataForm.creUserId = data.actionInfo.creUserId
                this.dataForm.creTime = data.actionInfo.creTime
                this.dataForm.remark = data.actionInfo.remark
                this.paramList = data.actionInfo.actionParamItem
              }
            })
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
      addParamList() {
        let newTeam = {
          actionParamName: '',　//名称
          actionParamDesc: '',　//描述
          paramIdentify:　'',　//标识
          actionParamValueItem: [],
          remark: ''
        }
        this.paramList.push(newTeam)
      },
      delParamList(index) {
        this.paramList.splice(index, 1)
      },
      addParamValueList(index){
        let newTeam = {
          paramValue: '',　//参数值
          remark: ''
        }
        this.paramList[index].actionParamValueItem.push(newTeam)
      },
      delParamValueList(index,index_val) {
        this.paramList[index].actionParamValueItem.splice(index_val, 1)
      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/rules/actioninfo/${!this.dataForm.actionId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'actionId': this.dataForm.actionId || undefined,
                'actionType': this.dataForm.actionType,
                'actionName': this.dataForm.actionName,
                'actionDesc': this.dataForm.actionDesc,
                'actionClass': this.dataForm.actionClass,
                'isEffect': this.dataForm.isEffect,
                'creUserId': this.dataForm.creUserId,
                'creTime': this.dataForm.creTime,
                'actionParamItem': this.paramList,
                'remark': this.dataForm.remark
              })
            }).then(({
              data
            }) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
