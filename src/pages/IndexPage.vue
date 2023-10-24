<template>
  <q-page>
    <div class="column" >
      <q-scroll-area
        :style="`height: ${ this.$q.screen.height - 180 }px; max-width: 100%;`"
        v-if="todos.length > 0">
        <q-list class="q-ma-sm">
          <template
            v-for="(todo, i) in todos"
            :key="i">
            <q-item
              clickable
              @click="changeStatus(i, true)"
              v-ripple
              :class="`q-ma-sm ${todo.isChecked ? 'done bg-blue-1' : ''}`"
              style="border: 1px solid #aeaeae; border-radius: 5px">
              <q-item-section avatar>
                <q-checkbox v-model="todo.isChecked" @update:model-value="changeStatus(i, false)" class="no-pointer-events"/>
              </q-item-section>
              <q-item-section>
                <q-item-label>{{ todo.label }}</q-item-label>
              </q-item-section>
              <q-item-section side v-if="todo.isChecked">
                <q-btn color="negative" @click="onDelete(i)" icon="mdi-delete" no-caps flat />
              </q-item-section>
            </q-item>
          </template>
        </q-list>
      </q-scroll-area>
      <div class="flex flex-center items-center" :style="`height: ${this.$q.screen.height - 180}px`" v-else>
        <div class="column items-center">
          <q-icon name="mdi-check" :size="`${this.$q.screen.xs ? '5em' :'10em'}`" color="primary"/>
          <span :class="`${this.$q.screen.xs ? 'text-h6' :'text-h5'} text-grey-7`">No Todo For Now</span>
        </div>
      </div>
      <q-page-sticky position="bottom-right" :offset="[20, 20]">
        <!-- <q-btn @click="prompt = !prompt" fab icon="add" color="primary" /> -->
        <q-btn @click="onCrated" fab icon="add" color="primary" />
      </q-page-sticky>
    </div>

    <q-dialog v-model="prompt" persistent>
        <q-card style="min-width: 350px">
          <q-card-section>
            <div class="text-h6">Add Todo</div>
          </q-card-section>
          <q-card-section class="q-pt-none">
            <q-form @submit="onSubmit">
              <div class="column">
                <q-input
                  outlined
                  v-model="todo"
                  autofocus
                  placeholder="Label Todo"
                  :rules="[
                    val => val.length > 0 || 'Please Add Label'
                  ]"/>
              </div>
              <div class="row justify-end q-gutter-sm">
                <q-btn color="primary" flat no-caps label="Cancel" v-close-popup />
                <q-btn color="primary" no-caps label="Add" type="submit" />
              </div>
            </q-form>
          </q-card-section>
        </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import Swal from 'sweetalert2'
export default {
  data () {
    return {
      todo: '',
      prompt: false,
      todos: []
    }
  },
  created () {
    this.getTodo()
  },
  methods: {
    getTodo () {
      const data = this.$q.localStorage.getItem('todo')
      if (!data) {
        this.setTodo()
        this.getTodo()
      } else {
        this.todos = data
        // console.log(this.todos)
      }
    },
    setTodo () {
      this.$q.localStorage.set('todo', this.todos)
    },
    changeStatus (index, isChange) {
      this.todos[index].isChecked = isChange ? !this.todos[index].isChecked : isChange
    },
    async onCrated () {
      const action = await Swal.fire({
        title: 'Enter your Todo',
        input: 'text',
        inputLabel: 'Your todo',
        inputValue: '',
        showCancelButton: true,
        inputValidator: (value) => {
          if (!value) {
            return 'You need to write something!'
          }
        }
      })
      // console.log(action)
      if (action.isConfirmed) {
        this.todos.push({
          id: this.todos.length,
          isChecked: false,
          label: action.value
        })
        this.todo = ''
        Swal.fire(
          'Success!',
          'Your todo has been added.',
          'success'
        )
        this.setTodo()
      }
    },
    onSubmit () {
      this.todos.push({
        id: this.todos.length,
        isChecked: false,
        label: this.todo
      })
      this.todo = ''
      this.prompt = !this.prompt
      Swal.fire(
        'Success!',
        'Your todo has been added.',
        'success'
      )
      this.setTodo()
    },
    async onDelete (index) {
      const oldTodo = this.todos
      await Swal.fire({
        title: 'Warning !',
        text: 'Do you want to continue',
        icon: 'warning',
        showCancelButton: true,
        cancelButtonText: 'Cancel',
        confirmButtonText: 'Yes'
      }).then(function (isConfirm) {
        if (isConfirm) {
          oldTodo.splice(index, 1)
          Swal.fire(
            'Deleted!',
            'Your todo has been deleted.',
            'success'
          )
        }
      })
      this.todos = oldTodo
      // console.log(this.todos)
      this.setTodo()
    }
  }
}
</script>
