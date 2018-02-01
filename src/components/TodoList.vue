<template>
  <div>
    <transition-group name="flip-list" tag="ul" class="todo-ul">
      <li class="todo-li" v-for="item in items" :key="item.id" :class="getStatusClassName(item.status)">
        <icon
          class="status"
          :name="getStatusIcon(item.status)"
          @click="switchStatus(item.id)"
        ></icon>
        <input class="todo-title" v-model="item.title"/>
        <span class="actions" @mouseout="sortByPriority">
          <icon
            class="priority"
            name="bookmark"
            :class="getPriorityClassName(item.priority)"
            @click="swithPriority(item.id)"
          ></icon>
          <icon
            class="delete"
            name="trash-2"
            @click="removeItem(item.id)"
          ></icon>
        </span>
      </li>
      <li class="todo-li" key="new">
        <input class="todo-title" v-model.trim="newTitle" @keypress="onNewTitleKeyPress"/>
      </li>
    </transition-group>
  </div>
</template>

<script>
import Vue from 'vue'
import Icon from './Icon'
import { STATUS, PRIORITY, LS_ITEMS_KEY } from '../constants'

function getLSKey(key, date) {
  date = date || new Date()
  const dateStr = `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`
  return key + '?date=' + dateStr
}

function getLS(key, defaultValue, date) {
  try {
    key = getLSKey(key, date)
    const value = window.localStorage.getItem(key)
    if (value === 0 || value) {
      return value
    } else {
      return defaultValue
    }
  } catch (e) {
    console.error(e)
    return defaultValue
  }
}

function setLS(key, value) {
  try {
    key = getLSKey(key)
    window.localStorage.setItem(key, value)
  } catch (e) {
    console.error(e)
  }
}

export default {
  name: 'TodoList',

  components: {
    Icon
  },

  data() {
    return {
      newTitle: '',
      items: []
    }
  },

  methods: {
    getStatusIcon(status) {
      switch (status) {
        case STATUS.TODO: return 'circle'
        case STATUS.DOING: return 'clock'
        case STATUS.FINISHED: return 'check-circle'
      }
    },
    getStatusClassName(status) {
      switch (status) {
        case STATUS.TODO: return 'todo'
        case STATUS.DOING: return 'doing'
        case STATUS.FINISHED: return 'finished'
      }
    },
    getPriorityClassName(priority) {
      switch (priority) {
        case PRIORITY.HIGH: return 'high'
        case PRIORITY.MEDIUM: return 'medium'
        case PRIORITY.LOW: return 'low'
      }
    },
    onNewTitleKeyPress(e) {
      // enter
      if (e.keyCode === 13 && this.newTitle) {
        const newItem = {
          id: (new Date()).getTime(),
          title: this.newTitle,
          priority: PRIORITY.MEDIUM,
          status: STATUS.TODO
        }
        this.items.push(newItem)
        this.newTitle = ''
        this.sortByPriority()
        setLS(LS_ITEMS_KEY, JSON.stringify(this.items))
      }
    },
    sortByPriority() {
      this.items.sort((a, b) => {
        return a.priority - b.priority || a.id - b.id
      })
    },
    switchStatus(id) {
      const idx = this.items.findIndex((x) => x.id === id)
      if (idx < 0) {
        return
      }
      const item = this.items[idx]
      switch (item.status) {
        case STATUS.TODO:
          item.status = STATUS.DOING
          break
        case STATUS.DOING:
          item.status = STATUS.FINISHED
          break
        case STATUS.FINISHED:
          item.status = STATUS.TODO
          break
      }
      Vue.set(this.items, idx, item)
      setLS(LS_ITEMS_KEY, JSON.stringify(this.items))
    },
    swithPriority(id) {
      const idx = this.items.findIndex((x) => x.id === id)
      if (idx < 0) {
        return
      }
      const item = this.items[idx]
      item.priority = (item.priority + 1) % Object.keys(PRIORITY).length
      Vue.set(this.items, idx, item)
      setLS(LS_ITEMS_KEY, JSON.stringify(this.items))
    },
    removeItem(id) {
      this.items = this.items.filter((item) => item.id !== id)
      setLS(LS_ITEMS_KEY, JSON.stringify(this.items))
    }
  },

  beforeMount() {
    try {
      const todayItems = JSON.parse(getLS(LS_ITEMS_KEY))
      this.items = todayItems
    } catch (e) {
      const yesterday = new Date()
      yesterday.setDate(yesterday.getDate() - 1)
      const yesterdayItems = JSON.parse(getLS(LS_ITEMS_KEY, '[]', yesterday))
      this.items = yesterdayItems.filter((item) => item.status !== STATUS.FINISHED)
    }
  },

  mounted() {
    this.sortByPriority()
  }
}
</script>

<style lang="stylus" scoped>
@require '../palette.styl'

.flip-list-move
  transition transform .2s

.todo-ul
  width 50%
  max-width 480px
  padding-left 0
  margin-left auto
  margin-right auto
  font-size 24px
  list-style-type none

.todo-li
  position relative
  text-align left
  border-bottom 1px solid color-grey-light

.todo-title
  width 100%
  padding 10px 0
  font-size 24px
  border 1px solid transparent
  outline none

.status
.delete
.priority
  font-size 24px
  cursor pointer

.delete
  color color-red-light

.priority
  &.high
    color color-purple
  &.medium
    color color-yellow-dark
  &.low
    color color-green-dark

.status
  position absolute
  left -30px
  top 13px

.todo-li.finished .todo-title
  color color-grey-light

.actions
  position absolute
  top -10px
  right -60px
  padding 20px 10px
  opacity .15
  transition opacity .2s

.todo-li:hover .actions
.actions:hover
  transition none
  opacity 1

@media screen and (max-width: 600px)
  .todo-ul
    max-width 400px

@media screen and (max-width: 480px)
  .todo-ul
    max-width 320px
</style>
