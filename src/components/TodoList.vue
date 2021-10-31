<template>
  <transition name="a" mode="out-in">
      <div v-if="TodoLists.length > 0">
          <transition-group tag="ul" name="todo">
              <li 
                v-for="(todolist, index) in TodoLists"
                :key="index"
              >
                <span class="name" @click="Done($event)">{{ todolist.name }}</span>
                <button @click="Delete(todolist.id)">🗑</button>
              </li>
          </transition-group>
      </div>
      <div v-else>
          <p>Nothing to do buddy - relax!</p>
      </div>
  </transition>
</template>

<script>
export default {
    props: {
        TodoLists: Array,
        Delete: Function
    },
    methods: {
        Done(e){
            console.log(e.target);
            const cn = e.target.className;
            if (cn == "name") {
                e.target.className = "done";
            } else {
                e.target.className = "name";
            }
        }
    }
}
</script>

<style scoped>
  .a-enter-from,
  .a-leave-to {
    opacity: 0;
    transform: translateX(-50px);
  }
  .a-enter-to,
  .a-leave-from {
    opacity: 1;
    transform: translateX(0px);
  }
  .a-enter-active,
  .a-leave-active {
    transition: all 0.5s ease;
  }

  .todo-enter-from,
  .todo-leave-to {
    opacity: 0;
    transform: scale(0.15);
  }
  .todo-enter-to,
  .todo-leave-from {
    opacity: 1;
    transform: scale(1);
  }
  .todo-enter-active,
  .todo-leave-active {
    transition: all 0.5s ease;
  }

  .name, .done{
      font-size: 1.5rem;
  }

  p {
      text-align: center;
  }
</style>