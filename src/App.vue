<script setup>
import { onMounted, ref } from "vue";

const tasksModel = ref({
  tasks: [],
  editing: null,
});

const createTask = (data, form$) => {
  addToStorage(form$.data);
  syncFromStorage();

  form$.reset();
  // form$.clear();
};

const addToStorage = (data) => {
  let storageData = localStorage.getItem("tasks");
  storageData = storageData ? JSON.parse(storageData) : [];

  storageData.push(data);
  localStorage.setItem("tasks", JSON.stringify(storageData));
};

const syncFromStorage = () => {
  let tasks = localStorage.getItem("tasks");

  tasksModel.value = {
    tasks: tasks ? JSON.parse(tasks) : [],
  };
};

const syncToStorage = () => {
  localStorage.setItem("tasks", JSON.stringify(tasksModel.value.tasks));
};

const edit = (index) => {
  tasksModel.value.editing = index;
};

const cancel = () => {
  tasksModel.value.editing = null;
  syncFromStorage();
};

const save = () => {
  syncToStorage();
  tasksModel.value.editing = null;
};

onMounted(() => {
  syncFromStorage();
});
</script>

<template>
  <div class="page">
    <div class="container">
      <h1>Vue To-Do List</h1>

      <Vueform size="lg" :endpoint="createTask">
        <!-- the Vueform component will call 'createTask' when the form is submitted -->
        
        <TextElement
          name="task"
          placeholder="Add a task"
          floating="Task name"
          rules="required"
        />
        <!-- input field of name 'task' maps to 'tasks' in tasksModel internally -->
        <!-- the input field -->

        <RadiogroupElement
          name="type"
          :items="['Personal', 'Business']"
          view="tabs"
          default="Personal"
        />
        <!-- the radio group -->
      
        <ButtonElement name="button" align="right" submits>
          Submit
        </ButtonElement>
        <!-- the submit button -->
      </Vueform>

      <hr class="divider" />
      <!-- the horizontal rule -->

      <Vueform v-model="tasksModel" sync>
      <!-- the sync prop will keep the form data and the `taskModel` in sync -->

        <ListElement
          name="tasks"
          :controls="{
            add: false,
          }"
          :add-class="{
            handle: 'task-sort-handle',
          }"
          sort
          @sort="syncToStorage"
          @remove="syncToStorage"
        >
        <!-- the name 'tasks' binds form data to tasksmodel and syncs with local storage. list rendering handled internally by listelement. -->
        <!-- adding is disabled but sort and remove are enabled -->
        <!-- the add-class is used to add styling the the handle element, which is used for drag and drop -->
        <!-- the event listener [@sort, @remove] will call 'syncToStorage' method when the list is sorted or an element is removed -->

          <template #default="{ index }">
          <!-- default slot for inserting custom content from parent. just like props but for content placement inside child components -->
            
            <ObjectElement
              :name="index"
              :add-class="[
                'task-container',
                tasksModel.tasks[index].type === 'Personal'
                  ? 'is-personal'
                  : 'is-business',
              ]"
            >
            <!-- used conditional rendereng for styling -->
            <!-- name is the index of the task -->
              
              <ButtonElement
                :label="`#${index + 1} - ${tasksModel.tasks[index].task}`"
                name="edit"
                align="right"
                :conditions="[['editing', '!=', index]]"
                :columns="{
                  label: 8,
                }"
                @click="edit(index)"
              >
                Edit
              </ButtonElement>
              <!-- label is the text inside the button -->
              <!-- conditions are used to show/hide the button -->
              <!-- on click the 'edit' method is called -->
              
              <TextElement
                name="task"
                :conditions="[['editing', index]]"
                :columns="6"
              />
              <!-- used to change the task name -->
              
              <RadiogroupElement
                name="type"
                view="tabs"
                :conditions="[['editing', index]]"
                :columns="2"
                :items="{
                  Personal: 'P',
                  Business: 'B',
                }"
              />
              <!-- used to change the task type -->
              
              <ButtonElement
                name="cancel"
                :conditions="[['editing', index]]"
                :columns="2"
                danger
                full
                @click="cancel"
              >
                Cancel
              </ButtonElement>
              <!-- on click the 'cancel' method is called -->

              <ButtonElement
                name="save"
                :conditions="[['editing', index]]"
                :columns="2"
                full
                @click="save"
              >
                Save
              </ButtonElement>
              <!-- on click the 'save' method is called -->

            </ObjectElement>
          </template>
        </ListElement>
        
        <HiddenElement name="editing" />
        <!-- used to store the index of the task being edited -->
      </Vueform>
    </div>
  </div>
</template>

<style lang="scss">
.page {
  background: #f1f5f9;
  width: 100%;
  min-height: 100vh;
  padding-top: 2rem;
}

.container {
  max-width: 600px;
  margin: 0 auto;
}

h1 {
  font-size: 48px;
  margin-bottom: 2rem;
  font-weight: 600;
}

.divider {
  margin: 2rem 0;
  border-color: #e2e8f0;
}

.task-container {
  background: #ffffff;
  padding: 1rem;

  &.is-personal {
    border-left: 3px solid green;
  }

  &.is-business {
    border-left: 3px solid purple;
  }
}

.task-wrapper {
  display: flex;
  align-items: center;
}

.vf-list-handle.task-sort-handle {
  top: 1rem;
}
</style>
