<script lang="ts">
	import { encryptTodos } from '$root/lib/util';
	import '$root/styles/todos.css';
	import type { FiltersType, Todos } from '$root/types/Todo';
	import { toast } from '@zerodevx/svelte-toast';
	import { Base64 } from 'js-base64';
	import forge from 'node-forge';
	import { fade } from 'svelte/transition';
	import type { PageData } from '../routes/$types';
	import AddTodo from './AddTodo.svelte';
	import Avatar from './Avatar.svelte';
	import TodoElement from './TodoElement.svelte';

	export let initialTodos: Todos[];

	export let data: PageData;

	let selectedFilter: FiltersType = 'all';

	let todos = initialTodos.map((a) => {
		return { ...a };
	});

	let public_key = forge.pki
		.publicKeyFromPem(Base64.decode(localStorage.getItem('public_key') as string))
		.n.toString(16);

	$: todosAmount = todos.length;
	$: incompleteTodos = todos.filter((todo) => !todo.completed).length;
	$: filteredTodos = filterTodos(todos, selectedFilter);
	$: completedTodos = todos.filter((todo) => todo.completed).length;
	$: pushTasksToDB(todos);

	async function pushTasksToDB(newTodos: Todos[]) {
		if (newTodos != initialTodos) {
			let encrypted = encryptTodos(Base64.decode(localStorage.getItem('dek')!), newTodos);

			let res = await fetch('/api/tasks', {
				method: 'POST',
				headers: {
					Accept: 'application/json',
					'Content-Type': 'application/json'
				},
				body: encrypted,
				credentials: 'same-origin'
			});

			console.log(await res.text());
			if (res.status == 500) {
				toast.push('Error while updating Todos. Try adding another one or reload page.');
			}
		}
	}

	function generateRandomId(): string {
		return Math.random().toString(16).slice(2);
	}

	async function fetchTodos() {
		// TODO: Implement
		/* await updateTodosFromServer(initialTodos, Base64.decode(localStorage.getItem('dek')!));

		todos = initialTodos; */
	}

	async function addTodo(todo: string) {
		await fetchTodos();

		let newTodo: Todos = {
			id: generateRandomId(),
			text: todo,
			completed: false
		};
		todos = [...todos, newTodo];
	}

	async function toggleCompleted(event: MouseEvent) {
		await fetchTodos();
		let { checked } = event.target as HTMLInputElement;

		todos = todos.map((todo) => ({
			...todo,
			completed: checked
		}));
	}

	async function completeTodo(id: string) {
		await fetchTodos();
		todos = todos.map((todo) => {
			if (todo.id === id) {
				todo.completed = !todo.completed;
			}
			return todo;
		});
	}

	async function removeTodo(id: string) {
		await fetchTodos();
		todos = todos.filter((todo) => todo.id !== id);
	}

	async function editTodo(id: string, newTodo: string) {
		await fetchTodos();
		let currentTodo = todos.findIndex((todo) => todo.id === id);
		todos[currentTodo].text = newTodo;
	}

	function setFilter(newFilter: any): void {
		selectedFilter = newFilter;
	}

	function filterTodos(todos: Todos[], filter: any): Todos[] {
		switch (filter) {
			default:
				return todos;
			case 'active':
				return todos.filter((todo) => !todo.completed);
			case 'completed':
				return todos.filter((todo) => todo.completed);
		}
	}

	let filters = ['all', 'active', 'completed'];

	function clearCompleted(): void {
		todos = todos.filter((todo) => todo.completed !== true);
	}
</script>

<main in:fade={{ duration: 1000 }}>
	<Avatar
		client_id={data.client_id}
		public_key={'0x' +
			public_key.slice(0, 5) +
			'...' +
			public_key.slice(public_key.length - 3, public_key.length)}
	/>
	<div class="todos-container">
		<h1 class="title">to-dos ????</h1>
		<div class="todos">
			<AddTodo {addTodo} {toggleCompleted} {todosAmount} />
			{#if todosAmount}
				<ul class="todo-list">
					{#each filteredTodos as todo (todo.id)}
						<TodoElement {removeTodo} {completeTodo} {todo} {editTodo} />
					{/each}
				</ul>

				<div class="actions">
					<span class="todo-count">
						{incompleteTodos}
						{incompleteTodos === 1 ? 'item' : 'items'} left</span
					>
					<div class="filters">
						{#each filters as filter}
							<button
								on:click={() => setFilter(filter)}
								class:selected={selectedFilter === filter}
								class="filter"
							>
								{filter}
							</button>
						{/each}
					</div>
					<button
						on:click={clearCompleted}
						class:hidden={completedTodos === 0}
						class="clear-completed">Clear completed</button
					>
				</div>
			{/if}
		</div>
	</div>
</main>
