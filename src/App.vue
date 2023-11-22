<template>
  <div  class="contain mt-5 ">
      <h2 class="text-center text-col1">November 2023 </h2>
      <h1 class="text-center mb-4 text-col"><b>My To Do List</b></h1>
      <!-- Input field to add new task -->
      <div class="mb-3">
        <input v-model="newTask" type="text" class="form-control enter" placeholder="Insert new task here">
        <div class="flex-1">
        <button @click="addTask" class="mt-1 button button-mat btn--7 pseudo-text cursor">
          <span class="pseudo-text ">Add Task</span>
        </button>
        </div>
      </div>
      <p class="mb-3 num"> Number of incomplete tasks: <b>{{ incompleteTasks }}</b> </p>
      <p class="mb-3 num1">
        Click the toogle when a task is completed
      </p>
      <!-- List of tasks -->
      <ul class="list-group">
        <li v-for="(task, index) in tasks" :key="index" class="form-control1 list-group-item d-flex justify-content-between align-items-center enter">
          <span @click="showModal"
          data-bs-toggle="modal" 
          data-bs-target="#editModal">{{ task.title }}</span>
          <span>{{ task.content }}</span>
          
          <div style="padding-left: 150px;">
            <button @click="showModal"
            data-bs-toggle="modal" 
            data-bs-target="#openEditModal" class="btn btn-sm btn-primary m-1 cursor btn-edit">
              <i class="fas fa-pencil-alt"> Edit</i>
            </button>
            <label class="toggle-switch">
              <input type="checkbox" v-model="task.completed" @change="toggleTaskCompletion(task)">
              <span class="slider" @click="showEditModal(task.completed, task)"></span>
            </label>
      
            <button @click="deleteTask(index)" class="btn btn-sm btn-danger cursor btn-delete">
              <i class="fas fa-trash-alt"> Delete</i>
            </button>
          </div>
      
        </li>
      </ul>
 
      
    <b-modal id="showEditModal" ref="my-modal" title="Edit Task">
        <template #modal-body>
          <input v-model="editedTask.title" type="text" class="form-control mb-3" placeholder="Title">
          <textarea v-model="editedTask.content" class="form-control" rows="4" placeholder="Content"></textarea>
        </template>
        <template #modal-footer="{ cancel, ok }">
          <b-button @click="saveTask" variant="primary">Save changes</b-button>
          <b-button @click="cancel">Close</b-button>
        </template>
      </b-modal>
 </div>

</template>

<script>
export default {
  data() {
    return {
        task: {
        completed: false
      },
      newTask: '',
      tasks: [],
      showEditModal: false,
      editedTask: { title: '', content: '', index: -1 },
    };
  },
  
  computed: {
    incompleteTasks() {
      return this.tasks.filter(task => !task.completed).length;
    }
  },
  methods: {
    getSavedTasks() {
      const savedTasks = localStorage.getItem('tasks');
      if (savedTasks) {
        this.tasks = JSON.parse(savedTasks);
      }
    },
    showModal() {
        this.$refs['my-modal'].show()
      },


    openEditModal() {
      this.showEditModal = true;
    },
    saveTasks() {
      this.showEditModal = false;
      localStorage.setItem('tasks', JSON.stringify(this.tasks));
    },
    cancel() {
      this.showEditModal = false;
    },
    addTask() {
      if (this.newTask.trim() !== '') {
        this.tasks.push({ title: this.newTask, content: '', completed: false });
        this.newTask = '';
        this.saveTasks(); 
      }
    },
    deleteTask(index) {
      this.tasks.splice(index, 1);
      this.saveTasks(); 
    },
    async toggleTaskCompletion(task) {
      if (task.completed) {
        console.log('Cette tâche est déjà marquée comme terminée');
        return;
      }

      try {
        const newCompletionState = !task.completed;
        await axios.put(`http://api/tasks/${task.id}`, { completed: newCompletionState });
        task.completed = newCompletionState;
        console.log('État de la tâche mis à jour :', task.completed);
      } catch (error) {
        console.error("Erreur lors de la mise à jour de la tâche :", error);
      }
    },
    showEditModal(completed, task) {
    if (completed) {
        this.editTask(task);
        this.$bvModal.show('editModal'); 
    }
  },
    editTask(task) {
        this.$emit("taskEdit", task);
    },
    editTaskInput(index) {
      this.editedTask.title = this.tasks[index].title; 
      this.editedTask.index = index;
    },
    openEditModal(task) {
      this.$emit("taskEdit", task);
    },
    saveTask() {
      if (this.editedTask.index !== -1) {
        const index = this.editedTask.index;
        this.tasks[index].title = this.editedTask.title;
        this.tasks[index].content = this.editedTask.content;
        this.editedTask.index = -1;
        this.saveTasks(); // Sauvegarder les tâches après une édition
      }
    }
  },
  mounted() {
    this.getSavedTasks(); 
  }
};

</script>

<style>
@import "assets/css/style.css";
</style>
