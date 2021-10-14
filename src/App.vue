<template>
  <div class="top">
    <div class="card form">
      <input
        type="text"
        v-bind:value="inputValue"
        v-on:input="changeValue"
        @keyup.enter="addtask"
      /><!--v-model -->
      <button class="btn" v-on:click="addtask">create todo note</button>
    </div>
  </div>

  <div class="search">
    <input type="text" v-model="searchValue" placeholder="Поиск задач" />
    <button @click="sort = 'all'">Все задачи</button>
    <button @click="sort = 'active'">выполненные</button>
    <button @click="sort = 'done'">активные</button>
  </div>

  <div class="bottom need" v-if="tasks.length">
    <h2>Задачи</h2>
    <ul>
      <li v-for="(item, idx) in taskslist" :key="item.id" :class="{complete:item.status == true}">
        <div class="task">
          <input type="checkbox" v-model="item.status"/>
          <div
            v-if="!item.editable"
            @dblclick="editTask(item)"
            class="item-title"
          >
            <span>{{ item.title }}</span>
          </div>
          <input
            v-else
            class="item-edit"
            @keyup.enter="doneEditable(item)"
            @blur="doneEditable(item)"
            @keyup.esc="cancelEdit(item)"
            type="text"
            v-model="item.title"
          />
        </div>
        <button v-on:click="removeTask(idx, 'done')">remove</button>
      </li>
    </ul>
  </div>
  <!--   -->
  <div class="bottom done" v-if="tasksDone.length">
    <h2>Выполнено</h2>
    <ul>
      <li v-for="(item, idx) in tasksDone" :key="item.id" :class="{complete:item.status == true}" >
        <div class="task">
          <input
            type="checkbox"
            v-model="item.status"
          />
          <span>{{ item.title }}</span>
        </div>
        <button v-on:click="removeTask(idx, 'notdone')">remove</button>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  data() {
    return {
      inputValue: "",
      tasks: [],
      tasksDone: [],
      searchValue: "",
      searchScope: [],
      beforeItem: "",
      sort: "all",
    };
  },
  methods: {
    changeValue(event) {
      this.inputValue = event.target.value;
    },
    addtask() {
      if (this.inputValue.trim().length === 0) {
        return;
      }
      this.tasks.push({
        title: this.inputValue,
        id: Math.random(),
        status: false,
        editable: false,
      });
      this.inputValue = "";
      console.log("tasks> ", this.tasks);
    },
    checkstatus(index, type) {
      if (type == "done") {
        this.tasks[index].status = false;
        const complete = this.tasks[index];
        this.tasksDone.push(complete);
      } else {
        this.tasksDone[index].status = true;
        this.tasksDone.splice(index, 1);
      }
    },
    removeTask(index, type) {
      type === "done"
        ? (
          this.tasks.splice(index, 1)
        )
        : this.tasksDone.splice(index, 1);
    },
    searchTask() {
      return this.tasks.filter((item) => {
        return item.title
          .toLowerCase()
          .includes(this.searchValue.toLowerCase());
      });
    },
    onefindscope() {
      return this.tasks.filter((item) =>
        item.title.toLowerCase().includes(this.searchValue.toLowerCase())
      );
    },
    twofindscope() {
      return this.tasksDone.filter((item) =>
        item.title.toLowerCase().includes(this.searchValue.toLowerCase())
      );
    },
    editTask(item) {
      this.beforeItem = item.title;
      item.editable = true;
    },
    doneEditable(item) {
      if (item.title.trim().length === 0) {
        item.title = this.beforeItem;
      }
      item.editable = false;
    },
    cancelEdit(item) {
      item.title = this.beforeItem;
      item.editable = false;
    },
  },
  computed: {
    taskslist() {
      if (this.sort == "all") {
        return this.tasks.filter((item) =>
          item.title.toLowerCase().includes(this.searchValue.toLowerCase())
        );
      } else if (this.sort == "active") {
        return this.tasks.filter((item) => item.status == true);
      } else if (this.sort == "done") {
        return this.tasks.filter((item) => item.status == false);
      }
      return this.tasks
    },
  },
  mounted(){
    console.log('mounted')
    const data = JSON.parse(localStorage.getItem('notepad'));
    if(data){
      console.log('data->',data)
      this.tasks = data.tasks;
      this.tasksDone = data.tasksDone;
      this.beforeItem = data.beforeItem;
      this.sort = data.sort;
    }
  },
  updated(){
    console.log('updated')
    const backup={
      tasks: this.tasks,
      tasksDone:this.tasksDone,
      beforeItem: this.beforeItem,
      sort: this.sort,
    }
    localStorage.setItem('notepad',JSON.stringify(backup))
  }
};
</script>
<style lang="scss"> 
input {
  outline: none;
}
.title-page {
  width: fit-content;
  margin: 10px auto;
}
.search {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  height: 50px;
  padding: 10px;
  width: 100%;
  background: #c6d1c84d;
}
.top {
  width: 100%;
  position: relative;
  background: #c6d1c84d;
  padding: 15px;
  .card {
    margin: 20px auto 10px;
    display: flex;
    flex-direction: column;
    width: 263px;
    height: 65px;
    padding: 9px;
    background: #1121218c;

    input {
      margin-bottom: 10px;
      font-size: 20px;
    }

    button {
      padding: 7px;
      display: block;
      margin: 0 auto;
      background: #52853b;
      border: none;
      width: -webkit-fit-content;
      width: -moz-fit-content;
      width: fit-content;
      height: 40px;
      color: white;
      cursor: pointer;
    }
  }
}
.bottom {
  width: 100%;
  position: relative;
  height: 60%;
  background: #c6d1c84d;
  padding: 20px;
  margin-top: 10px;
  transition: all 0.4s;
  h2 {
    margin-left: 250px;
  }
  li {
    display: flex;
    margin: 10px auto;
    list-style-type: none;
    width: 90%;
    max-width: 400px;
    min-width: 280px;
    background: #8b8b8b;
    color: #ededed;
    font-size: 20px;
    border-radius: 5px;
    padding: 15px;
    cursor: pointer;
    transition: all 0.3s ease-out;

    button {
      display: block;
      margin-left: auto;
      background: #8d6060;
      border: none;
      width: fit-content;
      height: 35px;
      color: white;
      cursor: pointer;
    }

    .task {
      padding: 5px;
      display: flex;
      align-items: center;
    }

    input {
      cursor: pointer;
    }
  }
}
.need li:hover {
  background: #c5c5c5;
  border-radius: 15px;
  font-size: 22px;
  color: #52853b;
  button {
    background: #808080;
  }
}
li.complete {
    background: #8b8b8b6e;
    font-size: 16px;
    padding: 5px;
    color: #808080;
    button {
      margin-right: 5px;
      padding: 2px;
      background: #dadada;
      color: black;
    }
    span {
      text-decoration: line-through;
    }
  }

.item-edit {
  background: #dfdcdc1f;
  outline: none;
  border: 1px dashed darkgray;
  color: white;
  font-weight: 100;
  font-size: 18px;
}
</style>
