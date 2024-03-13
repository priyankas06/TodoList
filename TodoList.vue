<template>
  <div class="todo-list">
    <h1>Todos</h1>
    <div class="input-with-button">
      <input
        type="text"
        v-model="todoName"
        @keyup.enter="addTodo"
        aria-label="Add a new Todo"
        placeholder="Add a new Todo"
        class="todo-input"
      />
      <button @click="addTodo">Submit</button>
    </div>
    <div class="todo-columns">
      <div class="todo-column incomplete">
        <h2>Incomplete</h2>
        <ul class="todo-items">
          <li v-for="todo of incompleteTodos" 
              :key="todo.id"
              @click="toggleTodoStatus(todo)"
              class="todo-item">
            <span class="todo-name">{{ todo.name }}</span>
            <span class="created-at">{{ todo.createdAt }}</span>
            <button @click="deleteTodo(todo.id)">del</button>
          </li>
        </ul>
      </div>
      <div class="todo-column completed">
        <h2>Completed</h2>
        <ul class="todo-items">
          <li v-for="todo of completedTodos" 
              :key="todo.id"
               @click="toggleTodoStatus(todo)"
              class="todo-item done">
            <span class="todo-name">{{ todo.name }}</span>
            <span class="created-at">{{ todo.createdAt }}</span>
            <button @click="deleteTodo(todo.id)">del</button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
const baseURL = "http://localhost:3001/todos";

export default {
  name: "TodoList",
  data() {
    return {
      todos: [],
      todoName: ""
    };
  },
  async created() {
    try {
      const res = await axios.get(baseURL);
      this.todos = res.data;
      
    } catch (e) {
      console.error(e);
    }
  },
  computed: {
    incompleteTodos() {
      return this.todos.filter(todo => !todo.done);
    },
    completedTodos() {
      return this.todos.filter(todo => todo.done);
    }
  },
  methods: {
    async toggleTodoStatus(todo) {
      try {
        const updatedTodo = { ...todo, done: !todo.done };
        await axios.patch(`${baseURL}/${todo.id}`, updatedTodo);
        const index = this.todos.findIndex(item => item.id === todo.id);
        this.todos.splice(index, 1, updatedTodo);
      } catch (e) {
        console.error(e);
      }
    },
    async addTodo() {
      try {
        const currentDate = new Date();
        const formattedDate = currentDate.toLocaleDateString();
        const formattedTime = currentDate.toLocaleTimeString();
        const res = await axios.post(baseURL, {
          name: this.todoName,
          createdAt: `${formattedDate} ${formattedTime}`
        });
        this.todos = [...this.todos, res.data];
        this.todoName = "";
      } catch (e) {
        console.error(e);
      }
    },
    async deleteTodo(todoId) {
      try {
        await axios.delete(`${baseURL}/${todoId}`);
        this.todos = this.todos.filter(todo => todo.id !== todoId);
      } catch (e) {
        console.error(e);
      }
    }
  }
};
</script>

<style scoped>
.todo-list {
  width: 1000px; 
  margin: 0 auto;
}

.todo-input {
  width: calc(100% - 90px);
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 0.25rem;
  font-size: 1rem;
  margin-right: 1rem;
  margin-bottom:1rem;
}

.input-with-button {
  display: flex;
  align-items: center;
}

button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.25rem;
  background-color: black;
  color: #fff;
  cursor: pointer;
}

.todo-columns {
  display: flex;
  justify-content: space-between;
}

.todo-column {
  flex: 1;
  border: 1px solid black;
  border-radius: 0.25rem;
  padding: 1rem;
  max-width: 45%; 
}

.todo-column.incomplete {
  margin-right: 1rem;
}

.todo-column.completed {
  margin-left: 1rem;
}

.todo-column h2 {
  font-size: 1rem;
}

.todo-items {
  list-style: none;
  padding: 0;
}

.todo-item {
  padding: 0.1rem;
  cursor: pointer;
  display:flex;
  align-items:center;
}
.todo-item button {
  padding: 0.2rem;
  margin-left: 10rem;
}

.todo-item.done {
  text-decoration: line-through;
}

.todo-name {
  font-size: 0.9rem;
  margin-right: 5rem;
}

.created-at {
  font-size: 0.8rem;
  color: black;
}
</style>