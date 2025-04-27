<template>
  <div class="container">
    <h2>📝 小鹿的代办事项</h2>
    <el-form :inline="true" class="form">
      <el-input v-model="newTodo.content" placeholder="请输入任务内容" class="input" />
      <el-date-picker v-model="newTodo.doTime" type="datetime" placeholder="处理时间" class="date" />

      <el-button type="primary" @click="addTodo" class="button">添加任务</el-button>
      <el-switch v-model="onlyUnfinished" active-text="未完成" class="switch" @change="() => fetchTodos(1)" />

    </el-form>

    <el-table :data="todoList" stripe class="table">
      <el-table-column label="序号" width="60" class="index-column">
        <template #default="{ $index }">
          {{ ($index + 1) + (currentPage - 1) * pageSize }}
        </template>
      </el-table-column>
      <el-table-column prop="content" label="内容">
        <template #default="{ row }">
          <span :style="{ color: row.done ? '#67C23A' : '#E6A23C' }">
            {{ row.content }}</span>
        </template>

      </el-table-column>
      <el-table-column label="处理时间">
        <template #default="{ row }">
          {{ new Date(row.doTime).toLocaleString('zh-CN', {
          
            month: '2-digit',
            day: '2-digit',
            hour: '2-digit',
            minute: '2-digit'
          }).replace(/\//g, '-') }}
        </template>
      </el-table-column>
      <el-table-column label="创建时间" width="180">
        <template #default="{ row }">
          {{ new Date(row.createdAt).toLocaleString('zh-CN', {
            year: 'numeric',
            month: '2-digit',
            day: '2-digit',
            hour: '2-digit',
            minute: '2-digit'
          }).replace(/\//g, '-') }}
        </template>
      </el-table-column>

      <el-table-column label="状态" width="100">
        <template #default="{ row }">
          <el-tag :type="row.done ? 'success' : 'info'">
            {{ row.done ? '已完成' : '未完成' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180" fixed="right">
        <template #default="{ row }">
          <el-button size="small" type="primary" @click="toggleDone(row)">
            切换状态
          </el-button>
          <el-button size="small" type="danger" @click="deleteTodo(row.id)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination class="pagination" v-model:current-page="currentPage" :page-size="pageSize" :total="total"
      layout="total, sizes, prev, pager, next" :page-sizes="[5, 10, 20, 50]" @current-change="fetchTodos(currentPage)"
      @size-change="(val) => { pageSize = val; fetchTodos(1); }" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'

import axios from 'axios'
const baseURL = process.env.NODE_ENV === 'development' ? 'http://localhost:8081' : ''

const todoList = ref([])
const currentPage = ref(1)
const pageSize = ref(10)
const total = ref(0)
const onlyUnfinished = ref(true)

const newTodo = ref({
  content: '',
  doTime: ''
})

const fetchTodos = async (page = 1) => {
  console.log("chakan", page)
  try {
    const res = await axios.get(`${baseURL}/todos`, {
      params: {
        page,
        pageSize: pageSize.value,
        onlyUnfinished: onlyUnfinished.value,
      }
    })
    todoList.value = res.data.data
    total.value = res.data.total
    currentPage.value = page
  } catch (err) {
    console.error('获取任务失败', err)
  }
}


const addTodo = async () => {
  if (!newTodo.value.content || !newTodo.value.doTime) {
    ElMessage({
      message: '请输入内容和处理时间',
      type: 'warning',
    })
    return
  }

  try {
    await axios.post(`${baseURL}/todos`, {
      content: newTodo.value.content,
      done: false,
      doTime: newTodo.value.doTime
    })
    newTodo.value.content = ''
    newTodo.value.doTime = ''
    fetchTodos(currentPage.value)
  } catch (err) {
    console.error('添加任务失败', err)
  }
}

const toggleDone = async (item) => {
  try {
    await axios.put(`${baseURL}/todos/${item.id}`, {
      ...item,
      done: !item.done
    })
    item.done = !item.done
    //fetchTodos(currentPage.value)
  } catch (err) {
    console.error('更新失败', err)
  }
}

const deleteTodo = async (id) => {
  try {
    await axios.delete(`${baseURL}/todos/${id}`)
    fetchTodos(currentPage.value)
  } catch (err) {
    console.error('删除失败', err)
  }
}

onMounted(() => {
  fetchTodos(currentPage.value)
})
</script>

<style>
.container {
  padding: 1rem;
}

.form {
  display: grid;
  grid-template-columns: 0.5fr 0.4fr 0.2fr 1.5fr;
  gap: 0.5rem
}

@media screen and (max-width: 768px) {
  .form {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.5rem
  }

  .index-column {
    display: none;
  }

}

.pagination {
  margin-top: 1rem;
  display: flex;
  justify-content: center;
}
</style>
