<template>
  <div class="form-wrapper">
    <form>
      
      <input v-model="inputValue" type="text" class="form-input" placeholder="Введите название задачи" />
      <select v-model="priorityValue" class="form-input" >
        <option value="" disabled selected hidden>Введите приоритет</option>
        <option value="Высокий">Высокий</option>
        <option value="Средний">Средний</option>
        <option value="Ниизкий">Низкий</option>
      </select>

      <button @click.prevent="createTask" >Создать</button>
    </form>
  </div>
  <div class="list-wrapper">
    <div class="radio-section">
      <label for="radio2">Показать список</label>
      <input type="radio"  id="radio2" class="radio-show" :value="true" v-model="isShow"/>
      <label for="radio1">Скрыть список</label>
      <input type="radio" id="radio1" class="radio-hidden"  :value="false"  v-model="isShow" />

    <div class="filters">
      <label><input type="checkbox" v-model="showAllTasks"> Все задачи</label>
      <label><input type="checkbox" v-model="showImportantTasks"> Важные задачи</label>
      <label><input type="checkbox" v-model="showCompletedTasks"> Выполненные задачи</label>
      <label><input type="checkbox" v-model="showUncompletedTasks"> Невыполненные задачи</label>
    </div>
  </div>
  <ul v-show="isShow">
    <li v-for="task of filteredTasks" :key="task.id" class="list-item" >
      <div>
        <p v-if="!task.isEdit" :class="{ 'completed': task.isCompleted }">{{ task.description }}</p>
        <input v-else v-model="task.description" type="text">
        <p>Приоритет: {{ task.priority }}</p> 
      </div>
      <div class="button-section">
        <button @click="task.isEdit = !task.isEdit">{{ task.isEdit ? "Сохранить" : "Редактировать"}}</button>
        <button @click="deleteTask(task.id)">Завершить</button>
        <button @click="toggleCompleted(task)">Отметить как {{ task.isCompleted ? 'невыполненную' : 'выполненную' }}</button>
      </div>
    </li>
    </ul>
    <div v-if="notification" class="notification" :class="notification.type">
      {{ notification.message }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const todoList = ref ([
  {
    id: 0,
    description: 'Сделать уроки',
    priority: 'Низкий',
    isEdit: false,
    isCompleted: false,
  },
  {
    id: 1,
    description: 'Сходить в магазин',
    priority: 'Высокий',
    isEdit: false,
    isCompleted: false,
  },
  {
    id: 2,
    description: 'Погулять',
    priority: 'Средний',
    isEdit: false,
    isCompleted: false,
  },
])

const deleteTask = (id) => {
  todoList.value = todoList.value.filter (el => el.id !== id)
  showNotification('Задача удалена', 'error');
}

const inputValue = ref('');

const createTask = () => {
  if (!inputValue.value && !priorityValue.value) {
    alert('Укажите название и приоритет задачи');
  } else if (!inputValue.value) {
    alert('Укажите название задачи');
  } else if (!priorityValue.value) {
    alert('Выберите приоритет задачи');
  } else {
    const newTask = {
      id: Date.now(),
      description: inputValue.value,
      priority: priorityValue.value,
      isEdit: false,
      isCompleted: false,
    };
    todoList.value.push(newTask);
    showNotification(`Задача "${newTask.description}" создана`, 'success');
  }
};


const isShow = ref(true);

const priorityValue = ref('');

const toggleCompleted = (task) => {
  task.isCompleted = !task.isCompleted;
};

const showAllTasks = ref(true);
const showImportantTasks = ref(false);
const showCompletedTasks = ref(false);
const showUncompletedTasks = ref(false)

const filteredTasks = computed(() => {
  let filtered = todoList.value;

  if (!showAllTasks.value && !showImportantTasks.value && !showCompletedTasks.value && !showUncompletedTasks.value) {
    return filtered;
  }

  if (!showAllTasks.value) {
    if (showImportantTasks.value) {
      filtered = filtered.filter(task => task.priority === 'Высокий');
    } else {
      if (showCompletedTasks.value) {
        filtered = filtered.filter(task => task.isCompleted);
      }
    }
  }

  if (showCompletedTasks.value && !showUncompletedTasks.value) {
    filtered = filtered.filter(task => task.isCompleted); 
  }

  if (!showCompletedTasks.value && showUncompletedTasks.value) {
    filtered = filtered.filter(task => !task.isCompleted); 
  }

  return filtered;
});

const notification = ref(null);

const showNotification = (message, type = 'success') => {
  notification.value = { message, type };
  setTimeout(() => {
    notification.value = null;
  }, 3000); 
};

</script>

<style scoped>

body {
  background-color: gray;
}

form {
  width: 100%;

}

.form-wrapper {
  width: 100%;
  height: 60px;
  display: flex;
  align-items: center;
  padding: 10px;
}

.form-input{
  width: 400px;
  height: 40px;
  border-radius: 4px;
  margin: 0 10px
}

.form-select {
  width: 200px;
  height: 40px;
  border-radius: 4px;
}

button {
  padding: 0 50px;
  height: 40px;
  background-color: teal;
  border-radius: 4px;
  margin: 0 10px;
  cursor: pointer;
  color: white;
  font-size: 14px;
  
}

button:hover {
  background-color: rgb(1, 95, 95);

}
.list-wrapper {
  border: 1px solid black;
  border-radius: 4px;
  margin: 10px;
  padding: 10px;
  background-color: white;
  min-height: 40px;
}
 ul {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
 }

 .list-item{
  list-style-type: none;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 60px;
  padding: 0 20px;
  border: 2px solid black;
  
 }
 .radio-section input {
  margin: 0 10px;
 }
 input[type=text]::placeholder, input[type=text] {
	font-size: 14px;
}

input[type='radio']:after {
  width: 15px;
  height: 15px;
  border-radius: 15px;
  top: -2px;
  left: -1px;
  position: relative;
  background-color: #d1d3d1;
  content: '';
  display: inline-block;
  visibility: visible;
  border: 2px solid white;
}

input[type='radio']:checked:after {
  width: 15px;
  height: 15px;
  border-radius: 15px;
  top: -2px;
  left: -1px;
  position: relative;
  background-color: teal;
  content: '';
  display: inline-block;
  visibility: visible;
  border: 2px solid white;
}

 .completed {
  text-decoration: line-through;
}

.filters {
  padding: 10px 0;
}

.notification {
  position: fixed;
  bottom: 20px; 
  right: 20px; 
  padding: 10px 50px;
  border-radius: 5px;
  color: #fff;
  z-index: 999;
  opacity: 0.7; 
}

.success {
  background-color: #4caf50;
}

.error {
  background-color: #f44336;
}
</style>