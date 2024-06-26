<template>
  <el-container class="common-layout">
    <el-main>
      <div style="display: flex; justify-content: end; margin-bottom: 20px">
        <el-input
          v-model="searchText"
          placeholder="请输入搜索内容"
          class="search-input"
          clearable
          @clear="handleSearchClear"
          @keydown.enter="handleSearch"
          style="width: 200px; margin-right: 20px"
        ></el-input>
        <el-button type="primary" size="default" @click="handleAdd">
          新增实验
        </el-button>
      </div>
      <el-table
        class="table"
        stripe
        border
        :data="filteredData"
        max-height="500"
      >
        <el-table-column fixed prop="id" label="ID" width="150" />
        <el-table-column prop="name" label="实验名称" width="280" />
        <el-table-column prop="timestamp" label="发布时间" width="370" />
        <el-table-column fixed="right" label="操作">
          <template #default="scope">
            <el-button
              type="primary"
              size="small"
              @click="handleClick('update', scope.row)"
            >
              修改
            </el-button>
            <el-button
              v-if="scope.row.state === 1"
              type="primary"
              size="small"
              color="#E5EAF3"
              @click="handleClick('retract', scope.row)"
            >
              撤回
            </el-button>
            <el-button v-else type="success" size="small" disabled>
              无法撤回
            </el-button>
            <el-button
              type="primary"
              size="small"
              color="red"
              @click="handleClick('delete', scope.row)"
            >
              删除
            </el-button>
            <el-button
              v-if="scope.row.state === 0"
              type="primary"
              size="small"
              @click="handleClick('publish', scope.row)"
            >
              发布
            </el-button>
            <el-button v-else type="success" size="small" disabled>
              已发布
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-main>
  </el-container>
  <!--  </el-container>-->
</template>

<script lang="ts" setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import http from '@/utils/http'
import { ElMessageBox } from 'element-plus'
import { ElMessage } from 'element-plus'

// Helper function to format timestamp
function formatTimestamp(timestamp: number[]): string {
  const [year, month, day, hour, minute, second] = timestamp
  return `${year}-${String(month).padStart(2, '0')}-${String(day).padStart(
    2,
    '0',
  )} ${String(hour).padStart(2, '0')}:${String(minute).padStart(
    2,
    '0',
  )}:${String(second).padStart(2, '0')}`
}

const searchText = ref('')
const router = useRouter()
const tableData = ref([]) // 初始化为空数组

const handleClick = async (action: string, row: any) => {
  if (action === 'delete') {
    try {
      const confirmDelete = await ElMessageBox.confirm('确定要删除吗？', '提示')
      if (confirmDelete) {
        await http.delete(`/exercise/${row.id}`)
        // 成功删除后，从 tableData 中移除对应的 row
        const index = tableData.value.findIndex((item) => item.id === row.id)
        if (index !== -1) {
          tableData.value.splice(index, 1) // 从响应式数据中移除
        }
        ElMessage.success('删除成功')
      }
    } catch (error) {
      console.error('删除失败:', error)
    }
  }
  if (action === 'publish') {
    await http.put(`/exercise/publish/${row.id}`)
    ElMessage.success('发布实验成功')
    // 更新数据列表中对应行的状态和按钮显示
    const index = tableData.value.findIndex((item) => item.id === row.id)
    if (index !== -1) {
      tableData.value[index].state = 1 // 假设 state 为 1 表示已发布
    }
  }

  if (action === 'retract') {
    await http.put(`/exercise/updateState/${row.id}`)
    ElMessage.success('撤回实验成功')
    // 更新数据列表中对应行的状态和按钮显示
    const index = tableData.value.findIndex((item) => item.id === row.id)
    if (index !== -1) {
      tableData.value[index].state = 0 // 假设 state 为 0 表示未发布
    }
  }

  if (action === 'update') {
    router.push({
      path: '/Editor',
      query: {
        id: row.id,
        name: row.name,
        content: row.content,
      },
    })
  }
}

const handleAdd = () => {
  // 实现添加新实验的逻辑
  router.push('/Editor')
}

const handleSearch = () => {
  // 实现搜索逻辑
}

const handleSearchClear = () => {
  // 处理清除搜索输入时的逻辑
  searchText.value = ''
}

const filteredData = computed(() => {
  if (!searchText.value) {
    return tableData.value
  }
  const filtered = tableData.value.filter((item) =>
    item.name.toLowerCase().includes(searchText.value.toLowerCase()),
  )
  return filtered
})

// 在组件挂载时发起 GET 请求
onMounted(async () => {
  try {
    const response = await http.get('/exercise/list')
    // 格式化 timestamp 并绑定数据到 tableData
    tableData.value = response.map((item: any) => ({
      ...item,
      timestamp: formatTimestamp(item.timestamp),
    }))
  } catch (error) {
    console.error('获取数据失败:', error)
  }
})
</script>

<style scoped>
/* 控制行样式 */
.control-row {
  margin-bottom: 15px;
}

.table {
  font-family: Consolas;
}

/* 搜索输入样式 */
.search-input {
  width: 100%;
}

.text-right {
  text-align: right;
}
</style>
