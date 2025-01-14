<template>
  <div class="todo-list h-screen">
    <h1 class="text-7xl text-white mb-5 font-bold text-center">
      Vue ToDo List
    </h1>
    <div class="todo-box">
      <!-- Task summary header -->

      <div
        class="todo-heading flex gap-3 items-center justify-end bg-gray-100 p-3"
      >
        <div class="tasks">
          <span class="bg-sky-600 font-bold rounded-2xl px-2 py-1 text-white">
            Tasks
            <span class="rounded-full bg-white text-sky-600 px-1.5">
              {{ tasks.length }}
            </span>
          </span>
        </div>
        <div class="tasks-done">
          <span class="bg-sky-600 font-bold rounded-2xl px-2 py-1 text-white">
            Tasks done
            <span class="rounded-full bg-white text-sky-600 px-1.5">
              {{ tasksDone }}
            </span>
          </span>
        </div>
        <!-- Delete buttons -->
        <div
          @click="deleteTasksDone"
          v-if="tasksDone > 0"
          class="delete-tasksDone bg-red-600 hover:bg-red-700 transition duration-200 p-1.5 cursor-pointer"
        >
          <font-awesome-icon :icon="['fas', 'trash']" class="text-white" />
          <button class="font-bold ml-3 text-white">Tasks Done</button>
        </div>
        <div
          @click="deleteTasks"
          v-if="tasks.length > 0"
          class="deleteTasks bg-red-600 p-1.5 hover:bg-red-700 transition duration-200 cursor-pointer"
        >
          <font-awesome-icon :icon="['fas', 'trash']" class="text-white" />
          <button class="font-bold ml-3 text-white">Tasks</button>
        </div>
      </div>

      <!-- Task list -->
      <div
        class="todo-body bg-white p-5 h-48 overflow-scroll overflow-x-hidden"
      >
        <h2
          v-if="tasks.length == 0"
          class="text-center mt-7 text-3xl text-gray-400"
        >
          Your task list is empty
        </h2>
        <transition-group name="fade" tag="div" class="task-list">
          <div
            v-for="(task, index) in tasks"
            :key="index"
            class="task mb-2 flex gap-3 hover:bg-gray-200 transition duration-400 bg-slate-50 rounded-md p-3 shadow-md outline outline-2 outline-gray-300"
          >
            <font-awesome-icon
              @click="toggleTask(task)"
              :icon="['fas', 'check']"
              class="text-white bg-gray-400 transition duration-200 p-1.5 text-sm rounded-full cursor-pointer"
              :class="{
                'bg-green-700 hover:bg-green-900': task.done,
                'hover:bg-gray-500': !task.done,
              }"
            />
            <p :class="{ 'line-through': task.done }">{{ task.text }}</p>
            <div class="hover-task hidden ml-auto text-xl mr-2 items-center">
              <font-awesome-icon
                @click="openEditOverlay(task)"
                :icon="['fas', 'pen-to-square']"
                class="text-sky-600 cursor-pointer mr-3"
              />
              <font-awesome-icon
                @click="deleteSingleTask(task)"
                :icon="['fas', 'trash']"
                class="text-red-600 cursor-pointer"
              />
            </div>
          </div>
        </transition-group>
      </div>

      <!-- Add task box -->
      <div class="add-task-box bg-gray-100 p-5 relative">
        <input
          id="addTask"
          name="addTask"
          @keydown.enter="addTask"
          type="text"
          v-model="newTask"
          placeholder="New task"
          class="w-full p-3 rounded-full outline-none outline-sky-600"
        />
        <client-only>
          <font-awesome-icon
            @click="addTask"
            :icon="['fas', 'plus']"
            class="text-2xl text-white bg-sky-600 rounded-full p-2 absolute right-8 mt-1 cursor-pointer hover:scale-110 transform transition duration-200"
          />
        </client-only>
      </div>
    </div>

    <!-- Overlay modal to edit tasks -->
    <div
      v-if="showOverlay"
      class="overlay absolute top-0 bottom-0 right-0 left-0 bg-slate-900 opacity-80"
    ></div>
    <div
      v-if="showOverlay"
      class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50"
    >
      <div class="bg-white relative rounded-lg shadow-lg p-6 w-11/12 max-w-md">
        <h1 class="text-2xl font-bold text-gray-800 mb-4">Edit Task</h1>
        <label
          for="taskName"
          class="block text-sm font-medium text-gray-700 mb-2"
        >
          Task Name
        </label>
        <input
          id="taskName"
          name="taskName"
          v-model="editTaskText"
          type="text"
          placeholder="Enter task name"
          class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-sky-500 focus:border-sky-500 mb-4"
        />
        <div class="flex justify-end space-x-3">
          <button
            @click="saveTask"
            class="bg-sky-600 hover:bg-sky-700 text-white font-medium px-4 py-2 rounded-lg transition duration-200"
          >
            <font-awesome-icon
              :icon="['fas', 'pen-to-square']"
              class="text-white cursor-pointer mr-3"
            />
            Save
          </button>
          <font-awesome-icon
            @click="closeOverlay"
            :icon="['far', 'circle-xmark']"
            class="text-4xl absolute top-5 cursor-pointer text-gray-600 hover:text-gray-800 transition duration-200"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.todo-list {
  max-width: 500px;
  margin: 0 auto;
}
/* Add task box button hover effect */
.add-task-box button:hover,
.delete-tasksDone:hover,
.deleteTasks:hover {
  transform: scale(1.1);
  transition: transform 0.3s ease;
}

/* Animation for entering/leaving tasks */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease, transform 0.5s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.task:hover .hover-task {
  display: block;
}
</style>

<script lang="ts">
import { defineComponent, ref, computed, watch, onMounted } from "vue";

interface Task {
  text: string;
  done: boolean;
}

export default defineComponent({
  name: "TodoList",
  setup() {
    const newTask = ref<string>(""); // For input field to add new task
    const tasks = ref<Task[]>([]); // Array to store tasks
    const showOverlay = ref<boolean>(false); // Controls modal visibility
    const editTaskText = ref<string>(""); // For editing the task
    const taskBeingEdited = ref<Task | null>(null); // Track the task being edited

    // Load tasks from localStorage on mount
    onMounted(() => {
      const storedTasks = localStorage.getItem("tasks");
      if (storedTasks) {
        tasks.value = JSON.parse(storedTasks);
      }
    });

    // Watch for changes in tasks and update localStorage
    watch(
      tasks,
      (newTasks: Task[]) => {
        localStorage.setItem("tasks", JSON.stringify(newTasks));
      },
      { deep: true }
    );

    // Computed property for tasks that are marked as done
    const tasksDone = computed(
      () => tasks.value.filter((task: { done: boolean }) => task.done).length
    );

    // Method to toggle a task's completion status
    const toggleTask = (task: Task): void => {
      task.done = !task.done;
    };

    // Method to add a new task
    const addTask = (): void => {
      if (newTask.value.trim()) {
        tasks.value.push({ text: newTask.value.trim(), done: false });
        newTask.value = "";
      }
    };

    // Method to delete all completed tasks
    const deleteTasksDone = (): void => {
      if (confirm("Are you sure you want to delete all completed tasks?")) {
        tasks.value = tasks.value.filter(
          (task: { done: boolean }) => !task.done
        );
      }
    };

    // Method to delete all tasks
    const deleteTasks = (): void => {
      if (confirm("Are you sure you want to delete all tasks?")) {
        tasks.value = [];
      }
    };

    // Method to open the overlay to edit a task
    const openEditOverlay = (task: Task): void => {
      editTaskText.value = task.text;
      taskBeingEdited.value = task; // Track the task being edited
      showOverlay.value = true;
    };

    // Method to save the edited task
    const saveTask = (): void => {
      // Ensure the input is not empty or just whitespace
      if (editTaskText.value.trim()) {
        // Update the task directly since we have a reference to it
        if (taskBeingEdited.value) {
          taskBeingEdited.value.text = editTaskText.value; // Update the task text
        }
        closeOverlay(); // Close the overlay/modal
      }
    };

    // Method to close the overlay without saving
    const closeOverlay = (): void => {
      showOverlay.value = false;
      editTaskText.value = "";
      taskBeingEdited.value = null; // Clear the reference
    };

    // Method to delete a single task
    const deleteSingleTask = (task: Task): void => {
      tasks.value = tasks.value.filter((t: Task) => t !== task);
    };

    return {
      newTask,
      tasks,
      tasksDone,
      toggleTask,
      addTask,
      deleteTasksDone,
      deleteTasks,
      openEditOverlay,
      saveTask,
      closeOverlay,
      editTaskText,
      showOverlay,
      deleteSingleTask,
    };
  },
});
</script>
