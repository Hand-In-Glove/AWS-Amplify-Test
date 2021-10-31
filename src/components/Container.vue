<template>
  <div class="container">
      <div class="welcome">
        <amplify-authenticator>
            <h1 class="title">Things to do:</h1>
            <transition name="f" appear>
                <form @submit.prevent="Create()">
                    <label>Enter New Item</label>
                    <input 
                        type="text"
                        placeholder="What to do?"
                        v-model="msg"
                        autocomplete="off"
                    >
                    <button>Add item</button>
                </form>
            </transition>
            <TodoList
                :TodoLists="TodoLists"
                :Delete="Delete"
            />
            <img class="gif" v-if="!!TodoLists.length" src="/BsfE.gif"/>
            <img class="gif" v-else src="/2Nrh.gif"/>
            <transition name="m" appear>
                <w-notification
                    v-show="showNotification"
                    warning
                    bottom
                    round
                    shadow
                    transition="bounce"
                >
                    Please enter a value
                </w-notification>
            </transition>
            <amplify-sign-out/>
        </amplify-authenticator>
      </div>
  </div>
</template>

<script>
import TodoList from './TodoList.vue';
import { AuthState, onAuthUIStateChange } from '@aws-amplify/ui-components';
import { API, graphqlOperation } from 'aws-amplify';
import { listItems } from '../graphql/queries';
import { createItem, deleteItem } from '../graphql/mutations';
import { onCreateItem, onDeleteItem } from '../graphql/subscriptions';

export default {
    components: {
        TodoList,
    },
    data: () => ({
        user: {},
        loading: true,
        form: {},
        msg: "",
        showNotification: false,
        timeout: 2000,
        exampleData: [],
        TodoLists: [],
    }),
    created(){
      onAuthUIStateChange((state, user) => {
        if (state === AuthState.SignedIn){
          this.user = user;
          this.getData();
        } 
      });
      
      API.graphql(graphqlOperation(onCreateItem)).subscribe((sourceData) => {
        const newItem = sourceData.value.data.onCreateItem;
        if (newItem) {
          if (this.TodoLists.some(item => item.id === newItem.id)) return;
          this.TodoLists = [...this.TodoLists, newItem];
        }
      });
      API.graphql(graphqlOperation(onDeleteItem)).subscribe((sourceData) => {
        const removedItem = sourceData.value.data.onDeleteItem;
        if (removedItem) {
          this.TodoLists = this.TodoLists.filter(item => item.id !== removedItem.id);
        }
      });
    },
    methods: {
        async getData() {
          try { 
            this.loading = true;
            const res = await API.graphql(graphqlOperation(listItems));
            this.TodoLists = res.data.listItems.items;
          } catch (error) {
            console.log("There was an error loading data.", error)
          }
          finally {
            this.loading = false;
          }
        },
        async createItem() {
          const name = this.msg;
          const item = { name };
          try {
            const res = await API.graphql(graphqlOperation(createItem, { input: item }));
            console.log("Item created...");
            this.TodoLists = [...this.TodoLists, res.data.createItem];
          } catch (error) {
            console.log("There was an error creating new item.", error)
          }
        }, 
        async deleteItem(id) {
          if (id) {
            try {
              await API.graphql(graphqlOperation(deleteItem, { input: { id } }));
              console.log("Item deleted...");
              this.TodoLists = this.TodoLists.filter(item => item.id !== id);
            } catch (error) {
              console.log("There was an error deleting data.", error)
            }
          }
        },
        Create() {
            setTimeout(() => {
                this.showNotification = false;
            }, this.timeout);

            if (this.msg.length) {
                // let id = Math.floor(Math.random()*100);
                // const next = { id, name: this.msg };
                // this.TodoLists.push(next);
                this.createItem();
                this.msg = "";
            }
            else {
                this.showNotification = true;
            }
        },
        Delete(idx) {
            // this.TodoLists = this.TodoLists.filter(item => item.id !== idx);
             this.deleteItem(idx);
        }
    }
}
</script>

<style scoped>
  .title{
    margin: 2rem;
    font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
  }
  .f-enter-from,
  .f-leave-to {
    opacity: 0;
    transform: translateX(-40px);
  }
  .f-enter-to,
  .f-leave-from {
    opacity: 1;
    transform: translateX(0px);
  }
  .f-enter-active,
  .f-leave-active {
    transition: all 0.5s ease;
  }
  .m-enter-active {
    animation: shake 0.5s ease-in;
  }

  .gif{
      display: block;
      margin: 1rem auto;
      max-width: 30vw;
  }

  @keyframes shake {
    0% { transform: translateX(-40px); opacity: 0; }
    25% { transform: translateX(-10px); opacity: 1; }
    50% { transform: translateY(-40px); }
    75% { transform: translateY(0px);}
    100% { transform: translateX(0px); }
  }
</style>