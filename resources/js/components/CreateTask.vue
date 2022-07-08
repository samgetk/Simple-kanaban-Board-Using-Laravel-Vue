<script>
//get current user's id from meta tag
Vue.prototype.$user_id = document
    .querySelector("meta[name='user-id']")
    .getAttribute("content");

export default {
     name: 'CreateTask',
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

        showModal(task) {
        this.editTask(task);
        this.isModalVisible = true;
      },
      closeModal() {
        this.isModalVisible = false;
      },
      close() {
        this.$emit('close');
      },

        fetchBoards() {
            fetch("api/boards/" + this.$user_id)
                .then(res => res.json())
                .then(res => {
                    this.boards = res.data;
                });
        },
        deleteBoard(id) {
            if (confirm("Are you sure?")) {
                fetch("api/board/" + id + "/" + this.$user_id, {
                    method: "delete"
                })
                    .then(res => res.json())
                    .then(data => {
                        this.fetchBoards();
                    })
                    .catch(error => console.log(error));
            }
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
            if (confirm("Are you sure?")) {
                fetch("api/task/" + id + "/" + this.$user_id, {
                    method: "delete"
                })
                    .then(res => res.json())
                    .then(data => {
                        window.location.reload();
                        this.fetchTasks();
                    })
                    .catch(error => console.log(error));
            }
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

<template>
  <transition name="modal-fade">
        <div style="max-width: 100px; max-height: 100px">

    <div class="modal-backdrop modal">
      <div class="modal"
        role="dialog"
        aria-labelledby="modalTitle"
        aria-describedby="modalDescription"
      >
        <header
          class="modal-header"
          id="modalTitle"
        >
          <slot name="header">
            Create Columns or Tasks
          </slot>
          <button
            type="button"
            class="btn-close"
            @click="close"
            aria-label="Close modal"
          >
            x
          </button>
        </header>

        <section
          class="modal-body"
          id="modalDescription"
        >
          <slot name="body">
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
          </slot>
        </section>

        <footer class="modal-footer">
          <slot name="footer">
          </slot>
          <button
            type="button"
            class="btn-green"
            @click="close"
            aria-label="Close modal"
          >
            Close 
          </button>
        </footer>
      </div>
    </div>
    </div>
  </transition>
</template>

<style>
  .modal-backdrop {
    position: fixed;
    top: 50;
    bottom: 50;
    left: 50;
    right: 50;
    background-color: rgba(0, 0, 0, 0.3);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .modal {
    background: #FFFFFF;
    box-shadow: 2px 2px 20px 1px;
    overflow-x: auto;
    display: flex;
    flex-direction: column;
  }

  .modal-header,
  .modal-footer {
    padding: 15px;
    display: flex;
  }

  .modal-header {
    position: relative;
    border-bottom: 1px solid #eeeeee;
    color: #4AAE9B;
    justify-content: space-between;
  }

  .modal-footer {
    border-top: 1px solid #eeeeee;
    flex-direction: column;
  }

  .modal-body {
    position: relative;
    padding: 20px 10px;
  }

  .btn-close {
    position: absolute;
    top: 0;
    right: 0;
    border: none;
    font-size: 20px;
    padding: 10px;
    cursor: pointer;
    font-weight: bold;
    color: #4AAE9B;
    background: transparent;
  }

  .btn-green {
    color: white;
    background: #4AAE9B;
    border: 1px solid #4AAE9B;
    border-radius: 2px;
  }

  .modal-fade-enter,
  .modal-fade-leave-to {
    opacity: 0;
  }

  .modal-fade-enter-active,
  .modal-fade-leave-active {
    transition: opacity .5s ease;
  }
</style>