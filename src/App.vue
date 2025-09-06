<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import Icons from './components/Icons.vue'

// 响应式数据
const todos = ref([])
const name = ref('')
const input_content = ref('')
const input_category = ref(null)
const isDarkMode = ref(false)
const showDeleteModal = ref(false)
const todoToDelete = ref(null)
const showLogoutModal = ref(false)

// 用户相关状态
const currentUser = ref(null)
const showLoginModal = ref(false)
const loginForm = ref({
	username: ''
})
const loginError = ref('')

// 监听主题变化
watch(isDarkMode, (newVal) => {
	localStorage.setItem('darkMode', JSON.stringify(newVal))
	document.documentElement.setAttribute('data-theme', newVal ? 'dark' : 'light')
})

// 监听name的变化，保存到用户数据
watch(name, (newVal) => {
	if (currentUser.value) {
		saveUserData()
	}
})

// 监听todos的变化，保存到用户数据
watch(todos, (newVal) => {
	if (currentUser.value) {
		saveUserData()
	}
}, {
	deep: true
})

// 切换主题
const toggleTheme = () => {
	isDarkMode.value = !isDarkMode.value
}

// 获取欢迎消息
const getWelcomeMessage = () => {
	const messages = [
		'今天也要加油哦！✨',
		'让我们一起完成今天的目标吧！🎯',
		'每一个小步骤都是进步！🌟',
		'保持专注，你可以做到的！💪',
		'今天是充满可能性的一天！🚀',
		'一步一个脚印，稳步前进！🌈',
		'相信自己，你很棒！⭐',
		'新的一天，新的开始！🌅'
	]
	
	const hour = new Date().getHours()
	if (hour < 6) {
		return '夜深了，记得早点休息哦！🌙'
	} else if (hour < 12) {
		return messages[Math.floor(Math.random() * messages.length)]
	} else if (hour < 18) {
		return '午后时光，继续保持动力！☀️'
	} else {
		return '晚上好！今天过得怎么样？🌆'
	}
}

// 用户管理函数
const getUsers = () => {
	return JSON.parse(localStorage.getItem('users')) || {}
}

const saveUserData = () => {
	if (!currentUser.value) return
	
	const users = getUsers()
	users[currentUser.value.username] = {
		...currentUser.value,
		name: name.value,
		todos: todos.value,
		lastLogin: new Date().getTime()
	}
	localStorage.setItem('users', JSON.stringify(users))
}

const loadUserData = (username) => {
	const users = getUsers()
	const userData = users[username]
	if (userData) {
		name.value = userData.name || ''
		todos.value = userData.todos || []
	}
}

const login = () => {
	loginError.value = ''
	const { username } = loginForm.value
	
	if (!username.trim()) {
		loginError.value = '请输入用户名'
		return
	}
	
	const users = getUsers()
	
	// 如果用户不存在，自动创建新用户
	if (!users[username]) {
		users[username] = {
			username: username,
			name: '',
			todos: [],
			createdAt: new Date().getTime(),
			lastLogin: new Date().getTime()
		}
		localStorage.setItem('users', JSON.stringify(users))
	}
	
	// 登录用户
	currentUser.value = users[username]
	localStorage.setItem('currentUser', username)
	loadUserData(username)
	resetLoginForm()
	showLoginModal.value = false
}

// 显示退出确认模态框
const showLogoutConfirmation = () => {
	showLogoutModal.value = true
}

// 确认退出
const confirmLogout = () => {
	if (currentUser.value) {
		saveUserData() // 保存当前数据
	}
	currentUser.value = null
	name.value = ''
	todos.value = []
	localStorage.removeItem('currentUser')
	showLogoutModal.value = false
	showLoginModal.value = true
}

// 取消退出
const cancelLogout = () => {
	showLogoutModal.value = false
}

// 原来的logout函数保留备用
const logout = () => {
	if (currentUser.value) {
		saveUserData() // 保存当前数据
	}
	currentUser.value = null
	name.value = ''
	todos.value = []
	localStorage.removeItem('currentUser')
	showLoginModal.value = true
}

const resetLoginForm = () => {
	loginForm.value = {
		username: ''
	}
	loginError.value = ''
}


const showLogin = () => {
	resetLoginForm()
	showLoginModal.value = true
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

// 显示删除确认模态框
const showDeleteConfirmation = (todo) => {
	todoToDelete.value = todo
	showDeleteModal.value = true
}

// 确认删除待办事项
const confirmDelete = () => {
	if (todoToDelete.value) {
		todos.value = todos.value.filter((t) => t !== todoToDelete.value)
	}
	cancelDelete()
}

// 取消删除
const cancelDelete = () => {
	showDeleteModal.value = false
	todoToDelete.value = null
}

// 删除待办事项的函数（保留以防万一，但现在不直接使用）
const removeTodo = (todo) => {
	todos.value = todos.value.filter((t) => t !== todo)
}

// 自动调整文本框高度
const autoResize = (event) => {
	const textarea = event.target
	textarea.style.height = 'auto'
	textarea.style.height = textarea.scrollHeight + 'px'
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
	// 加载主题设置
	isDarkMode.value = JSON.parse(localStorage.getItem('darkMode')) || false
	document.documentElement.setAttribute('data-theme', isDarkMode.value ? 'dark' : 'light')
	
	// 尝试恢复用户登录状态
	const lastUser = localStorage.getItem('currentUser')
	if (lastUser) {
		const users = getUsers()
		if (users[lastUser]) {
			currentUser.value = users[lastUser]
			loadUserData(lastUser)
		} else {
			// 用户数据不存在，清除状态
			localStorage.removeItem('currentUser')
			showLoginModal.value = true
		}
	} else {
		// 没有登录用户，显示登录模态框
		showLoginModal.value = true
	}
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
				<div class="nav-actions">
					<div class="user-info" v-if="currentUser">
						<span class="user-avatar">
							<Icons name="user" />
						</span>
						<span class="username">{{ currentUser.username }}</span>
					</div>
					<button class="theme-toggle" @click="toggleTheme" :class="{ active: isDarkMode }">
						<span class="theme-icon">
							<Icons :name="isDarkMode ? 'sun' : 'moon'" />
						</span>
					</button>
					<button class="logout-button" @click="showLogoutConfirmation" v-if="currentUser" title="退出登录">
						<span class="logout-icon">
							<Icons name="log-out" />
						</span>
					</button>
				</div>
			</div>
		</nav>

		<main class="main-content">
			<!-- 问候区域 -->
			<section class="greeting glass">
				<div class="greeting-content">
					<div class="welcome-header">
						<h2 class="greeting-title">
							<span class="greeting-text">欢迎回来，</span>
							<span class="user-name">{{ currentUser?.username || '用户' }}</span>
							<span class="wave">👋</span>
						</h2>
						<p class="welcome-message">{{ getWelcomeMessage() }}</p>
					</div>
					
					<!-- 进度统计 -->
					<div class="stats-grid">
						<div class="stat-card">
							<div class="stat-number">{{ totalTodos }}</div>
							<div class="stat-label">总待办事项</div>
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
						创建新待办事项
					</h3>
				</div>
				
				<form @submit.prevent="addTodo" class="todo-form">
					<div class="form-group">
						<label class="form-label">你想做什么？</label>
						<input 
							type="text" 
							class="form-input"
							placeholder="例如：去跑步..."
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
						添加待办事项
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
						我的待办事项
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
								<textarea 
									v-model="todo.content" 
									class="todo-text"
									maxlength="100"
									@input="autoResize"
									rows="1"
								></textarea>
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
										@click="showDeleteConfirmation(todo)"
										title="删除待办事项"
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
					<h3 class="empty-title">全部完成！</h3>
					<p class="empty-text">您暂时没有待办事项。在上方创建一个待办事项开始吧。</p>
				</div>
			</section>
		</main>

		<!-- 删除确认模态框 -->
		<Transition name="modal">
			<div v-if="showDeleteModal" class="modal-overlay" @click="cancelDelete">
				<div class="delete-modal" @click.stop>
					<div class="modal-header">
						<div class="modal-icon">
							<span class="warning-icon">
								<Icons name="alert-triangle" />
							</span>
						</div>
						<h3 class="modal-title">确认删除待办事项</h3>
					</div>
					
					<div class="modal-content">
						<p class="modal-message">
							确定要删除这个待办事项吗？此操作无法撤销。
						</p>
						
						<div class="modal-actions">
							<button 
								class="modal-button cancel" 
								@click="cancelDelete"
								type="button"
							>
								<span class="button-icon">
									<Icons name="x" />
								</span>
								取消
							</button>
							
							<button 
								class="modal-button confirm" 
								@click="confirmDelete"
								type="button"
							>
								<span class="button-icon">
									<Icons name="trash" />
								</span>
								删除
							</button>
						</div>
					</div>
				</div>
			</div>
		</Transition>

		<!-- 退出确认模态框 -->
		<Transition name="modal">
			<div v-if="showLogoutModal" class="modal-overlay" @click="cancelLogout">
				<div class="logout-modal" @click.stop>
					<div class="modal-header">
						<div class="modal-icon">
							<span class="warning-icon">
								<Icons name="log-out" />
							</span>
						</div>
						<h3 class="modal-title">确认退出登录</h3>
					</div>
					
					<div class="modal-content">
						<p class="modal-message">
							确定要退出登录吗？您的待办事项已自动保存。
						</p>
						
						<div class="modal-actions">
							<button 
								class="modal-button cancel" 
								@click="cancelLogout"
								type="button"
							>
								<span class="button-icon">
									<Icons name="x" />
								</span>
								取消
							</button>
							
							<button 
								class="modal-button confirm" 
								@click="confirmLogout"
								type="button"
							>
								<span class="button-icon">
									<Icons name="log-out" />
								</span>
								退出
							</button>
						</div>
					</div>
				</div>
			</div>
		</Transition>

		<!-- 登录模态框 -->
		<Transition name="modal">
			<div v-if="showLoginModal" class="modal-overlay">
				<div class="login-modal" @click.stop>
					<div class="modal-header">
						<div class="modal-icon">
							<span class="login-icon">
								<Icons name="user" />
							</span>
						</div>
						<h3 class="modal-title">欢迎使用 CC-TODO</h3>
						<p class="modal-subtitle">请输入您的用户名开始使用</p>
					</div>
					
					<div class="modal-content">
						<form @submit.prevent="login" class="login-form">
							<div class="form-group">
								<label for="username" class="form-label">用户名</label>
								<div class="input-wrapper">
									<span class="input-icon">
										<Icons name="user" />
									</span>
									<input
										id="username"
										v-model="loginForm.username"
										type="text"
										class="form-input"
										placeholder="请输入用户名"
										autocomplete="username"
										required
									>
								</div>
							</div>
							
							<div v-if="loginError" class="error-message">
								<span class="error-icon">
									<Icons name="alert-circle" />
								</span>
								{{ loginError }}
							</div>
							
							<div class="form-actions">
								<button 
									type="submit"
									class="login-button"
									:disabled="!loginForm.username.trim()"
								>
									<span class="button-icon">
										<Icons name="log-in" />
									</span>
									开始使用
								</button>
							</div>
						</form>
					</div>
				</div>
			</div>
		</Transition>
	</div>
</template>
