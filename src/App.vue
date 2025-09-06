<script setup>
import { ref, onMounted, computed, watch } from 'vue'
//使用ref创建响应式数据
const todos = ref([])
const name = ref('')
//新待办事项的内容和类别
const input_content = ref('')
const input_category = ref(null)
//监听name的变化，保存到localStorage
watch(name, (newVal) => {
	localStorage.setItem('name', newVal)
})
//监听todos的变化，保存到localStorage
watch(todos, (newVal) => {
	localStorage.setItem('todos', JSON.stringify(newVal))
}, {
	deep: true //深度监听
})
//添加新待办事项的函数
const addTodo = () => {
	//验证输入是否有效
	if (input_content.value.trim() === '' || input_category.value === null) {
		return
	}
    //将新待办事项添加到列表
	todos.value.push({
		content: input_content.value,
		category: input_category.value,
		done: false,
		editable: false,
		createdAt: new Date().getTime()
	})
	// 清空输入字段
    input_content.value = '';
    input_category.value = null;
}
//按创建时间升序排列待办事项
const todos_asc=computed(()=>todos.value.sort((a,b)=>{
	return a.createdAt-b.createdAt
}))
//删除待办事项的函数
const removeTodo=(todo)=>{
    todos.value=todos.value.filter((t)=>t!==todo)
}
//组件加载时可从localStorage加载数据
onMounted(()=>{
	name.value=localStorage.getItem('name')||''
	todos.value=JSON.parse(localStorage.getItem('todos'))||[]
})

</script>

<template>
	<main class="app">
		<section class="greeting">
			<h2 class="title">
				What's up, <input type="text" id="name" placeholder="Name here" v-model="name">
			</h2>
		</section>
		<section class="create-todo">
			<h3>CREATE A TODO</h3>
			<form id="new-todo-form" @submit.prevent="addTodo">
				<h4>What's on your todo list?</h4>
				<input 
					type="text" 
					name="content" 
					id="content" 
					placeholder="e.g. make a video"
					v-model="input_content" />
				<h4>Pick a category</h4>
				<div class="options">
					<label>
						<input 
							type="radio" 
							name="category" 
							id="category1" 
							value="business"
							v-model="input_category" />
						<span class="bubble business"></span>
						<div>Business</div>
					</label>
					<label>
						<input 
							type="radio" 
							name="category" 
							id="category2" 
							value="personal"
							v-model="input_category" />
						<span class="bubble personal"></span>
						<div>Personal</div>
					</label>
				</div>
				<input type="submit" value="Add todo" />
			</form>
		</section>
        <section class="todo-list">
			<h3>TODO LIST</h3>
			<div class="list" id="todo-list">

				<div v-for="todo in todos_asc" :class="`todo-item ${todo.done && 'done'}`">
					<label>
						<input type="checkbox" v-model="todo.done" />
						<span :class="`bubble ${
							todo.category == 'business' 
								? 'business' 
								: 'personal'
						}`"></span>
					</label>

					<div class="todo-content">
						<input type="text" v-model="todo.content" />
					</div>

					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Delete</button>
					</div>
				</div>

			</div>
		</section>

		</main>
</template>
