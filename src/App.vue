<template>
  <main :class="`${defaultTheme} min-vh-100 py-5 d-flex align-items-center justify-content-center flex-column`">
    <div class="todo container">
      <header class="d-flex align-items-center justify-content-between mb-5">
        <h1 class="logo text-uppercase mb-0 fw-bold">Todo</h1>
        <div class="dark-light-icon">
          <img v-if="defaultTheme === 'dark'" @click="defaultTheme = 'light'" src="/images/icon-sun.svg"
            alt="">
          <img v-else @click="defaultTheme = 'dark'" src="/images/icon-moon.svg" alt="">
        </div>
      </header>
      <div class="add-todo p-3 px-4 rounded d-flex align-items-center gap-3 mb-3">
        <span class="d-block rounded-circle"></span>
        <input class="border-0 flex-fill p-0" type="text" placeholder="Create new todo..." v-model="todoInput"
          @keypress.enter="addTodo">
      </div>
      <div class="todo-list rounded d-flex aling-items-center flex-column">
        <draggable v-model="todos" class="todo-body flex-fill position-relative">
          <transition-group type="transition" name="list" v-if="todos.length > 0">
            <div class="todo p-4 px-4 d-flex align-items-center justify-content-between"
              v-for="(todo, index) in filteration" :key="todo">
              <div :class="`left-side flex-fill d-flex align-items-center gap-3 ${todo.completed ? 'complete' : ''}`">
                <span class="d-block rounded-circle position-relative" @click="todo.completed = !todo.completed">
                  <img class="check position-absolute" src="/images/icon-check.svg" alt="">
                </span>
                <h5 class="mb-0 flex-fill">{{ todo.name }}</h5>
              </div>
              <img @click="removeTodo(todo)" class="cross" src="/images/icon-cross.svg" alt="">
            </div>
          </transition-group>
          <div class="d-flex align-items-center justify-content-center position-absolute w-100 h-100" v-else>
            <h3 class="text-center">Nothing To Show</h3>
          </div>
        </draggable>
        <footer class="d-flex p-3 px-4 align-items-center justify-content-between">
          <span class="total">{{ todosLeft }} items left</span>
          <div class="navs d-flex align-items-center justify-content-center gap-3">
            <span @click="defaultActiveTab = 'all'" :class="{ active: defaultActiveTab === 'all' }">All</span>
            <span @click="defaultActiveTab = 'active'" :class="{ active: defaultActiveTab === 'active' }">Active</span>
            <span @click="defaultActiveTab = 'completed'"
              :class="{ active: defaultActiveTab === 'completed' }">Completed</span>
          </div>
          <span class="clear" @click="clearCompleted">Clear Completed</span>
        </footer>
      </div>
      <div class="navs-mobile mt-3 d-none d-flex align-items-center justify-content-center gap-3 p-3 px-4 rounded">
        <span @click="defaultActiveTab = 'all'" :class="{ active: defaultActiveTab === 'all' }">All</span>
        <span @click="defaultActiveTab = 'active'" :class="{ active: defaultActiveTab === 'active' }">Active</span>
        <span @click="defaultActiveTab = 'completed'"
          :class="{ active: defaultActiveTab === 'completed' }">Completed</span>
      </div>
      <span class="info text-center d-block mt-5 fw-bold">Drag and drop to reorder list</span>
    </div>
  </main>
</template>
<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import { VueDraggableNext as draggable } from 'vue-draggable-next'

const defaultTheme = ref('dark')
const defaultActiveTab = ref('all')

const todoInput = ref('')
const todos = ref([])
const todosLeft = ref(0)

let filteration = computed(() => {
  if (defaultActiveTab.value === 'all') {
    return todos.value.filter(el => el)
  } else if (defaultActiveTab.value === 'active') {
    todosLeft.value = todos.value.filter(el => el.completed === false).length
    return todos.value.filter(el => el.completed === false)
  } else if (defaultActiveTab.value === 'completed') {
    return todos.value.filter(el => el.completed === true)
  }
})

let addTodo = () => {
  if (todoInput.value === '') {
    return
  }
  todos.value.push({
    name: todoInput.value,
    completed: false
  })
  localStorage.setItem('todos', JSON.stringify(todos.value))
  todoInput.value = ''
}

let removeTodo = (todo) => {
  todos.value = todos.value.filter(el => el != todo)
}

let clearCompleted = () => {
  todos.value = todos.value.filter(el => el.completed === false)
}

watch([todos, defaultTheme], ([newTodos, newDefaultTheme]) => {
  todosLeft.value = computed(() => {
    return newTodos.filter(el => el.completed === false).length
  })
  localStorage.setItem('todos', JSON.stringify(newTodos))
  localStorage.setItem("theme", newDefaultTheme)
}, { deep: true })


onMounted(() => {
  if (localStorage.getItem('todos')) {
    todos.value = JSON.parse(localStorage.getItem('todos'))
  }
  if (localStorage.getItem('theme')) {
    defaultTheme.value = localStorage.getItem('theme')
  }
})


</script>
<style lang="scss">
* {
  box-sizing: border-box;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

:root {
  // ### Primary
  --Bright-Blue: hsl(220, 98%, 61%);
  --Check-Background: linear-gradient(hsl(192, 100%, 67%), hsl(280, 87%, 65%));

  // ### Neutral
  // ### Light Theme
  --Very-Light-Gray: hsl(0, 0%, 98%);
  --Very-Light-Grayish-Blue: hsl(236, 33%, 92%);
  --Light-Grayish-Blue-light: hsl(233, 11%, 84%);
  --Dark-Grayish-Blue-light: hsl(236, 9%, 61%);
  --Very-Dark-Grayish-Blue-light: hsl(235, 19%, 35%);

  // ### Dark Theme
  --Very-Dark-Blue: hsl(235, 21%, 11%);
  --Very-Dark-Desaturated-Blue: hsl(235, 24%, 19%);
  --Light-Grayish-Blue-dark: hsl(234, 39%, 85%);
  --Light-Grayish-Blue-hover: hsl(236, 33%, 92%);
  --Dark-Grayish-Blue-dark: hsl(234, 11%, 52%);
  --Very-Dark-Grayish-Blue-dark-1: hsl(233, 14%, 35%);
  --Very-Dark-Grayish-Blue-dark-2: hsl(237, 14%, 26%);
}

body {
  font-family: 'Josefin Sans', sans-serif;
  font-size: 18px;
}

// Dark Theme
.dark {
  background-image: url(/images/bg-desktop-dark.jpg);
  background-color: var(--Very-Dark-Blue);

  .logo {
    color: var(--Very-Light-Gray);
  }

  .add-todo {
    background-color: var(--Very-Dark-Desaturated-Blue);
    box-shadow: 0 5px 15px #07070773;

    span {
      border-color: var(--Very-Dark-Grayish-Blue-dark-2) !important;
    }

    input {
      color: var(--Light-Grayish-Blue-dark);

      &::placeholder {
        color: var(--Dark-Grayish-Blue-dark);
      }
    }
  }

  .todo-list {
    background-color: var(--Very-Dark-Desaturated-Blue);
    box-shadow: 0 5px 15px #07070773;

    .todo-body {
      .todo {
        &:not(:last-child) {
          border-color: var(--Very-Dark-Grayish-Blue-dark-1) !important;
        }

        span {
          background: linear-gradient(var(--Very-Dark-Desaturated-Blue), var(--Very-Dark-Desaturated-Blue)) padding-box,
            linear-gradient(var(--Very-Dark-Grayish-Blue-dark-2), var(--Very-Dark-Grayish-Blue-dark-2)) border-box;

          &:hover {
            background: linear-gradient(var(--Very-Dark-Desaturated-Blue), var(--Very-Dark-Desaturated-Blue)) padding-box,
              var(--Check-Background) border-box;
          }
        }

        h5 {
          color: var(--Light-Grayish-Blue-dark);
          font-size: 18px;
        }

        .left-side {
          &.complete {
            span {
              background: var(--Check-Background) !important;
              border: none;
            }

            h5 {
              text-decoration: line-through;
              color: var(--Very-Dark-Grayish-Blue-dark-1);
            }
          }
        }
      }

      h3 {
        color: var(--Dark-Grayish-Blue-dark);
      }
    }

    footer {
      color: var(--Very-Dark-Grayish-Blue-dark-1);

      .clear,
      .navs span {
        &:hover {
          color: var(--Light-Grayish-Blue-hover);
        }
      }

      .navs span {
        &.active {
          color: var(--Bright-Blue);
        }
      }
    }
  }

  .info {
    color: var(--Very-Dark-Grayish-Blue-dark-1);
  }

  @media (max-width: 767px) {
    background-image: url(/images/bg-mobile-dark.jpg);

    .navs-mobile {
      background-color: var(--Very-Dark-Desaturated-Blue) !important;
      box-shadow: 0 5px 15px #07070773;
      color: var(--Very-Dark-Grayish-Blue-dark-1);

      span {
        &:hover {
          color: var(--Light-Grayish-Blue-hover);
        }

        &.active {
          color: var(--Bright-Blue);
        }
      }
    }
  }
}

// Light Theme
.light {
  background-image: url(/images/bg-desktop-light.jpg);
  background-color: var(--Very-Light-Gray);

  .logo {
    color: var(--Very-Light-Gray);
  }

  .add-todo {
    background-color: white;
    box-shadow: 0 5px 15px #b4b4b45e;

    span {
      border-color: var(--Very-Light-Grayish-Blue) !important;
    }

    input {
      color: var(--Very-Dark-Grayish-Blue-light);

      &::placeholder {
        color: var(--Dark-Grayish-Blue-dark);
      }
    }
  }

  .todo-list {
    background-color: white;
    box-shadow: 0 5px 15px #b4b4b45e;

    .todo-body {
      .todo {
        &:not(:last-child) {
          border-color: var(--Light-Grayish-Blue-light) !important;
        }

        span {
          background: linear-gradient(white, white) padding-box,
            linear-gradient(var(--Very-Light-Grayish-Blue), var(--Very-Light-Grayish-Blue)) border-box;

          &:hover {
            background: linear-gradient(white, white) padding-box,
              var(--Check-Background) border-box;
          }
        }

        h5 {
          color: var(--Very-Dark-Grayish-Blue-light);
          font-size: 18px;
        }

        .left-side {
          &.complete {
            span {
              background: var(--Check-Background) !important;
              border: none;
            }

            h5 {
              text-decoration: line-through;
              color: var(--Light-Grayish-Blue-light);
            }
          }
        }
      }

      h3 {
        color: var(--Dark-Grayish-Blue-light);
      }
    }

    footer {
      border-color: var(--Light-Grayish-Blue-light) !important;
      color: var(--Dark-Grayish-Blue-light);

      .clear,
      .navs span {
        &:hover {
          color: var(--Very-Dark-Grayish-Blue-light);
        }
      }

      .navs span {
        &.active {
          color: var(--Bright-Blue);
        }
      }
    }
  }

  .info {
    color: var(--Dark-Grayish-Blue-light);
  }

  @media (max-width: 767px) {
    background-image: url(/images/bg-mobile-light.jpg);

    .navs-mobile {
      background-color: white !important;
      box-shadow: 0 5px 15px #b4b4b45e;
      color: var(--Dark-Grayish-Blue-light);

      span {
        &:hover {
          color: var(--Very-Dark-Grayish-Blue-light);
        }

        &.active {
          color: var(--Bright-Blue);
        }
      }
    }
  }
}

main {
  transition: 0.2s;
  background-size: 100%;
  background-repeat: no-repeat;

  .todo {
    width: 550px;
    max-width: 100%;
    transition: 0.5s;

    header {
      h1 {
        letter-spacing: 15px;
      }

      .dark-light-icon {
        cursor: pointer;
      }
    }

    .add-todo {

      span {
        width: 25px;
        height: 25px;
        border: 2px solid;
      }

      input {
        background: none;
        border: none;
        outline: none;
        caret-color: var(--Bright-Blue);
      }
    }

    .todo-list {
      min-height: 300px;

      .todo-body {
        .todo {
          cursor: pointer;

          &:not(:last-child) {
            border-bottom: 1px solid;
          }

          .cross {
            transition: 0.2s;
          }

          .left-side {
            &.complete {
              span .check {
                opacity: 1;
              }
            }

            span {
              width: 25px;
              height: 25px;
              border: 2px solid transparent;

              .check {
                opacity: 0;
                top: 50%;
                left: 50%;
                transform: translate(-50%, -50%);
              }
            }
          }

          &:hover {
            .cross {
              opacity: 1;
            }
          }

          .cross {
            opacity: 0;
          }
        }
      }

      footer {
        border-top: 1px solid;
        font-size: 15px;

        span {
          &:not(.clear, .total) {
            font-weight: bold;
          }

          transition: 0.2s;
        }

        .clear,
        .navs span {
          cursor: pointer;
        }
      }
    }

    .info {
      font-size: 12px;
    }
  }

  @media (max-width: 767px) {
    .todo {
      footer {
        .navs {
          display: none !important;
        }
      }

      .navs-mobile {
        display: flex !important;
      }
    }
  }
}

.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
.list-leave-active {
  position: absolute;
}
</style>