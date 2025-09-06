<script setup>
import { ref, onMounted, computed, watch } from 'vue'

// 响应式数据
const todos = ref([])
const name = ref('')
const input_content = ref('')
const input_category = ref(null)
const isDarkMode = ref(false)

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
	if (input_content.value.trim() === '' || input_category.value === null) {
		return
	}
	
	todos.value.push({
		content: input_content.value,
		category: input_category.value,
		done: false,
		editable: false,
		createdAt: new Date().getTime()
	})
	
	// 清空输入字段
	input_content.value = ''
	input_category.value = null
}

// 按创建时间升序排列待办事项
const todos_asc = computed(() => todos.value.sort((a, b) => {
	return a.createdAt - b.createdAt
}))

// 删除待办事项的函数
const removeTodo = (todo) => {
	todos.value = todos.value.filter((t) => t !== todo)
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
					<span class="icon">✨</span>
					TodoZen 2025
				</h1>
				<button class="theme-toggle" @click="toggleTheme" :class="{ active: isDarkMode }">
					<span class="theme-icon">{{ isDarkMode ? '☀️' : '🌙' }}</span>
				</button>
			</div>
		</nav>

		<main class="main-content">
			<!-- 问候区域 -->
			<section class="greeting glass">
				<div class="greeting-content">
					<h2 class="greeting-title">
						<span class="greeting-text">Hello there,</span>
						<input 
							type="text" 
							class="name-input" 
							placeholder="Your name" 
							v-model="name"
							maxlength="20"
						>
						<span class="wave">👋</span>
					</h2>
					
					<!-- 进度统计 -->
					<div class="stats-grid" v-if="totalTodos > 0">
						<div class="stat-card">
							<div class="stat-number">{{ totalTodos }}</div>
							<div class="stat-label">Total Tasks</div>
						</div>
						<div class="stat-card">
							<div class="stat-number">{{ completedTodos }}</div>
							<div class="stat-label">Completed</div>
						</div>
						<div class="stat-card">
							<div class="stat-number">{{ progressPercentage }}%</div>
							<div class="stat-label">Progress</div>
							<div class="progress-bar">
								<div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
							</div>
						</div>
					</div>
				</div>
			</section>

			<!-- 创建待办事项 -->
			<section class="create-todo glass">
				<div class="section-header">
					<h3 class="section-title">
						<span class="section-icon">➕</span>
						Create New Task
					</h3>
				</div>
				
				<form @submit.prevent="addTodo" class="todo-form">
					<div class="form-group">
						<label class="form-label">What's on your mind?</label>
						<input 
							type="text" 
							class="form-input"
							placeholder="e.g. Learn Vue.js, Go for a run..."
							v-model="input_content"
							maxlength="100"
						/>
					</div>
					
					<div class="form-group">
						<label class="form-label">Choose a category</label>
						<div class="category-options">
							<label class="category-option" :class="{ active: input_category === 'business' }">
								<input 
									type="radio" 
									name="category" 
									value="business"
									v-model="input_category" 
								/>
								<div class="category-content">
									<span class="category-icon">💼</span>
									<span class="category-name">Business</span>
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
									<span class="category-icon">🎯</span>
									<span class="category-name">Personal</span>
								</div>
								<div class="category-indicator personal"></div>
							</label>
						</div>
					</div>
					
					<button type="submit" class="add-button" :disabled="!input_content.trim() || !input_category">
						<span class="button-icon">✨</span>
						Add Task
					</button>
				</form>
			</section>

			<!-- 待办事项列表 -->
			<section class="todo-list">
				<div class="section-header" v-if="todos.length > 0">
					<h3 class="section-title">
						<span class="section-icon">📋</span>
						Your Tasks
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
									<span class="check-icon">✓</span>
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
										{{ todo.category === 'business' ? '💼 Business' : '🎯 Personal' }}
									</span>
								</div>
							</div>

							<div class="todo-actions">
								<button 
									class="action-button delete" 
									@click="removeTodo(todo)"
									title="Delete task"
								>
									<span class="action-icon">🗑️</span>
								</button>
							</div>
						</div>
					</TransitionGroup>
				</div>

				<!-- 空状态 -->
				<div class="empty-state glass" v-else>
					<div class="empty-icon">🎉</div>
					<h3 class="empty-title">All caught up!</h3>
					<p class="empty-text">You have no tasks yet. Create one above to get started.</p>
				</div>
			</section>
		</main>
	</div>
</template>
