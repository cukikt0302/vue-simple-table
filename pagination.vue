<template>
<div>
  <ul :class="ul_class" v-if="total_page*per_page*current_page > 0">

    <li :disabled="!showFirst" :class="{'disabled' : !showFirst}">
      <a @click.prevent="firstPage" href="#">
        <slot v-if="$slots.firstPage" name="firstPage"></slot>
        <span v-else>First</span>
      </a>
    </li>

    <li :disabled="!showPrev" :class="{'disabled' : !showPrev}">
      <a @click.prevent="prevPage" href="#">
        <slot v-if="$slots.prevPage" name="prevPage"></slot>
        <span v-else>{{ current_page - 1}}</span>
      </a>
    </li>

    <li class="active" :class="current" @click="set_toggle_input()">
      <span style="position: relative;">{{ current_page }}</span>
      <div v-if="toggle_input" :style="`position: absolute; ${placement}: -10px;`">

        <form @submit.prevent="change_current_page()">
          <input type="number" v-focus v-selected 
            @blur="toggle_input = false" @keydown.esc="toggle_input = false"
            style="width: 80px; display: inline-block;" 
            required min="1" :max="total_page"
            :value="current_page" v-model.number="input_value">
            <small style="color: auto;">Max: {{ total_page }}</small>
        </form>

      </div>
    </li>

    <li :disabled="!showNext" :class="{'disabled' : !showNext}">
      <a @click.prevent="nextPage" href="#">
        <slot v-if="$slots.nextPage" name="nextPage"></slot>
        <span v-else>{{ current_page + 1}}</span>
      </a>
    </li>

    <li :disabled="!showLast" :class="{'disabled' : !showLast}">
      <a @click.prevent="lastPage" href="#">
        <slot v-if="$slots.lastPage" name="lastPage"></slot>
        <span v-else>Last</span>
      </a>
    </li>

  </ul>
</div>
</template>

<script>
export default {
  name: 'pagination',

  props: {
    placement: {// top or bottom
      default() { return 'bottom' }
    },
    ul_class: {
      default() { return 'pagination pagination-sm' }
    },
    current: {
      default() { return 'active' }
    },
    total_page: {
      default() { return 0} 
    },
    per_page: {
      default() { return 0 } 
    }
  },

  data() {
    return {
      current_page: 1,
      toggle_input: false,
      input_value: ''
    }
  },

  computed: {
    showNext() {
      return this.current_page < this.total_page
    },

    showPrev() {
      return this.current_page > 1
    },

    showFirst() {
      return (this.current_page - 1) > 1
    },

    showLast() {
      return (this.current_page + 1) < this.total_page
    },
  },

  methods: {
    nextPage() {
      if (this.showNext)
        this.current_page += 1
    },

    prevPage() {
      if (this.showPrev)
        this.current_page -= 1
    },

    firstPage() {
      if (this.showFirst)
        this.current_page = 1
    },

    lastPage() {
      if (this.showLast)
        this.current_page = this.total_page
    },

    set_toggle_input() {
      this.input_value = this.current_page
      this.toggle_input = !this.toggle_input
    },

    change_current_page() {
      this.toggle_input = false
      if (this.current_page !== this.input_value)
      this.current_page = this.input_value
    }
  },

  watch: {
    current_page(newVal) {
      this.$emit('current_page', newVal)
    }
  }
}
</script>
