<template>
  <div class="device smart  ui-container">
    <base-table
      :height="tableHeight"
      :tableData="tableData"
      :columns="columns"
      stripe
      v-loading="tableLoading"
      :pageTotal="total"
      :pageSize="search.pageSize"
      :pageNo="search.pageNo"
      @on-current-page-change="onCurrentChange"
      @on-page-size-change="onSizeChange">

      <slot>
        <template slot="caption">
          <el-input clearable @keyup.enter.native="handleSearch" class="caption-item" placeholder="输入料号" v-model="search.materialNo"></el-input>
          <el-input clearable @keyup.enter.native="handleSearch" class="caption-item" placeholder="输入版本" v-model="search.versionNo"></el-input>
          <el-button type="primary" icon="el-icon-search" @click="handleSearch">查询</el-button>
        </template>
        <template slot="actionBar">
          <el-button type="primary" icon="obicon obicon-cloud-download" @click="dialogVisible = true">版本上传</el-button>
        </template>
      </slot>
    </base-table>
    <el-dialog  v-if="dialogVisible" top="10%" width="660px" title="上传版本" :visible.sync="dialogVisible" :close-on-click-modal="false">
      <el-form class="ob-form" ref="upload" autoComplete="on" :rules="modelRules" :model="model" label-width="80px" style="width: 90%; margin: 0 auto;">
        <el-form-item label="料号" prop="materialNo">
          <el-input v-model="model.materialNo" placeholder="输入料号"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="log">
          <el-input v-model="model.log" placeholder="输入描述信息"></el-input>
        </el-form-item>
        <el-form-item label="文件" prop="filepick">
          <span class="file-name">{{fileName}}</span>
          <el-upload
            class="upload-btn"
            ref="uploadBtn"
            accept=".bin"
            :data="uploadData"
            :show-file-list="false"
            :on-change="onUploadChange"
            :before-upload="onBeforeUpload"
            :on-success="onUploadSuccess"
            :on-error="onUploadFail"
            :auto-upload="false"
            action="/consumer/image/uploadFirmware">
            <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleUpload()">上传</el-button>
      </span>
    </el-dialog>

    <el-dialog  v-if="proveDialogVisible" top="10%" width="660px" title="固件校验" :visible.sync="proveDialogVisible" :close-on-click-modal="false">
      <el-form class="ob-form" ref="prove" autoComplete="on" :rules="proveRules" :model="proveModel" label-width="80px" style="width: 90%; margin: 0 auto;">
        <el-form-item label="手机号码" prop="mobile">
          <el-input v-model="proveModel.mobile" placeholder="输入手机号码"></el-input>
        </el-form-item>
        <el-form-item label="OBOX" prop="obox">
          <!-- <el-input v-model="proveModel.obox" placeholder=""></el-input> -->
          <el-select v-model="proveModel.obox" style="width: 100%;" clearable>
            <!-- <el-option :label="i" :value="''+ (i-1)"></el-option> -->
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="proveDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="doProve()">确认</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import BaseTable from '@/assets/package/table-base'
import SystemAPI from '@/api/system'
import { PAGINATION_PAGENO, PAGINATION_PAGESIZE } from '@/common/constants'
import Helper from '@/common/helper'
export default {
  data () {
    return {
      tableLoading: false,
      tableHeight: 0,
      total: 0,
      search: {
        materialNo: '',
        versionNo: '',
        pageNo: PAGINATION_PAGENO,
        pageSize: PAGINATION_PAGESIZE
      },
      tableData: [],
      columns: [],
      // 上传modal
      dialogVisible: false,
      model: {
        materialNo: '',
        log: '',
        filepick: ''
      },
      modelRules: {
        materialNo: [{ required: true, message: '料号不能为空', trigger: 'blur' }],
        log: [{ required: true, message: '描述信息不能为空', trigger: 'blur' }],
        filepick: [{ required: true, message: '文件不能为空', trigger: 'change' }],
      },
      fileName: '请选择文件',
      // 校验
      proveDialogVisible: false,
      proveModel: {
        mobile: '',
        obox: ''
      },
      proveRules: {
        mobile: [{ required: true, message: '手机号码不能为空', trigger: 'blur' }],
        obox: [{ required: true, message: '请选择obox', trigger: 'blur' }],
      }
    }
  },
  components: { BaseTable },
  created () {
    this.columns = this.getColumns()
    this.getVersionList()
  },
  computed: {
    layoutHeight () {
      return this.tableHeight + 180
    },
    uploadData () {
      return {
        materialNo: this.model.materialNo,
        log: this.model.log,
      }
    }
  },
  watch: {
    dialogVisible (val) {
      if (val === false) {
        this.$refs.upload.resetFields()
        this.$refs.uploadBtn.clearFiles()
        this.fileName = '请选择文件'
      }
    }
  },
  mounted () {
    Helper.windowOnResize(this, this.fixLayout)
  },
  methods: {
    fixLayout () {
      this.tableHeight = Helper.calculateTableHeight() - 20 - 10
    },
    getColumns () {
      return [{
        label: '料号',
        prop: 'materialNo',
        align: 'center'
      }, {
        label: '版本',
        prop: 'versionNo',
        align: 'center'
      }, {
        label: '存储路径',
        prop: 'url',
        align: 'center'
      }, {
        label: '描述',
        prop: 'log',
        align: 'center'
      }, {
        label: '状态',
        prop: 'status',
        align: 'center'
      }, {
        label: '上传时间',
        prop: 'optTime',
        align: 'center',
        formatter (val) {
          return val && Helper.parseTime(val)
        }
      }, {
        label: '操作',
        prop: 'operator',
        align: 'center',
        renderBody: this.getToolboxRender
      }]
    },
    getToolboxRender (h, row) {
      const toolbox = []
      const status = row.status || 2 // 1 - 已发布， 2 - 待校验， 3 - 校验成功、未发布， 4 - 校验失败
      const remove = <el-button class="colors" size="tiny" type="danger" icon="obicon obicon-trash" title='删除' onClick={() => this.handleRemove(row)}></el-button>
      const release = <el-button class="colors" size="tiny" type="success" icon="obicon obicon-send" title='发布' onClick={() => this.handleRelease(row)}></el-button>
      const prove = <el-button class="colors" size="tiny" type="warning" icon="obicon obicon-interaction" title='校验' onClick={() => this.handleProve(row)}></el-button>
      status === 2 && toolbox.push(prove) // 待校验
      status === 3 && toolbox.push(release) // 待发布
      toolbox.push(remove)
      return toolbox
    },
    getVersionList () {
      this.tableLoading = true
      SystemAPI.getVersionList(this.search).then(resp => {
        if (resp.status === 0) {
          this.tableData = resp.data.firmware
          this.total = resp.total
        }
        this.tableLoading = false
      }).catch(err => {
        this.tableLoading = false
      })
    },
    onCurrentChange (pageNo) {
      this.search.pageNo = pageNo
      this.getVersionList()
    },
    onSizeChange (pageSize) {
      this.search.pageSize = pageSize
      this.getVersionList()
    },
    handleSearch () {
      this.search.pageNo = PAGINATION_PAGENO
      this.getVersionList()
    },
    handleUpload () {
      const that = this
      this.$refs.upload.validate(valid => {
        if (valid) {
          if (that.$refs.uploadBtn.uploadFiles.length > 1) {
            that.$refs.uploadBtn.uploadFiles = [that.$refs.uploadBtn.uploadFiles.pop()]
          }
          if (!that.$refs.uploadBtn.uploadFiles.length) {
            return this.$message({
              title: false,
              type: 'warning',
              message: '请重新选择文件'
            })
          }
          that.$refs.uploadBtn.submit()
        }
      })
    },
    onUploadChange (file, fileList) {
      if (file) {
        this.fileName = file.name
        this.model.filepick = file.name
      } else {
        this.model.filepick = ''
      }
    },
    onBeforeUpload (file) {
      if (!file) {
        return this.$message({
          title: false,
          type: 'warning',
          message: '请选择文件'
        })
      }
      const suffix = file.name && file.name.slice(file.name.lastIndexOf('.') + 1)
      if (!['bin'].includes(suffix)) {
        this.$message({
          type: 'error',
          message: '文件类型错误'
        })
        return false
      }
      this.loader = this.$loading({
        text: '文件上传中'
      })
    },
    onUploadSuccess (response, file, fileList) {
      this.loader && this.loader.close()
      if (response.status === 0) {
        this.dialogVisible = false
        this.getVersionList()
      } else if (response.status === 900) {
        this.$message({
          type: 'error',
          message: response.message || '文件名格式不正确，请按照约定的格式命名文件'
        })
        // file.status = 'ready'
        this.$refs.uploadBtn.clearFiles()
      } else {
        this.$message({
          type: 'error',
          message: response.message || '上传失败'
        })
        // file.status = 'ready'
        this.$refs.uploadBtn.clearFiles()
      }
    },
    onUploadFail (response, file, fileList) {
      this.loader && this.loader.close()
      this.$message({
        type: 'error',
        message: '上传失败'
      })
    },
    handleRemove (row) {
      this.$confirm('确认删除该记录？', '确认提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        closeOnClickModal: false
      }).then(() => {
        this.doRemove(row.id)
      }).catch(() => {
        console.log('cancel')
      })
    },
    doRemove (id) {
      SystemAPI.deleteVersion(id).then(resp => {
        if (resp.status === 0) {
          this.$message({
            type: 'success',
            message: '删除成功'
          })
          this.getVersionList()
        } else {
          this.$message({
            type: 'error',
            message: '删除失败'
          })
        }
      }).catch(() => {
        this.$message({
          type: 'error',
          message: '服务异常'
        })
      })
    },
    handleRelease (row) {
      console.log('release')
    },
    handleProve (row) {
      console.log('prove')
      this.proveDialogVisible = true
    },
    doProve () {

    }
  }
}
</script>
<style lang="scss" scoped>
  .ui-container{
    margin: 20px;
    border-radius: 14px;
  }
  .upload-btn{
    float: right;
  }
</style>
<style lang="scss">
  // .colors{
  //   &{
  //     color: #FFFFFF!important;
  //   }
  //   &.el-button--warning {
  //     background-color: #E6A23C!important;
  //     border-color: #E6A23C!important;
  //   }
  //   &.el-button--success {
  //     background-color: #67C23A!important;
  //     border-color: #67C23A!important;
  //   }
  //   &.el-button--danger {
  //     background-color: #F56C6C!important;
  //     border-color: #F56C6C!important;
  //   }
  // }
</style>
