<!-- <script setup lang="ts">
import { ref } from 'vue';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faBars, faX } from '@fortawesome/free-solid-svg-icons';

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
const dropIndex = ref<number | null>(null)
const dragIndex = ref<number | null>(null)
const currentTarget = ref<HTMLElement | null>(null)
const OFFSET = 1;

const addTodo = () => {
  if (newTodo.value.trim() !== '') {
    todos.value.push({ id: Date.now(), text: newTodo.value.trim(), completed: false });
    newTodo.value = '';
  }
}

const removeTodo = (id: number) => {
  todos.value = todos.value.filter(todo => todo.id !== id);
};

const dragStart = (id: number, e: DragEvent) => {
  dragIndex.value = id
  dropIndex.value = id + OFFSET // default to current slot
  e.dataTransfer?.setData('text/plain', String(id)) // Firefox needs data set
  e.dataTransfer!.effectAllowed = 'move'
  const el = e.target as HTMLElement;
  el.style.opacity = '0.5'
  currentTarget.value = el
  console.log("opacity set to 0.5");
}

const drop = () => {
  if (dragIndex.value == null || dropIndex.value == null) return
  const from = dragIndex.value
  let to = dropIndex.value - OFFSET

  // If you drag down past yourself, removing first shifts the target left by 1
  if (to > from) to -= 1

  if (from !== to) {
    const next = [...todos.value]
    const [moved] = next.splice(from, 1)
    next.splice(to, 0, moved)
    todos.value = next
  }
  if (currentTarget.value) currentTarget.value.style.opacity = '1'
  dragIndex.value = null
  dropIndex.value = null
}

const dragOverItem = (index: number, e: DragEvent) => {
  // Allow dropping
  e.preventDefault()
  e.stopPropagation()

  // Determine if cursor is in the top or bottom half of the item
  const target = e.currentTarget as HTMLElement
  const rect = target.getBoundingClientRect()
  const before = e.clientY < rect.top + rect.height / 2

  dropIndex.value = (before ? index : index + 1) + OFFSET
}

const dragOverList = (e: DragEvent) => {
  e.preventDefault()
  // Only when actually below the final item (not just any UL gap)
  const items = Array.from((e.currentTarget as HTMLElement).querySelectorAll('li[data-item]')) as HTMLElement[]
  const last = items[items.length - 1]
  if (!last) return
  const lastBottom = last.getBoundingClientRect().bottom
  if (e.clientY > lastBottom) {
    dropIndex.value = todos.value.length + OFFSET
  }
}

const handleOutsideDrop = () => {
  dropIndex.value = null
  dragIndex.value = null
  if (currentTarget.value) currentTarget.value.style.opacity = '1'
}
</script>

<template>
  <div class="flex flex-col gap-4">
    <h1>ToDo List:</h1>
    <div>
      <input class="border p-2 rounded-lg w-2/3" @keyup.enter="addTodo" v-model="newTodo"
        placeholder="Add a new task..." />
    </div>
    <div>
      <ul @dragover="dragOverList" @drop.prevent.stop="drop" class="flex flex-col gap-2 py-4">

        <li class="drop-line"
          aria-hidden="true"
          @dragover.prevent.stop="dropIndex = 0 + OFFSET"></li>

        <li class="flex justify-between items-center border p-2 rounded-lg bg-white/60 drag:opacity-100"
          v-for="(todo, index) in todos" 
          :key="todo.id" draggable="true" 
          @dragstart="dragStart(index, $event)"
          @dragover.stop="dragOverItem(index, $event)" 
          @drop.stop="drop"
          @dragend="handleOutsideDrop()"
          @dragenter.stop="dragOverItem(index, $event)">

          <div class="flex items-center">
            <input type="checkbox" class="mr-2" v-model="todo.completed" />
            <span :class="{ 'line-through': todo.completed }">{{ todo.text }}</span>
          </div>
          <div v-if="todo.completed === false" class="flex gap-4">
            <div>
              <FontAwesomeIcon class="cursor-pointer" :icon="faX" @click="removeTodo(todo.id)" />
            </div>
            <div>
              <FontAwesomeIcon class="cursor-move" :icon="faBars" @dragstart="dragStart(index, $event)" />
            </div>
          </div>
        </li>

        <li class="drop-line"
          aria-hidden="true"
          @dragover.prevent.stop="dropIndex = todos.length + OFFSET"></li>
      </ul>
    </div>
  </div>
</template> -->
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