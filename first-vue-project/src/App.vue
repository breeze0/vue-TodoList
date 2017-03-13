<template>
  <div id="app" class="container">
    <header>
      <h1 v-html="tittle"></h1>
      <input type="checkbox" v-model="allDone">
      <input type="text" placeholder="Please add a new todo"
      v-model="newtodo"
      v-on:keyup.enter="addNewtodo"
      >
    </header>
    <section>
      <ul>
        <li v-for="(item,index) in filtereditems" 
            >
          <div class="view" v-show="item.showView">
            <input class="checkboxStyle" type="checkbox" v-model="item.isFinished"  >
            <label class="inputStyle" v-bind:class="{finished:item.isFinished}" @dblclick="editTodo(item)">{{item.label}}</label>
            <button class="destroy" v-on:click="deleteTodo(index)" >X</button>
          </div>
          <input
            v-show="!item.showView" 
            v-model="item.label"
            v-focus="true"
            v-on:blur="doneEdit(item,index)"
            v-on:keyup.enter="doneEdit(item, index)"
            v-on:keyup.esc="canelEdit(item)"
          >
        </li>
      </ul>
    </section>
    <footer v-show="items.length">
      <span class="item-count"><strong> {{remaining}} </strong>{{remaining | pluralize}} left</span>
      <ul class="filters">
        <li class="filters-li"><a href="#all" @click="changeVisibilityAll" v-bind:class="{selected: visibility == 'all'}">All</a></li>
        <li class="filters-li"><a href="#active" @click="changeVisibilityActive" v-bind:class="{selected: visibility == 'active'}">Active</a></li>
        <li class="filters-li"><a href="#finished" @click="changeVisibilityFinished" v-bind:class="{selected: visibility == 'finished'}">Finished</a></li>
        <button @click="clearFinished" v-show="items.length > remaining">ClearFinished</button>
      </ul>
    </footer>
  </div>
</template>

<script>
  import Store from './store'
  var filters = {
    all: function(items) {
      return items
    },
    active:function(items) {
      return items.filter(function(item) {
        return !item.isFinished
      })
    },
    finished:function(items) {
      return items.filter(function(item) {
        return item.isFinished
      })
    }
  }
  export default {
    data() {
      return {
        tittle: "this is a todo list <span>!</span>  ",
        items:Store.fetch(),
        newtodo:"",
        visibility:"all"
      }
    },

    watch: {
      items: {
        handler: function (items) {
          Store.save(items)
        },
        deep:true
      }
    },
    computed: {
      allDone: {
        get: function() {
         var i=0
         if(this.items.length != 0) {   
           this.items.forEach(function(item) {
                if (!item.isFinished) {
                  i++
                }
            })
            return i === 0
          }
          else {
            return false
          }
        }, 
        set: function(value) {
          this.items.forEach(function(item) {
            item.isFinished=value
          })
        }
      },
      remaining: function() {
        return filters.active(this.items).length
      },
      filtereditems: function() {
        return filters[this.visibility](this.items)
      },
    },
    filters: {
      pluralize: function(n) {
        return n===1? 'item':'items'
      } 
    },
    methods: {
      addNewtodo:function(){
        if (this.newtodo) {
        this.items.push({
          label:this.newtodo,
          isFinished:false,
          showView: true
        })
      }
        this.newtodo=""
      },
      deleteTodo: function (index) {
        this.items.splice(index,1)
      },
      editTodo: function(item) {
        this.beforeEditCache=item.label
        item.showView = false
      },
      doneEdit: function(item,index) {
        if(!item.label){
          this.deleteTodo(index)
        }
        item.showView= true
      },
      canelEdit: function(item) {
        item.label=this.beforeEditCache
        item.showView= true

      },
      clearFinished: function() {
        this.items=filters.active(this.items)
      },
      changeVisibilityAll: function() {
        this.visibility='all'
      },
      changeVisibilityActive: function() {
        this.visibility='active'
      },
      changeVisibilityFinished: function() {
        this.visibility='finished'
      }
    },
      directives: {
        focus: function(el1,value) {
          if (value) {
            el1.focus()
          }
        }
      } 
  }

      
  
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.container {
  width: 500px;
  height: auto;
}
.finished {
  color: gray;
  text-decoration: line-through;
}
ul li {
  list-style: none;
}
.view {
  display: flex;
  justify-content: space-between;
  width: 240px;
  margin: auto
}
.destroy {  
  border:none;
  background-color: white;  
}
.filters-li {
  position: relative;
  float: left;
  padding-left: 20px;
}
.item-count {
  position: relative;
  float: left;
  margin-left: 100px; 
}
.filters-li a.selected {
  border:1px solid #999;
} 
.filters-li a {
  text-decoration: none;
}

</style>
