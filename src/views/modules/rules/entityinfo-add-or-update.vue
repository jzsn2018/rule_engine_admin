<template>
  <el-dialog :title="!dataForm.entityId ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="名称" prop="entityName">
        <el-input v-model="dataForm.entityName" placeholder="名称"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="entityDesc">
        <el-input v-model="dataForm.entityDesc" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="标识" prop="entityIdentify">
        <el-input v-model="dataForm.entityIdentify" placeholder="标识"></el-input>
      </el-form-item>
      <el-form-item label="包路径" prop="pkgName">
        <el-input v-model="dataForm.pkgName" placeholder="包路径"></el-input>
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
      <el-form-item label="关联字段" class="flex-100" prop="item">
        <el-button type="primary" @click="addTeam">添加</el-button>
        <el-row v-for="(item,index) in fieldItem" :key="index" :gutter="20">
          <el-col :span="14">
            <el-input placeholder="请输入内容" v-model="item.itemName">
              <template slot="prepend">字段名</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.itemIdentify">
              <template slot="prepend">字段标识</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.itemDesc">
              <template slot="prepend">属性描述</template>
            </el-input>
            <el-input placeholder="请输入内容" v-model="item.remark">
              <template slot="prepend">备注</template>
            </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="delTeam(index)">删除</el-button>
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
        fieldItem: [], // 关联字段
        dataForm: {
          entityId: 0,
          entityName: '',
          entityDesc: '',
          entityIdentify: '',
          pkgName: '',
          isEffect: '',
          remark: '',
          fieldItemStr: ''
        },
        dataRule: {
          entityName: [{
            required: true,
            message: '名称不能为空',
            trigger: 'blur'
          }],
          /* entityDesc: [{
            required: true,
            message: '描述不能为空',
            trigger: 'blur'
          }], */
          entityIdentify: [{
            required: true,
            message: '标识不能为空',
            trigger: 'blur'
          }],
          pkgName: [{
            required: true,
            message: '包路径不能为空',
            trigger: 'blur'
          }],
          /* isEffect: [{
            required: true,
            message: '是否有效(1是0否)不能为空',
            trigger: 'blur'
          }] */
           item: [{
            required: false,
            message: '关联字段不能为空',
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
        this.dataForm.entityId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.entityId) {
            this.$http({
              url: this.$http.adornUrl(`/rules/entityinfo/info/${this.dataForm.entityId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({
              data
            }) => {
              if (data && data.code === 0) {
                this.dataForm.entityName = data.entityInfo.entityName
                this.dataForm.entityDesc = data.entityInfo.entityDesc
                this.dataForm.entityIdentify = data.entityInfo.entityIdentify
                this.dataForm.pkgName = data.entityInfo.pkgName
                //this.dataForm.isEffect = data.entityInfo.isEffect.toString()
                this.dataForm.remark = data.entityInfo.remark
                this.fieldItem = data.entityInfo.item
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
      addTeam() {
        let newTeam = {
          itemName: '',
          itemIdentify: '',
          itemDesc: '',
          remark: ''
        }
        this.fieldItem.push(newTeam)
      },
      delTeam(index) {
        this.fieldItem.splice(index, 1)
      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/rules/entityinfo/${!this.dataForm.entityId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.entityId || undefined,
                'entityName': this.dataForm.entityName,
                'entityDesc': this.dataForm.entityDesc,
                'entityIdentify': this.dataForm.entityIdentify,
                'pkgName': this.dataForm.pkgName,
                'creUserId': this.dataForm.creUserId,
                'creTime': this.dataForm.creTime,
                'isEffect': this.dataForm.isEffect,
                'remark': this.dataForm.remark,
                'item': this.fieldItem
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
