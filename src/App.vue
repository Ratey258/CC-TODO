<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import Icons from './components/Icons.vue'

// 响应式数据
const todos = ref([])
const name = ref('')
const input_content = ref('')
const input_category = ref(null)
const isDarkMode = ref(false)
const showDeleteConfirm = ref(false)
const taskToDelete = ref(null)
const inputError = ref('')

// 监听主题变化
watch(isDarkMode, (newVal) => {
	localStorage.setItem('darkMode', JSON.stringify(newVal))
	document.documentElement.setAttribute('data-theme', newVal ? 'dark' : 'light')
})

// 监听name的变化，保存到localStorage
watch(name, (newVal) => {
	localStorage.setItem('name', newVal)
})

// 监听todos的变化，保存到localStorage
watch(todos, (newVal) => {
	localStorage.setItem('todos', JSON.stringify(newVal))
}, {
	deep: true
})

// 切换主题
const toggleTheme = () => {
	isDarkMode.value = !isDarkMode.value
}

// 添加新待办事项的函数
const addTodo = () => {
	if (!validateInput()) {
		return
	}
	
	todos.value.push({
		content: input_content.value.trim(),
		category: input_category.value,
		done: false,
		editable: false,
		createdAt: new Date().getTime()
	})
	
	// 清空输入字段
	input_content.value = ''
	input_category.value = null
	inputError.value = ''
}

// 快速添加任务的函数
const quickAddTask = (taskContent, category = 'personal') => {
	todos.value.push({
		content: taskContent,
		category: category,
		done: false,
		editable: false,
		createdAt: new Date().getTime()
	})
}

// 输入验证函数
const validateInput = () => {
	inputError.value = ''
	
	if (input_content.value.trim().length === 0) {
		inputError.value = '请输入任务内容'
		return false
	}
	
	if (input_content.value.length > 100) {
		inputError.value = '任务内容不能超过100个字符'
		return false
	}
	
	if (!input_category.value) {
		inputError.value = '请选择任务分类'
		return false
	}
	
	return true
}

// 按创建时间升序排列待办事项
const todos_asc = computed(() => todos.value.sort((a, b) => {
	return a.createdAt - b.createdAt
}))

// 显示删除确认对话框
const showDeleteDialog = (todo) => {
	taskToDelete.value = todo
	showDeleteConfirm.value = true
}

// 确认删除
const confirmDelete = () => {
	if (taskToDelete.value) {
		todos.value = todos.value.filter((t) => t !== taskToDelete.value)
	}
	cancelDelete()
}

// 取消删除
const cancelDelete = () => {
	showDeleteConfirm.value = false
	taskToDelete.value = null
}

// 删除待办事项的函数（保持向后兼容）
const removeTodo = (todo) => {
	showDeleteDialog(todo)
}

// 统计数据
const totalTodos = computed(() => todos.value.length)
const completedTodos = computed(() => todos.value.filter(todo => todo.done).length)
const progressPercentage = computed(() => {
	if (totalTodos.value === 0) return 0
	return Math.round((completedTodos.value / totalTodos.value) * 100)
})

// 组件加载时从localStorage加载数据
onMounted(() => {
	name.value = localStorage.getItem('name') || ''
	todos.value = JSON.parse(localStorage.getItem('todos')) || []
	isDarkMode.value = JSON.parse(localStorage.getItem('darkMode')) || false
	document.documentElement.setAttribute('data-theme', isDarkMode.value ? 'dark' : 'light')
})

</script>

<template>
	<div class="app">
		<!-- 背景装饰 -->
		<div class="bg-decoration">
			<div class="gradient-orb orb-1"></div>
			<div class="gradient-orb orb-2"></div>
			<div class="gradient-orb orb-3"></div>
		</div>

		<!-- 顶部导航栏 -->
		<nav class="navbar glass">
			<div class="nav-content">
				<h1 class="app-title">
					<span class="icon">
						<Icons name="app" />
					</span>
					CC-TODO
				</h1>
				<button class="theme-toggle" @click="toggleTheme" :class="{ active: isDarkMode }">
					<span class="theme-icon">
						<Icons :name="isDarkMode ? 'sun' : 'moon'" />
					</span>
				</button>
			</div>
		</nav>

		<main class="main-content">
			<!-- 问候区域 -->
			<section class="greeting glass">
				<div class="greeting-content">
					<h2 class="greeting-title">
						<span class="greeting-text">你好，</span>
						<input 
							type="text" 
							class="name-input" 
							placeholder="请输入姓名" 
							v-model="name"
							maxlength="20"
						>
						<span class="wave">👋</span>
					</h2>
					
					<!-- 进度统计 -->
					<div class="stats-grid" v-if="totalTodos > 0">
						<div class="stat-card primary">
							<div class="stat-icon">📊</div>
							<div class="stat-content">
								<div class="stat-number">{{ totalTodos }}</div>
								<div class="stat-label">总任务</div>
							</div>
						</div>
						<div class="stat-card success">
							<div class="stat-icon">✅</div>
							<div class="stat-content">
								<div class="stat-number">{{ completedTodos }}</div>
								<div class="stat-label">已完成</div>
							</div>
						</div>
						<div class="stat-card info">
							<div class="stat-icon">📈</div>
							<div class="stat-content">
								<div class="stat-number">{{ progressPercentage }}%</div>
								<div class="stat-label">完成率</div>
								<div class="progress-bar">
									<div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</section>

			<!-- 创建待办事项 -->
			<section class="create-todo glass">
				<div class="section-header">
					<h3 class="section-title">
						<span class="section-icon">
							<Icons name="plus" />
						</span>
						创建新任务
					</h3>
				</div>
				
				<form @submit.prevent="addTodo" class="todo-form">
					<div class="form-group">
						<label class="form-label">你想做什么？</label>
						<input 
							type="text" 
							class="form-input"
							:class="{ 'error': inputError }"
							placeholder="例如：去跑步..."
							v-model="input_content"
							@input="validateInput"
							maxlength="100"
						/>
						<div v-if="inputError" class="error-message">
							{{ inputError }}
						</div>
						<div class="input-counter" :class="{ 'warning': input_content.length > 80 }">
							{{ input_content.length }}/100
						</div>
					</div>
					
					<div class="form-group">
						<label class="form-label">选择分类</label>
						<div class="category-description">
							选择合适的分类来更好地组织你的任务
						</div>
						<div class="category-options">
							<label class="category-option" :class="{ active: input_category === 'business' }">
								<input 
									type="radio" 
									name="category" 
									value="business"
									v-model="input_category" 
								/>
								<div class="category-content">
									<span class="category-icon">
										<Icons name="briefcase" />
									</span>
									<div class="category-info">
										<span class="category-name">工作</span>
										<span class="category-desc">项目任务、会议、报告等</span>
									</div>
								</div>
								<div class="category-indicator business"></div>
							</label>
							
							<label class="category-option" :class="{ active: input_category === 'personal' }">
								<input 
									type="radio" 
									name="category" 
									value="personal"
									v-model="input_category" 
								/>
								<div class="category-content">
									<span class="category-icon">
										<Icons name="target" />
									</span>
									<div class="category-info">
										<span class="category-name">个人</span>
										<span class="category-desc">学习、运动、生活等</span>
									</div>
								</div>
								<div class="category-indicator personal"></div>
							</label>
						</div>
					</div>
					
					<button type="submit" class="add-button" :disabled="!input_content.trim() || !input_category">
						<span class="button-icon">
							<Icons name="star" />
						</span>
						添加任务
					</button>
				</form>
			</section>

			<!-- 待办事项列表 -->
			<section class="todo-list">
				<!-- 空状态 -->
				<div v-if="todos.length === 0" class="empty-state glass">
					<div class="empty-illustration">
						<Icons name="celebrate" class="empty-icon" />
					</div>
					<h3 class="empty-title">开始你的第一个任务吧！</h3>
					<p class="empty-description">
						创建任务来管理你的日程，让生活更有条理
					</p>
					<div class="empty-suggestions">
						<div class="suggestion-item" @click="quickAddTask('学习新技能', 'personal')">
							💡 学习新技能
						</div>
						<div class="suggestion-item" @click="quickAddTask('锻炼身体', 'personal')">
							🏃 锻炼身体
						</div>
						<div class="suggestion-item" @click="quickAddTask('整理房间', 'personal')">
							🏠 整理房间
						</div>
						<div class="suggestion-item" @click="quickAddTask('完成项目报告', 'work')">
							📊 完成项目报告
						</div>
					</div>
				</div>

				<div class="section-header" v-if="todos.length > 0">
					<h3 class="section-title">
						<span class="section-icon">
							<Icons name="list" />
						</span>
						我的任务
					</h3>
				</div>

				<div class="todos-container" v-if="todos.length > 0">
					<TransitionGroup name="todo" tag="div" class="todos-list">
						<div 
							v-for="todo in todos_asc" 
							:key="todo.createdAt"
							class="todo-item glass"
							:class="{ 
								'completed': todo.done,
								'business': todo.category === 'business',
								'personal': todo.category === 'personal'
							}"
						>
							<label class="todo-checkbox">
								<input type="checkbox" v-model="todo.done" />
								<span class="checkmark">
									<span class="check-icon">
										<Icons name="check" />
									</span>
								</span>
							</label>

							<div class="todo-content">
								<input 
									type="text" 
									v-model="todo.content" 
									class="todo-text"
									maxlength="100"
								/>
								<div class="todo-category">
											<span class="category-badge" :class="todo.category">
												<span class="badge-icon">
													<Icons :name="todo.category === 'business' ? 'briefcase' : 'target'" />
												</span>
												{{ todo.category === 'business' ? '工作' : '个人' }}
											</span>
								</div>
							</div>

							<div class="todo-actions">
									<button 
										class="action-button delete" 
										@click="removeTodo(todo)"
										title="删除任务"
									>
									<span class="action-icon">
										<Icons name="trash" />
									</span>
								</button>
							</div>
						</div>
					</TransitionGroup>
				</div>

				<!-- 空状态 -->
				<div class="empty-state glass" v-else>
					<div class="empty-icon">
						<Icons name="celebrate" />
					</div>
					<h3 class="empty-title">全部完成！</h3>
					<p class="empty-text">您暂时没有任务。在上方创建一个任务开始吧。</p>
				</div>
			</section>
		</main>

		<!-- 删除确认对话框 -->
		<div v-if="showDeleteConfirm" class="modal-overlay" @click="cancelDelete">
			<div class="delete-modal" @click.stop>
				<div class="modal-icon">
					<Icons name="trash" class="warning-icon" />
				</div>
				<h3 class="modal-title">确认删除任务？</h3>
				<p class="modal-message">
					"{{ taskToDelete?.content }}" 将被永久删除，此操作无法撤销。
				</p>
				<div class="modal-actions">
					<button class="btn btn-secondary" @click="cancelDelete">
						取消
					</button>
					<button class="btn btn-danger" @click="confirmDelete">
						删除
					</button>
				</div>
			</div>
		</div>
	</div>
</template>
