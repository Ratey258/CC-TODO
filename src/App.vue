<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import Icons from './components/Icons.vue'

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
						<div class="stat-card">
							<div class="stat-number">{{ totalTodos }}</div>
							<div class="stat-label">总任务</div>
						</div>
						<div class="stat-card">
							<div class="stat-number">{{ completedTodos }}</div>
							<div class="stat-label">已完成</div>
						</div>
						<div class="stat-card">
							<div class="stat-number">{{ progressPercentage }}%</div>
							<div class="stat-label">进度</div>
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
						<span class="section-icon">
							<Icons name="plus" />
						</span>
						创建新任务
					</h3>
				</div>
				
				<form @submit.prevent="addTodo" class="todo-form">
					<div class="form-group">
						<label class="form-label">你在想什么？</label>
						<input 
							type="text" 
							class="form-input"
							placeholder="例如：学习Vue.js，去跑步..."
							v-model="input_content"
							maxlength="100"
						/>
					</div>
					
					<div class="form-group">
						<label class="form-label">选择分类</label>
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
									<span class="category-name">工作</span>
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
									<span class="category-name">个人</span>
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
	</div>
</template>
