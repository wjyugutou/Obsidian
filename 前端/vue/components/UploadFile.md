```vue
<script setup lang="ts">
import modal from '@/plugins/modal'
import type { OssUploadData } from '@/api/system/oss'
import { delOss, getOssUploadUrl, listByIds } from '@/api/system/oss'
import { globalHeaders } from '@/utils/request'
import { genFileId } from 'element-plus'
import type { UploadFile, UploadFiles, UploadRawFile } from 'element-plus'

// 使用更严格的类型定义
type FileType = 'doc' | 'docx' | 'xls' | 'xlsx' | 'ppt' | 'pptx' | 'txt' | 'pdf' | 'png' | 'jpg' | 'jpeg'

// 使用泛型约束 modelValue 的类型
interface Props {
  modelValue?: OssUploadData[]
  /** 数量限制 */
  limit?: number
  /** 大小限制，单位 MB */
  fileSize?: number
  /** 允许的文件类型列表 */
  fileType?: FileType[]
  /** 是否显示提示 */
  isShowTip?: boolean
  /** 禁用组件（仅查看文件） */
  disabled?: boolean
  /** 移除时是否删除OSS文件；编辑既有业务附件时可仅更新关联。 */
  deleteRemote?: boolean
  /** 是否自动上传到 OSS；false 时仅本地选文件（导入等场景） */
  autoUpload?: boolean
  /** 超出数量限制时用新文件覆盖（常与 limit=1 联用） */
  replaceOnExceed?: boolean
  /** 是否启用拖拽上传区域 */
  drag?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: () => [],
  limit: 5,
  fileSize: 5,
  // fileType: () => ['doc', 'docx', 'xls', 'xlsx', 'ppt', 'pptx', 'txt', 'pdf'],
  isShowTip: true,
  disabled: false,
  deleteRemote: true,
  autoUpload: true,
  replaceOnExceed: false,
  drag: false,
})

const emit = defineEmits(['update:modelValue'])

const number = ref(0)
const uploadList = ref<any[]>([])

const uploadFileUrl = ref(getOssUploadUrl())
const headers = ref(globalHeaders())

const fileList = ref<any[]>([])
const showTip = computed(() => !!(props.isShowTip && (props.fileType || props.fileSize)))

const fileUploadRef = ref<ElUploadInstance>()

// 监听 fileType 变化，更新 fileAccept
const fileAccept = computed(() => props.fileType?.map(type => `.${type}`).join(','))

watch(
  () => props.modelValue,
  async (val) => {
    if (val) {
      let temp = 1
      // 首先将值转为数组
      let list: any[] = []
      if (Array.isArray(val)) {
        list = val
      }
      else {
        const res = await listByIds(val)
        list = res.data.map((oss) => {
          return {
            name: oss.originalName,
            url: oss.url,
            ossId: oss.ossId,
          }
        })
      }
      // 然后将数组转为对象数组
      fileList.value = list.map((item) => {
        const mapped: any = {
          name: item.name,
          url: item.url,
          ossId: item.ossId,
          uid: item.uid || Date.now() + temp++,
        }
        if (item.raw)
          mapped.raw = item.raw
        return mapped
      })
    }
    else {
      fileList.value = []
      return []
    }
  },
  { deep: true, immediate: true },
)

/** 校检格式和大小，不通过返回 false */
function validateFile(file: File): boolean {
  if (props.fileType.length) {
    const fileName = file.name.split('.')
    const fileExt = fileName[fileName.length - 1]?.toLowerCase()
    const isTypeOk = props.fileType.includes(fileExt as FileType)
    if (!isTypeOk) {
      modal.msgError(`文件格式不正确, 请上传${props.fileType.join('/')}格式文件!`)
      return false
    }
  }
  if (file.name.includes(',')) {
    modal.msgError('文件名不正确，不能包含英文逗号!')
    return false
  }
  if (props.fileSize) {
    const isLt = file.size / 1024 / 1024 < props.fileSize
    if (!isLt) {
      modal.msgError(`上传文件大小不能超过 ${props.fileSize} MB!`)
      return false
    }
  }
  return true
}

// 上传前校检格式和大小
function handleBeforeUpload(file: any) {
  if (!validateFile(file))
    return false
  if (!props.autoUpload)
    return false
  modal.loading('正在上传文件，请稍候...')
  number.value++
  return true
}

/** 本地模式：选择/拖拽后写入带 raw 的列表 */
function handleChange(uploadFile: UploadFile, uploadFiles: UploadFiles) {
  if (props.autoUpload)
    return
  if (uploadFile.status === 'fail')
    return

  const raw = uploadFile.raw
  if (raw && !validateFile(raw)) {
    fileList.value = fileList.value.filter(f => f.uid !== uploadFile.uid)
    emit('update:modelValue', listToModelValue(fileList.value))
    fileUploadRef.value?.handleRemove(uploadFile)
    return
  }

  const next = uploadFiles
    .filter(f => f.raw && f.status !== 'fail')
    .map(f => ({
      name: f.name,
      raw: f.raw,
      uid: f.uid,
      status: 'ready',
    }))
  fileList.value = props.limit > 0 ? next.slice(-props.limit) : next
  emit('update:modelValue', listToModelValue(fileList.value))
}

// 文件个数超出
function handleExceed(files: File[]) {
  if (props.replaceOnExceed && files[0]) {
    const file = files[0] as UploadRawFile
    if (!validateFile(file))
      return
    fileUploadRef.value?.clearFiles()
    file.uid = genFileId()
    fileUploadRef.value?.handleStart(file)
    if (props.autoUpload)
      fileUploadRef.value?.submit()
    return
  }
  modal.msgError(`上传文件数量不能超过 ${props.limit} 个!`)
}

// 上传失败
function handleUploadError() {
  modal.msgError('上传文件失败')
  modal.closeLoading()
}

// 上传成功回调
function handleUploadSuccess(res: any, file: UploadFile) {
  if (res.code === 200) {
    uploadList.value.push({
      name: res.data.fileName,
      url: res.data.url,
      ossId: res.data.ossId,
    })
    uploadedSuccessfully()
  }
  else {
    number.value--
    modal.closeLoading()
    modal.msgError(res.msg)
    fileUploadRef.value?.handleRemove(file)
    uploadedSuccessfully()
  }
}

// 删除文件
function handleDelete(index: number) {
  const ossId = fileList.value[index].ossId
  if (ossId && props.deleteRemote)
    delOss(ossId)
  fileList.value.splice(index, 1)
  emit('update:modelValue', listToModelValue(fileList.value))
}

// 上传结束处理
function uploadedSuccessfully() {
  if (number.value > 0 && uploadList.value.length === number.value) {
    fileList.value = fileList.value.filter(f => f.url !== undefined).concat(uploadList.value)
    uploadList.value = []
    number.value = 0
    emit('update:modelValue', listToModelValue(fileList.value))
    modal.closeLoading()
  }
}

// 获取文件名称
function getFileName(name: string) {
  // 如果是url那么取最后的名字 如果不是直接返回
  if (name.lastIndexOf('/') > -1) {
    return name.slice(name.lastIndexOf('/') + 1)
  }
  else {
    return name
  }
}

function listToModelValue(list: any[]): OssUploadData[] {
  if (!props.autoUpload) {
    return list.map(item => ({
      name: item.name,
      raw: item.raw,
      uid: item.uid,
    }))
  }
  return list
    .filter(item => item?.ossId)
    .map(item => ({
      name: item.name,
      url: item.url,
      ossId: item.ossId,
      fileName: item.name,
    }))
}
</script>

<template>
  <div class="upload-file">
    autoUpload: {{ autoUpload }}
    <el-upload
      v-if="!disabled"
      ref="fileUploadRef"
      class="upload-file-uploader"
      :drag="drag"
      :multiple="limit > 1"
      :auto-upload="autoUpload"
      :action="autoUpload ? uploadFileUrl : ''"
      :before-upload="handleBeforeUpload"
      :file-list="fileList"
      :limit="limit"
      :accept="fileAccept"
      :on-change="handleChange"
      :on-error="autoUpload ? handleUploadError : undefined"
      :on-exceed="handleExceed"
      :on-success="autoUpload ? handleUploadSuccess : undefined"
      :show-file-list="false"
      :headers="autoUpload ? headers : undefined"
    >
      <template v-if="drag">
        <el-icon class="el-icon--upload">
          <UploadFilled />
        </el-icon>
        <div class="el-upload__text">
          拖拽文件到此处，或<em>点击选择</em>
        </div>
      </template>
      <el-button v-else type="primary">
        选取文件
      </el-button>
    </el-upload>
    <!-- 上传提示 -->
    <div v-if="showTip && !disabled" class="el-upload__tip">
      请上传
      <template v-if="fileSize">
        大小不超过 <b style="color: #f56c6c">{{ fileSize }}MB</b>
      </template>
      <template v-if="fileType">
        格式为
        <ElTooltip v-if="fileType.length > 8" placement="top" :content="fileType.join('/')">
          <b style="color: #f56c6c" class="file-type">{{ fileType.join('/') }}</b>
        </ElTooltip>
        <b v-else style="color: #f56c6c" class="file-type">{{ fileType.join('/') }}</b>
      </template>
      的文件
    </div>
    <!-- 文件列表 -->
    <transition-group class="upload-file-list el-upload-list el-upload-list--text" name="el-fade-in-linear" tag="ul">
      <li v-for="(file, index) in fileList" :key="file.uid" class="el-upload-list__item ele-upload-list__item-content">
        <el-link v-if="file.url" :href="`${file.url}`" underline="never" target="_blank">
          <el-icon><document /></el-icon> {{ getFileName(file.name) }}
        </el-link>
        <span v-else class="upload-file-name">
          <el-icon><document /></el-icon> {{ getFileName(file.name) }}
        </span>
        <div class="ele-upload-list__item-content-action">
          <el-button v-if="!disabled" type="danger" link @click="handleDelete(index)">
            删除
          </el-button>
        </div>
      </li>
    </transition-group>
  </div>
</template>

<style lang="scss" scoped>
.upload-file-uploader {
  margin-bottom: 5px;

}
.file-type {
  display: inline-block;
  max-width: 200px;
  vertical-align: top;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.upload-file-list .el-upload-list__item {
  border: 1px solid #e4e7ed;
  line-height: 2;
  margin-bottom: 10px;
  position: relative;
}

.upload-file-list .ele-upload-list__item-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: inherit;
}

.upload-file-name {
  display: inline-flex;
  align-items: center;
  gap: 4px;
}

.ele-upload-list__item-content-action .el-link {
  margin-right: 10px;
}
</style>

```