<template>
  <div class="errorLogContainer">
    <!--操作-->
    <div class="mr-3 rowSS">
      <el-button type="primary" @click="errorLogProd">错误日志测试</el-button>
      <el-button type="primary" @click="errorLogImg">图片加载错误测试</el-button>
      <el-button type="primary" @click="multiDelBtnClick">
        <!-- 感觉写法复杂了-->
        <el-icon style="vertical-align: middle">
          <Delete />
        </el-icon>
        <span style="vertical-align: middle">删除</span>
      </el-button>
      <!--条件搜索-->
      <el-form ref="refsearchFormMixin" :inline="true" class="demo-searchFormMixin ml-2">
        <el-form-item label-width="0px" label="" prop="errorLog" label-position="left">
          <el-input v-model="searchFormMixin.errorLog" class="widthPx-150" placeholder="错误日志" />
        </el-form-item>
        <el-form-item label-width="0px" label="" prop="pageUrl" label-position="left">
          <el-input v-model="searchFormMixin.pageUrl" class="widthPx-150" placeholder="页面路径" />
        </el-form-item>
        <el-form-item label-width="0px" label="" prop="createTime" label-position="left">
          <el-date-picker
            v-model="startEndArrMixin"
            type="datetimerange"
            format="YYYY-MM-DD"
            value-format="YYYY-MM-DD HH:mm:ss"
            @change="dateTimePacking"
            class="widthPx-250"
            range-separator="-"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
          />
        </el-form-item>
      </el-form>
      <!--查询按钮-->
      <el-button @click="searchBtnClick">查询</el-button>
    </div>
    <!--表格和分页-->
    <el-table
      id="resetElementDialog"
      ref="refuserTable"
      :height="`calc(100vh - ${settings.delWindowHeight})`"
      size="mini"
      border
      @selection-change="handleSelectionChange"
      :data="usertableData"
    >
      <el-table-column type="selection" align="center" width="50" />
      <el-table-column align="center" prop="errorLog" label="错误日志" min-width="300" />
      <el-table-column align="center" prop="pageUrl" label="页面路径" width="180" />
      <el-table-column align="center" prop="version" label="版本号" width="60" />
      <el-table-column align="center" prop="browserType" label="浏览器类型" width="180" />
      <el-table-column align="center" prop="createTime" label="创建时间" width="140" />
      <!--点击操作-->
      <el-table-column fixed="right" align="center" label="操作" width="80">
        <template #default="{ row }">
          <el-button type="text" size="small" @click="tableDelClick(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--分页-->
    <div class="block columnCC mt-2">
      <el-pagination
        :current-page="pageNum"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="pageTotalMixin"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
    <!--详情-->
    <el-dialog
      v-if="detailDialogMixin"
      :title="dialogTitleMixin"
      v-model="detailDialogMixin"
      width="40vw"
      :close-on-click-modal="false"
    >
      <div class="detail-container rowBC elODialogModalBodyH60vh">
        <div class="detail-container-item">DBC文件名：{{ detailData.username }}</div>
      </div>
      <div class="detail-container rowBC elODialogModalBodyH60vh">
        <div class="detail-container-item" style="color: green" v-if="detailData.status === 1">状态： 启用</div>
        <div class="detail-container-item" v-else>状态： 停用</div>
      </div>
      <div class="detail-container rowBC elODialogModalBodyH60vh">
        <div class="detail-container-item">说明：{{ detailData.remark }}</div>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button type="primary" @click="detailDialogMixin = false">确 定</el-button>
        </span>
      </template>
    </el-dialog>
    <!--图片错误demo-->
    <img v-if="imgShow" src="http://img.png" />
  </div>
</template>

<script setup lang="ts">
import { Delete } from '@element-plus/icons'
import bus from '@/utils/bus'
import { onMounted, getCurrentInstance, ref, reactive } from 'vue'
import settings from '@/settings'
let { proxy }: any = getCurrentInstance()

/*
 * 一般根据页面层次来排序 如此页面 表格查询和筛选->table的操作
 * 每个模块按：响应数据定义->公用方法->请求方法->页面按钮操作方法 进行排序
 * */

let errorLogProd = () => {
  throw new Error('产生的错误日志')
}
//img loader err test
let imgShow = ref(false)
const errorLogImg = () => {
  imgShow.value = !imgShow.value
}
/*表格查询和筛选*/
let usertableData = ref([])
let searchFormMixin: ObjTy = reactive({
  errorLog: '',
  pageUrl: '',
  createTime: '',
  id: ''
})
let selectPageReq = () => {
  const data: ObjTy = Object.assign(searchFormMixin, {
    pageNum: pageNum,
    pageSize: pageSize
  })
  Object.keys(data).forEach((fItem) => {
    if (data[fItem] === '' || data[fItem] === null || data[fItem] === undefined) delete data[fItem]
  })
  let reqConfig = {
    url: '/ty-user/errorCollection/selectPage',
    method: 'get',
    data,
    isParams: true,
    isAlertErrorMsg: false
  }
  proxy.$axiosReq(reqConfig).then((resData: ObjTy) => {
    usertableData.value = resData.data?.records
    proxy.pageTotalMixin = resData.data?.total
  })
}
import tablePageHook from '@/hooks/tablePageHook'
import { ObjTy } from '@/types/common'
let { pageNum, pageSize, handleCurrentChange, handleSizeChange } = tablePageHook(selectPageReq)
const dateTimePacking = (timeArr: Array<string>) => {
  if (timeArr && timeArr.length === 2) {
    searchFormMixin.startTime = timeArr[0]
    searchFormMixin.endTime = timeArr[1]
  } else {
    searchFormMixin.startTime = ''
    searchFormMixin.endTime = ''
  }
}
onMounted(() => {
  selectPageReq()
  bus.on('reloadErrorPage', () => {
    selectPageReq()
  })
})
const searchBtnClick = () => {
  pageNum.value = 1
  selectPageReq()
}

/*添加和修改*/
/*详情*/
let detailData = ref({})

/*删除*/
let deleteByIdReq = (id: string) => {
  return proxy.$axiosReq({
    url: '/ty-user/errorCollection/deleteById',
    data: { id: id },
    isParams: true,
    method: 'delete',
    bfLoading: true
  })
}
let tableDelClick = async (row: ObjTy) => {
  await proxy
    .elConfirmMixin('确定', `您确定要删除【${row.pageUrl}】吗？`)
    .then(() => {
      deleteByIdReq(row.id).then(() => {
        selectPageReq()
        proxy.elMessageMixin(`【${row.pageUrl}】删除成功`)
      })
    })
    //不写catch会报错
    .catch(() => {})
}

/*批量删除*/
let multipleSelection = ref([])
const handleSelectionChange = (val: any) => {
  multipleSelection.value = val
}
const multiDelBtnClick = async () => {
  let rowDeleteIdArrMixin = []
  // let selectionArr = proxy.$refs.refuserTable //--c
  let deleteNameTitle = ''
  rowDeleteIdArrMixin = multipleSelection.value.map((mItem: ObjTy) => {
    deleteNameTitle = deleteNameTitle + mItem.pageUrl + ','
    return mItem.id
  })
  if (rowDeleteIdArrMixin.length === 0) {
    proxy.elMessageMixin('表格选项不能为空', 'warning')
    return
  }
  let stringLength = deleteNameTitle.length - 1
  await proxy.elConfirmMixin('删除', `您确定要删除【${deleteNameTitle.slice(0, stringLength)}】吗`)
  const data = rowDeleteIdArrMixin
  proxy
    .$axiosReq({
      url: `/ty-user/errorCollection/deleteBatchIds`,
      data,
      method: 'DELETE',
      bfLoading: true
    })
    .then(() => {
      proxy.elMessageMixin('删除成功')
      selectPageReq()
    })
}
</script>

<style scoped lang="scss">
/*详情*/
.detail-container {
  flex-wrap: wrap;
}

.detail-container-item {
  min-width: 40%;
  margin-bottom: 20px;
}

.detailDialog-title {
  margin-bottom: 14px;
  font-weight: bold;
  font-size: 16px;
}
</style>
