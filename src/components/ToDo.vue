<template>
  <div class="hello">
    <nav>
      <span class="button is-danger small is-rounded" @click="purge">Purge</span>
      <span class="button is-link small is-rounded" @click="signOut">SignOut</span>
    </nav>

    <h1 class=title>
      My Todos
      <span class="is-size-4 has-text-grey">({{ remaining.length }}/{{ todos.length }})</span>
    </h1>

    <article id="launch-modal">
      <section class="section">
        <div class="container">
          <div class="content has-text-centered">
            <p class="control">
              <button @click="active" class="button is-info is-rounded">Add ToDo</button>
            </p>
          </div>
        </div>
      </section>
      
      <div class="modal" :class="{'is-active':isActive}">
        <div v-click-outside="close" @click="addClose" class="modal-background"></div>
        <div class="modal-content">
          <div class="box">
            <div class="content has-text-centered">
              <div class="control">
                <form class="field is-grouped" @submit.prevent="Add">
                  <div class="control is-expanded">
                    <input class="input is-rounded" type="text" v-model="newItem" placeholder="Title">
                    <textarea class="textarea" type="text" v-model="newContent" style="white-space: pre;" placeholder="Todo"></textarea>
                    <input class="button is-info is-rounded" type="submit" value="Add">
                  </div>
                </form>
              </div>
              <span>&nbsp;</span>
            </div>
          </div>
        </div>
      </div>
    </article>

      <div class="modal" :class="{'is-active':isEditActive}">
        <div v-click-outside="close" @click="editClose" class="modal-background"></div>
        <div class="modal-content">
          <div class="box">
            <div class="content has-text-centered">
              <div class="control">
                <form class="field is-grouped" @submit.prevent="Edit">
                  <div class="control is-expanded">
                    <input class="input is-rounded" type="text" v-model="editedItem" placeholder="Title">
                    <textarea class="textarea" type="text" v-model="editedContent" style="white-space: pre;" placeholder="Todo"></textarea>
                    <input class="button is-info is-rounded" type="submit" value="Edit">
                  </div>
                </form>
              </div>
              <span>&nbsp;</span>
            </div>
          </div>
        </div>
      </div>

    <ul class="todos is-size-5" v-if="todos.length">
      <li v-for="(todo, index) in todos" :key="todo">
        <label class="checkbox">
          <input type="checkbox" v-model="todo.isDone">
        </label>
        <div class="card">
          <head class="card-header">
            <p class="card-header-title">
              <span :class="{done: todo.isDone}">{{ todo.title }}</span>
            </p>
          </head>
          <div class="card-content">
            <div class="content">
              <span :class="{done: todo.isDone}" style="white-space: pre;">{{ todo.content }}</span>
            </div>
          </div>
          <footer class="card-footer">
            <span @click="editActive" class="card-footer-item button">Edit</span>
            <span @click="deleteItem(index)" class="card-footer-item button is-danger">Delete</span>
          </footer>
        </div>
      </li>
    </ul>

    <ul v-else class="subtitle is-size-4">
      <li>Nothing Todos</li>
    </ul>
  </div>
</template>

<script>
  import ClickOutside from 'vue-click-outside'
  export default {
    name: 'ToDo',
    newItem: "",
    props: ["user"],
    data () {
      return {
        todos: [],
        components: [],
        isActive: false,
        isEditActive: false
      }
    },
    created: function () {
      firebase
        .database()
        .ref('todos/' + this.user.uid)
        .once('value')
        .then(result => {
          if (result.val()) {
            this.todos = result.val();
            this.components = result.val();
          }
        })
    },
    methods: {
      active: function() {
        this.isActive = !this.isActive
        firebase
          .database()
          .ref('todos/' + this.user.uid)
          .set(this.todos);
      },
      editActive: function (){
        this.isEditActive = !this.isEditActive
        firebase
          .database()
          .ref('todos/' + this.user.uid)
          .set(this.todos);
      },
      Add: function() {
        this.todos.push({
          title: this.newItem,
          content: this.newContent,
          isDone: false,
        }),
        firebase
          .database()
          .ref('todos/' + this.user.uid)
          .set(this.todos);

        this.newItem = ""
        this.newContent = ""
        this.isActive = !this.isActive
      },
      Edit: function (index){
        this.todos.splice(index, 1,{
          title: this.editedItem,
          content: this.editedContent
        });

        firebase
          .database()
          .ref('todos/' + this.user.uid)
          .set(this.todos);

        this.editedItem = ""
        this.editedContent = ""
        this.isEditActive = !this.isEditActive

      },
      addClose: function () {
        this.newItem = ""
        this.newContent = ""
        this.isActive = !this.isActive
      },
      editClose: function () {
        this.newItem = ""
        this.newContent = ""
        this.isEditActive = !this.isEditActive
      },
      deleteItem: function(index) {
        if(confirm('Are You Sure?')){
          this.todos.splice(index, 1);
        }
        firebase
          .database()
          .ref('todos/' + this.user.uid)
          .set(this.todos);
      },
      purge: function() {
        if(!confirm('delete finished?')){
          return; //Noを選んだら何もせす返す
        }
        this.todos = this.remaining;
        firebase
          .database()
          .ref('todos/' + this.user.uid)
          .set(this.todos);        
      },
      signOut: function () {
        console.log('')
        firebase.auth().signOut();
      }
    },
    mounted () {
      this.popupItem = this.$el
    },
    directives: {
      ClickOutside
    },
    computed: {
      remaining: function() {
        return this.todos.filter(function(todo) { //dataのtodosを参照し、
          return !todo.isDone; //未チェックのTodoを返す
        })
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
nav {
  padding-bottom: 2%;
}
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.hello li > span.done{
  text-decoration: line-through;
  color: dimgrey;
}
.todos {
  padding-top: 2%;
  padding-bottom: 2%;
}
.subtitle {
  padding-top: 3%;
  padding-bottom: 1%;
}
.control {
  /* padding-top: 12%; */
  text-align: center;
}
.field {
  max-width: 300px;
  margin: auto;
}
.close {
  position:absolute;
  top:0%;
  right:0%; 
}
</style>
