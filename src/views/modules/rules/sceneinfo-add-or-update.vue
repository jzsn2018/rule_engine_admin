<template>
  <el-dialog :title="!dataForm.sceneId ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="名称" prop="sceneName">
        <el-input v-model="dataForm.sceneName" placeholder="名称"></el-input>
      </el-form-item>
      <el-form-item label="标识" prop="sceneIdentify">
        <el-input v-model="dataForm.sceneIdentify" placeholder="标识"></el-input>
      </el-form-item>
      <!-- <el-form-item label="类型" prop="sceneType">
        <el-input v-model="dataForm.sceneType" placeholder="类型"></el-input>
      </el-form-item> -->
      <el-form-item label="描述" prop="sceneDesc">
        <el-input v-model="dataForm.sceneDesc" placeholder="描述"></el-input>
      </el-form-item>
      <!-- <el-form-item label="是否有效" prop="isEffect">
      <el-select v-model="dataForm.isEffect" placeholder="是否有效">
        <el-option v-for="item in isEffectList" :key="item.value" :label="item.value" :value="item.code">
        </el-option>
      </el-select>
    </el-form-item> -->
      <el-form-item label="关联实体" prop="entityId">
        <el-select v-model="dataForm.entityId" placeholder="请选择实体名称" multiple collapse-tags>
          <el-option v-for="item in entityList" :key="item.id" :label="item.fieldName" :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
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
        entityList: [],
        dataForm: {
          sceneId: 0,
          sceneIdentify: '',
          sceneType: '',
          sceneName: '',
          sceneDesc: '',
          isEffect: '',
          remark: '',
          entityId: ''
        },
        dataRule: {
          sceneIdentify: [{
            required: true,
            message: '标识不能为空',
            trigger: 'blur'
          }],
           entityId: [{
            required: true,
            message: '关联实体不能为空',
            trigger: 'blur'
          }],
          sceneName: [{
            required: true,
            message: '名称不能为空',
            trigger: 'blur'
          }],
          sceneDesc: [{
            required: true,
            message: '描述不能为空',
            trigger: 'blur'
          }],
          /* isEffect: [{
            required: true,
            message: '是否有效不能为空',
            trigger: 'blur'
          }], */
          /* remark: [{
            required: false,
            message: '备注不能为空',
            trigger: 'blur'
          }] */
        }
      }
    },
    mounted() {
      (async () => {
        await this.getisEffectListDict();
        await this.selectEntityName();
      })()
    },
    methods: {
      init(id) {
        this.dataForm.sceneId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.sceneId) {
            this.$http({
              url: this.$http.adornUrl(`/rules/sceneinfo/info/${this.dataForm.sceneId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({
              data
            }) => {
              if (data && data.code === 0) {
                this.dataForm.sceneIdentify = data.sceneInfo.sceneIdentify
                this.dataForm.sceneType = data.sceneInfo.sceneType
                this.dataForm.sceneName = data.sceneInfo.sceneName
                this.dataForm.sceneDesc = data.sceneInfo.sceneDesc
                this.dataForm.isEffect = data.sceneInfo.isEffect.toString()
                this.dataForm.remark = data.sceneInfo.remark
                this.dataForm.entityId = data.sceneInfo.entityId
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
      selectEntityName() {
        this.$http({
          url: this.$http.adornUrl('/rules/entityinfo/listEntityName'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.entityList = data.list;
          } else {
            this.$message.error(data.msg)
          }
        })


      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          console.log(this.dataForm.entityId,'log')
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/rules/sceneinfo/${!this.dataForm.sceneId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'sceneId': this.dataForm.sceneId || undefined,
                'sceneIdentify': this.dataForm.sceneIdentify,
                'sceneType': this.dataForm.sceneType,
                'sceneName': this.dataForm.sceneName,
                'sceneDesc': this.dataForm.sceneDesc,
                'isEffect': this.dataForm.isEffect,
                'creUserId': this.dataForm.creUserId,
                'creTime': this.dataForm.creTime,
                'remark': this.dataForm.remark,
                'entityId': this.dataForm.entityId
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
