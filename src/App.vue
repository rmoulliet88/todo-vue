<script setup lang="ts">
import { computed, ref } from 'vue';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faBars, faX } from '@fortawesome/free-solid-svg-icons';
import { VueDraggable } from 'vue-draggable-plus'


interface Todo {
  id: number;
  text: string;
  completed: boolean;
  dateCompleted?: Date;
  dueDate?: Date;
}

const todos = ref<Todo[]>([
  { id: 1, text: 'Learn Vue 3', completed: false },
  { id: 2, text: 'Build a ToDo App', completed: false },
  { id: 3, text: 'Master TypeScript', completed: false }
]);

const newTodo = ref('');
const todoSorted = computed(() => {
  return [...todos.value].sort((a, b) => Number(a.completed) - Number(b.completed));
})

const addTodo = () => {
  if (newTodo.value.trim() !== '') {
    todos.value.push({ id: Date.now(), text: newTodo.value.trim(), completed: false });
    newTodo.value = '';
  }
}

const removeTodo = (id: number) => {
  todos.value = todos.value.filter(todo => todo.id !== id);
};

</script>

<template>
  <div class="flex flex-col gap-4">
    <h1 class="text-2xl font-bold mb-4">ToDo List:</h1>
    <div>
      <input class="border p-2 rounded-lg w-2/3" @keyup.enter="addTodo" v-model="newTodo"
        placeholder="Add a new task..." />
    </div>
    <div>
      <VueDraggable 
        v-model="todos" 
        :animation="200" 
        handle=".drag-handle" 
        tag="ul" 
        class="flex flex-col gap-2"
        ghost-class="drag-ghost"
        chosen-class="drag-chosen">

        <li v-for="todo in todoSorted" :key="todo.id"
          class="flex justify-between items-center border p-2 rounded-lg bg-white/60">
          <div class="flex items-center">
            <input type="checkbox" v-model="todo.completed" class="mr-2" />
            <span :class="{ 'line-through': todo.completed }">{{ todo.text }}</span>
          </div>
          <div v-if="todo.completed === false" class="flex gap-4">
            <div>
              <FontAwesomeIcon class="cursor-pointer" :icon="faX" @click="removeTodo(todo.id)" />
            </div>
            <div class="cursor-move drag-handle">
              <FontAwesomeIcon :icon="faBars" />
            </div>
          </div>
        </li>
      </VueDraggable>
    </div>
  </div>
</template>
<style scoped>
.drag-ghost {
  opacity: 0.4;
  background: #6c4ab6;
  color: white;
}

.drag-chosen {
  border: 2px solid #6c4ab6;  /* highlight picked-up item */
}
</style>