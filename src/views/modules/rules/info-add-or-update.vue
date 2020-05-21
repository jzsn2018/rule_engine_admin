<template>
  <el-dialog :title="!dataForm.ruleId ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">

      <el-form-item label="场景" prop="sceneId" >
        <el-select v-model="dataForm.sceneId" placeholder="请选择场景" style="width: 50%;">
          <el-option v-for="item in sceneDataList" :key="item.sceneId" :label="item.sceneName" :value="item.sceneId">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="动作" prop="actionRuleRelItem" style="width: 50%;">
        <el-select v-model="dataForm.actionRuleRelItem" placeholder="请选择动作" multiple collapse-tags>
          <el-option v-for="item in actionDataList" :key="item.id" :label="item.actName" :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="名称" prop="ruleName">
        <el-input v-model="dataForm.ruleName" placeholder="名称"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="ruleDesc">
        <el-input v-model="dataForm.ruleDesc" placeholder="描述"></el-input>
      </el-form-item>

      <!-- <el-form-item label="是否启用" prop="ruleEnabled">
      <el-select v-model="dataForm.ruleEnabled" placeholder="是否启用">
        <el-option v-for="item in ruleEnabledList" :key="item.value" :label="item.value" :value="item.code">
        </el-option>
      </el-select>

    </el-form-item>
    <el-form-item label="是否有效" prop="isEffect">
      <el-select v-model="dataForm.isEffect" placeholder="是否有效">
        <el-option v-for="item in isEffectList" :key="item.value" :label="item.value" :value="item.code">
        </el-option>
      </el-select>
    </el-form-item> -->
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
      </el-form-item>

      <el-form-item label="条件" class="flex-100" prop="item">
        <el-button type="primary" @click="addConditionInfoList">添加条件</el-button>
        <el-row v-for="(item,index) in conditionInfoList" :key="index" :gutter="20">
          <el-col :span="14">
            <el-input placeholder="请输入内容" v-model="item.conditionName">
              <template slot="prepend">条件名称</template>
            </el-input>
            <el-input placeholder="示例:$2$<10(说明:金额小于10;$实体属性ID$<10)" v-model="item.conditionExpression">
              <template slot="prepend">条件表达式</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.conditionDesc">
              <template slot="prepend">条件描述</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.remark">
              <template slot="prepend">备注</template>
            </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="delConditionInfoList(index)">删除</el-button>
          </el-col>
        </el-row>
      </el-form-item>

       <el-form-item label="属性" class="flex-100" prop="item">
        <el-button type="primary" @click="addPropertyList">添加属性</el-button>
        <el-row v-for="(item,index) in propertyList" :key="index" :gutter="20">
          <el-col :span="14">
            <!-- <el-input placeholder="请输入内容" v-model="item.rulePropertyName">
              <template slot="prepend">名称</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.rulePropertyIdentify">
              <template slot="prepend">标识</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.rulePropertyValue">
              <template slot="prepend">规则属性值</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.defaultValue">
              <template slot="prepend">默认值</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.rulePropertyDesc">
              <template slot="prepend">描述</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.remark">
              <template slot="prepend">备注</template>
            </el-input> -->
              <el-input placeholder="请输入属性值" v-model="item.rulePropertyValue" class="input-with-select">
                <el-select v-model="item.rulePropertyId" slot="prepend"
                 placeholder="请选择属性"
                  clearable
                  style="width: 200px;">
                  <el-option v-for="item_val in propertySelectList" :key="item_val.id" :label="item_val.identifyName" :value="item_val.id"></el-option>
                </el-select>
              </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="delPropertyList(index)">删除</el-button>
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
        sceneDataList: [],
        ruleEnabledList: [],
        conditionInfoList: [],
        actionDataList: [],
        propertyList: [],
        propertySelectList: [],
        dataForm: {
          ruleId: 0,
          sceneId: '',
          ruleName: '',
          ruleDesc: '',
          ruleEnabled: '',
          isEffect: '',
          actionRuleRelItem: [],
          remark: ''
        },
        dataRule: {
          sceneId: [{
            required: true,
            message: '场景不能为空',
            trigger: 'blur'
          }],
          ruleName: [{
            required: true,
            message: '名称不能为空',
            trigger: 'blur'
          }],
          ruleDesc: [{
            required: true,
            message: '描述不能为空',
            trigger: 'blur'
          }],
          ruleEnabled: [{
            required: true,
            message: '是否启用不能为空',
            trigger: 'blur'
          }],
          isEffect: [{
            required: true,
            message: '是否有效不能为空',
            trigger: 'blur'
          }],
          remark: [{
            required: false,
            message: '备注不能为空',
            trigger: 'blur'
          }]
        }
      }
    },
    mounted() {
      (async () => {
        this.getPropertySelectList()
        this.getisEffectListDict()
        this.getSceneDataList()
        this.getActionDataList()
      })()
    },
    methods: {
      init(id) {
        this.dataForm.ruleId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.ruleId) {
            this.$http({
              url: this.$http.adornUrl(`/rules/info/info/${this.dataForm.ruleId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({
              data
            }) => {
              if (data && data.code === 0) {
                this.dataForm.sceneId = data.info.sceneId
                this.dataForm.ruleName = data.info.ruleName
                this.dataForm.ruleDesc = data.info.ruleDesc
                this.dataForm.ruleEnabled = data.info.ruleEnabled.toString()
                this.dataForm.isEffect = data.info.isEffect.toString()
                this.dataForm.remark = data.info.remark
                this.dataForm.actionRuleRelItem = data.info.actionRuleRelItem
                this.conditionInfoList = data.info.conditionInfoItem
                this.propertyList = data.info.propertyInfoItem
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
            'key': 'ruleEnabled,isEffect'
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.isEffectList = data.dict.isEffect
            this.ruleEnabledList = data.dict.ruleEnabled
          }
        })
      },
      // 获取规则属性下来列表
      getPropertySelectList(){
        this.$http({
          url: this.$http.adornUrl('/rules/propertyinfo/select'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if(data && data.code === 0){
            this.propertySelectList=data.data
          }else{
            this.propertySelectList=[]
          }
        })
      },
      // 获取场景数据列表
      getSceneDataList() {
        this.$http({
          url: this.$http.adornUrl('/rules/sceneinfo/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': 1,
            'limit': 999
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.sceneDataList = data.page.list
          } else {
            this.sceneDataList = []
          }
        })
      },
      // 获取动作数据列表
      getActionDataList() {
        this.$http({
          url: this.$http.adornUrl('/rules/actioninfo/getActionNameMap'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.actionDataList = data.list
          } else {
            this.actionDataList = []
          }
        })
      },
      addConditionInfoList() {
        let newTeam = {
          conditionName: '', //条件名称
          conditionExpression: '', //条件表达式
          conditionDesc: '', //条件描述
          remark: ''
        }
        this.conditionInfoList.push(newTeam)
      },
      delConditionInfoList(index) {
        this.conditionInfoList.splice(index, 1)
      },
      addPropertyList() {
        let newTeam = {
          rulePropertyName: '', //名称
          defaultValue: '', //默认值
          rulePropertyDesc: '', //描述
          rulePropertyIdentify: '', //标识
          rulePropertyValue: '', //规则属性值
          rulePropertyId: '',
          remark: ''
        }
        this.propertyList.push(newTeam)
      },
      delPropertyList(index) {
        this.propertyList.splice(index, 1)
      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/rules/info/save`),
              method: 'post',
              data: this.$http.adornData({
                'ruleId': this.dataForm.ruleId || undefined,
                'sceneId': this.dataForm.sceneId,
                'ruleName': this.dataForm.ruleName,
                'ruleDesc': this.dataForm.ruleDesc,
                'ruleEnabled': this.dataForm.ruleEnabled,
                'isEffect': this.dataForm.isEffect,
                'creUserId': this.dataForm.creUserId,
                'creTime': this.dataForm.creTime,
                'remark': this.dataForm.remark,
                'conditionInfoItem': this.conditionInfoList,
                'propertyInfoItem': this.propertyList,
                'actionRuleRelItem': this.dataForm.actionRuleRelItem,
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
<style>
 /* .el-select .el-input {
    width: 130px;
  }
  .input-with-select .el-input-group__prepend {
    background-color: #fff;
  } */
</style>
