<template>
    <div class="card">
        <div class="card-header">
            <h1 class="text-center">Simple Kanban Board</h1>
        </div>
        <div class="card-body">
                        <a href="/export"><button class="btn btn-primary" > Export DB <font-awesome-icon class="ml-2" icon="fa-solid fa-file-export" /> </button></a>
<button
      type="button"
      class="btn btn-outline-success"
      @click="showModal"
    >
      Add Column or Task <font-awesome-icon class="ml-2" icon="fa-solid fa-circle-plus" />
    </button>

    <CreateTask
      v-show="isModalVisible"
      @close="closeModal"
    />
        
            <div class="py-4">
                <div class="row">
                    <div class="col-sm-12">
                        <main class="flexbox py-4">
         
                            <div class="row">
      
  
                                <div
                                    class="col text-center"
                                    v-for="board in boards"
                                    v-bind:key="board.id"
                                >
                                    <h3 class="alert alert-primary">{{ board.name }} 
                                    <div class="float-right">
                                        <button
                                                @click="editBoard(board)"
                                                class="btn btn-outline-success btn-sm"
                                            >
                                                <span class="h5"><font-awesome-icon icon="fa-solid fa-pen-to-square" /></span>
                                            </button>

                                    <button
                                                @click="deleteBoard(board.id)"
                                                class="btn btn-outline-danger btn-sm"
                                            >
                                                <span class="h5"><font-awesome-icon icon="fa-solid fa-trash-can" /></span>
                                            </button>
                                    </div>
                                     
                                        
                                            </h3>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col" v-for="board in boards" v-bind:key="board.id">
                                    <Board :id="board.id">
                                        <div v-for="task in tasks" v-bind:key="task.id">
                                            <div v-if="board.name == task.board_name">
                                                <Task :id="task.id" draggable="true">
                                                <div class="border">

                                                
                                                    <h3 calss="ml-2">{{ task.name }} <span class="float-right"><button
                                                                @click="editTask(task)"
                                                                class="btn btn-outline-success btn-sm mr-2"
                                                            >
                                                                <span class="h6"><font-awesome-icon icon="fa-solid fa-arrow-up-right-from-square" /></span>
                                                            </button></span></h3></div>
                                                    <div class="row">
                                                        <div class="col-xl-12 py-1">
                                                            
                                                        </div>
                                                        <div class="col-xl-12 py-1">
                                                            <button
                                                                @click="deleteTask(task.id)"
                                                                class="btn btn-outline-danger btn-sm"
                                                            >
                                                                <span class="h6"><font-awesome-icon  icon="fa-solid fa-trash-can" /></span>
                                                            </button>
                                                        </div>
                                                    </div>
                                                </Task>
                                            </div>
                                        </div>
                                    </Board>
                            
                                </div>
                            </div>
                        </main>
                    </div>
                </div>
            </div>
      <div class="row">
                
                <div class="col-sm py-4">
                    <form @submit.prevent="addTask">
                        <h2 class="text-center">Tasks</h2>
                        <div class="form-group">
                            <label for="task-name">
                                <span class="h5">Column Name</span>
                            </label>
                            <select class="custom-select" v-model="task.board_name">
                                <option value disabled>Select a Column</option>
                                <option
                                    v-for="board in boards"
                                    v-bind:key="board.id"
                                    :value="board.name"
                                >{{ board.name }}</option>
                            </select>
                        </div>
                        <div class="form-group">
                            <label for="task-name">
                                <span class="h5">Task Name</span>
                            </label>
                            <input type="text" class="form-control" v-model="task.name" />
                        </div>
                        <div class="form-group">
                            <label for="task-decription">
                                <span class="h5">Task Description</span>
                            </label>
                            <textarea class="form-control" v-model="task.description"></textarea>
                        </div>
                        <button type="submit" class="btn btn-primary btn-block">
                            <span class="h5">{{ addVariable}}</span>
                        </button>
                    </form>
                </div>
                <div class="col-sm py-4">
                    <form @submit.prevent="addBoard">
                        <h2 class="text-center">Column</h2>
                        <div class="form-group">
                            <label for="board-name">
                                <span class="h5">Column Name</span>
                            </label>
                            <input type="text" class="form-control" v-model="board.name" />
                        </div>
                        <button type="submit" class="btn btn-primary btn-block">
                            <span class="h5">{{ addVariable}}</span>
                        </button>
                    </form>
                </div>
            </div>

        </div>
    </div>
</template>

<script>
//  get current user's id from meta tag
Vue.prototype.$user_id = document
    .querySelector("meta[name='user-id']")
    .getAttribute("content");

import CreateTask from './CreateTask.vue'

export default {
    component: { CreateTask, },
    data() {
        return {
            addVariable:"Add",
            isModalVisible: false,
            boards: [],
            board: {
                id: "",
                name: ""
            },
            board_id: "",

            tasks: [],
            task: {
                id: "",
                board_name: "",
                name: "",
                description: ""
            },
            task_id: "",
            edit: false
        };
    },
    created() {
        //  get all boards
        this.fetchBoards();
        //  get all tasks
        this.fetchTasks();
    },
    methods: {
        //  board data handling

        showModal() {
        this.isModalVisible = true;
      },
      closeModal() {
        this.isModalVisible = false;
      },

        fetchBoards() {
            fetch("api/boards/" + this.$user_id)
                .then(res => res.json())
                .then(res => {
                    this.boards = res.data;
                });
        },
        deleteBoard(id) {
                fetch("api/board/" + id + "/" + this.$user_id, {
                    method: "delete"
                })
                    .then(res => res.json())
                    .then(data => {
                        this.fetchBoards();
                    })
                    .catch(error => console.log(error));
            
        },
        addBoard() {
            if (this.edit === false) {
                fetch("api/board/" + this.$user_id, {
                    method: "post",
                    body: JSON.stringify(this.board),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.board.name = "";
                        this.fetchBoards();
                    })
                    .catch(error => console.log(error));
            } else {
                fetch("api/board/" + this.$user_id, {
                    method: "put",
                    body: JSON.stringify(this.board),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.board.name = "";
                        window.location.reload();
                    })
                    .catch(error => console.log(error));
            }
        },
        editBoard(board) {
            this.addVariable ="Edit";
            this.edit = true;
            this.board.id = board.id;
            this.board.board_id = board.id;
            this.board.name = board.name;
            window.scrollTo(1200, 1200);
        },

        //  task data handling

        fetchTasks() {
            fetch("api/tasks/" + this.$user_id)
                .then(res => res.json())
                .then(res => {
                    this.tasks = res.data;
                });
        },
        deleteTask(id) {
          
                fetch("api/task/" + id + "/" + this.$user_id, {
                    method: "delete"
                })
                    .then(res => res.json())
                    .then(data => {
                        window.location.reload();
                        this.fetchTasks();
                    })
                    .catch(error => console.log(error));
            
        },
        addTask() {
            if (this.edit === false) {
                fetch("api/task/" + this.$user_id, {
                    method: "post",
                    body: JSON.stringify(this.task),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.task.board_name = "";
                        this.task.name = "";
                        this.task.description = "";
                        window.location.reload();
                    })
                    .catch(error => console.log(error));
            } else {
                fetch("api/task/" + this.$user_id, {
                    method: "put",
                    body: JSON.stringify(this.task),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.task.board_name = "";
                        this.task.name = "";
                        this.task.description = "";
                        window.location.reload();
                    })
                    .catch(error => console.log(error));
            }
        },
        editTask(task) {
            this.addVariable ="Edit";
            this.edit = true;
            this.task.id = task.id;
            this.task.task_id = task.id;
            this.task.board_name = task.board_name;
            this.task.name = task.name;
            this.task.description = task.description;
            window.scrollTo(1200, 1200);
        }
    }
};
</script>
