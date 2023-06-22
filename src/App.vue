<script>
  import StatusFilter from './components/StatusFilter.vue'
  import TodoItem from './components/TodoItem.vue';
  import Message from './components/Message.vue';
  import { getTodos, createTodo, updateTodo, deleteTodo } from './api/todos';

  export default {
    components: {
    StatusFilter,
    TodoItem,
    Message,
},

    data() {
      return {
        todos: [],
        title: '',
        status: 'all',
        errorMessage: '',
      }
    },
    mounted() {
      getTodos()
        .then(({ data }) => 
          this.todos = data
        )
        .catch(() => {
          this.errorMessage = 'Unable to load todos'
        });
    },
    computed: {
      activeTodos() {
        return this.todos.filter((todo => !todo.completed))
      },
      completedTodos() {
        return this.todos.filter((todo => todo.completed))
      },
      visibleTodos() {
        switch (this.status) {
          case 'active':
            return this.activeTodos;

          case 'completed':
            return this.completedTodos;
          
          default:
            return this.todos;
        }
      }
    },
/*     watch: {
      todos: {
        deep: true,
        handler() {
        localStorage.setItem('todos', JSON.stringify(this.todos))
        },
      }
    }, */
    methods: {
      handleSubmit() {
        createTodo(this.title)
          .then(({data}) => {
            this.todos.push(data);
            this.title = '';
          })
      },
      updateTodo({id, title, completed}) {
        updateTodo({id, title, completed})
          .then(({data}) => {
            this.todos = this.todos.map(todo => 
              todo.id !== id
                ? todo
                : data
            )
          })
      },
      deleteTodo(todoId) {
        deleteTodo(todoId)
          .then(() => {
            this.todos = this.todos
              .filter(todo => todo.id !== todoId);
          });
      }
    },
  }
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button 
          class="todoapp__toggle-all"
          :class="{active: activeTodos.length === 0}"
        ></button>

        <form @submit.prevent="handleSubmit">
          <input
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
          />
        </form>
      </header>

      <TransitionGroup 
        name="list" 
        class="todoapp__main" 
        tag="section"
      >
        <TodoItem 
          v-for="todo, index of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @update="updateTodo"
          @delete="deleteTodo(todo.id)"
        />

      </TransitionGroup>

      <footer class="todoapp__footer">
        <span class="todo-count">
          {{ activeTodos.length }} items left
        </span>

        <StatusFilter 
          v-model="status"
        />
        
        <button class="todoapp__clear-completed" v-if="activeTodos.length > 0">
          Clear completed
        </button>
      </footer>
    </div>

    <Message 
      class="is-warning"
      :active="errorMessage !== ''"
      @hide="errorMessage = ''"
    >
      <template #default>
        <p>{{ errorMessage }}</p>
      </template>

      <template #header>
        <p>Server Error</p>
      </template>
    </Message>
  </div>
</template>

<style>
  .list-enter-active,
  .list-leave-active {
    max-height: 60px;
    transition: all 0.5s ease;
  }
  .list-enter-from,
  .list-leave-to {
    opacity: 0;
    max-height: 0px;
    transform: scaleY(0);
  }
</style>