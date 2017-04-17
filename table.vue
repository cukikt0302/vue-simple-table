<template>
<div>
  <table :class="config.table_class">

    <thead>
      <tr>
        <th class="text-capitalize" v-for="(thead, index) in header" 
        :class="thead.class" @click="sort_colum(thead)">

          <span v-if="config.checkbox && index === 0">
            <input type="checkbox" v-model="checkall">
          </span>

          <span v-else>
            {{ thead.title }}&nbsp;
            <i v-if="thead.key" class="fa" :class="{
              'fa-caret-up' : toggle_sort === 1 && thead.key == column_sort, 
              'fa-caret-down' : toggle_sort === -1 && thead.key == column_sort
            }"></i>
          </span>

        </th>
      </tr>
    </thead>

    <tbody>
      <tr v-for="(row, r) in data" :class="{'tr-active': context_menu_row.id === row.id}"
      @contextmenu.prevent="$refs.ctxMenu.open($event, row)"
      v-if="row.id !== toggle_edit_row">

        <td v-for="(item, k, c) in row" :key="item.id" :class="body[c].class">

          <div v-if="body[c].hasOwnProperty('method') && has_ev(body[c].method)">

            <span v-if="check_ev(body[c].method, 'click')" 
            @click="click(check_ev(body[c].method, 'click'), row, r, c, k)">
              {{ item }}
            </span>

            <span v-else-if="check_ev(body[c].method, 'dblclick')"
            @dblclick="dblclick(check_ev(body[c].method, 'dblclick'), row, r, c, k)">
              <span v-if="action.r === r && action.c === c">

                <slot v-if="check_slot(body[c].method)" 
                :name="check_slot(body[c].method)"
                :row="row" :r="r" :c="c" :k="k"></slot>

                <span v-else>Can't find <b>Slot</b></span>

              </span>

              <span v-else>{{ item }}</span>
            </span>

            <span v-else-if="check_ev(body[c].method, 'hover')"
            @mouseover="hover(check_ev(body[c].method, 'hover'), row, r, c, k)">
              {{ item }}
            </span>

          </div>

          <div v-else>
            <span v-if="config.checkbox && k == 'id' && c === 0">
              <input type="checkbox" :value="item" v-model="checkbox">
            </span>
            <span v-else>{{ item }}</span>
          </div>

        </td>

      </tr>

      <tr v-else class="tr-active">
        <td :colspan="header.length">
          <slot name="edit_row" :row="find_by_id(toggle_edit_row)"></slot>
        </td>
      </tr>
    </tbody>

  </table>

  <context-menu v-if="config.edit_row" id="context-menu" ref="ctxMenu"
  @ctx-open="onCtxOpen" @ctx-cancel="resetCtxLocals" @ctx-close="onCtxClose">
    <li @click.prevent="set_edit_row">Edit</li>
    <li @click.prevent="remove_row">Delete</li>
  </context-menu>

  <context-menu v-else style="display: none" id="context-menu" ref="ctxMenu"></context-menu>
</div>
</template>

<style>
  .tr-active { border-left: 1px solid #1a2226; }
  .ctx-menu { font-size: 1em !important; }
  #context-menu li {
    padding: 3px 20px;
    font-size: .9em;
    font-weight: 600;
    line-height: 1.5;
    cursor: context-menu;
  }
  #context-menu li:hover {
    color: #2b2d2f;
    text-decoration: none;
    background-color: #f5f5f5;
  }
</style>

<script>
export default {
  name: 'post-general',

  props: {
    config: { default() { return false } },
    header: { default() { return {} } },
    body: { default() { return {} } },
    action: { default() { return {} } },
    data: { default() { return {} } }
  },

  data() {
    return {
      checkbox: [],
      context_menu_row: '',
      toggle_edit_row: false,
      toggle_sort: -1,
      column_sort: ''
    }
  },

  computed: {
    checkall: {
      get: function () {
        return this.data 
          ? this.checkbox.length == this.data.length 
          : false;
      },

      set: function (value) {
        var checkbox = [];
        if (value) {
          this.data.forEach(function (item) {
            checkbox.push(item.id);
          });
        }
        this.checkbox = checkbox;
      }
    }
  },

  watch: {
    'checkbox': function(newVal) {
      this.$emit('set_checkbox', newVal)
    }
  },

  methods: {
    sort_colum(item) {
      if ( !this.config.sort || !item.hasOwnProperty('key')) return
      this.toggle_sort = -(1) * this.toggle_sort
      this.column_sort = item.key
      this.data.sort((a, b) => {
        let nameA = a[item.key].toLowerCase(), nameB = b[item.key].toLowerCase()
        return (nameA == nameB ? 0 : nameA < nameB ? -1 : 1) * (this.toggle_sort)
      })
    },

    has_ev(item) {
      if (item['click'] != undefined) {
        return true;
      } else if (item['dblclick'] != undefined) {
        return true;
      } else if (item['hover'] != undefined) {
        return true;
      } else {
        return false;
      }
    },

    check_ev(item, str) {
      return Object.keys(item)[0] == str ? item[Object.keys(item)[0]] : false;
    },

    check_slot(item) {
      return Object.keys(item)[1] == 'slot' ? item[Object.keys(item)[1]] : false;
    },

    click(Emit, row, r, c, k) { this.$emit(Emit, row, r, c, k) },
    dblclick(Emit, row, r, c, k) { this.$emit(Emit, row, r, c, k) },
    hover(Emit, row, r, c, k) { this.$emit(Emit, row, r, c, k) },

    onCtxOpen(row) { this.context_menu_row = row },
    onCtxClose(row) {},
    resetCtxLocals(row) { this.context_menu_row = {} },

    set_edit_row() {// context_menu_row when right click context menu onCtxOpen()
      this.toggle_edit_row = this.context_menu_row.id
    },

    cancel_edit_row(clearContext) { 
      this.toggle_edit_row = false
      if ( !clearContext) this.context_menu_row = {}
    },

    remove_row() {
      if (this.checkbox.length > 0) {
        if (confirm(
          `Remove ${this.checkbox.length} ${(
            this.checkbox.length === 1 ? 'row' : 'rows'
          )} has checked ?`
        )) {
          for (let i = this.checkbox.length -1; i >= 0; i--) {
            let obj = this.data.filter(ob => ob.id === this.checkbox[i])[0]
            this.data.splice(this.data.indexOf(obj), 1)
          }
        }
      } else {
        let index = this.data.indexOf(this.context_menu_row),
            confirmStr = `Remove row: ${(
              index + 1 === 1 ? '1 ST' 
                : index + 1 === 2 ? '2 ND' 
                : index + 1 === 3 ? '3 RD' 
                : (index + 1) + ' TH'
            )}`;
        if (confirm(confirmStr)) { this.data.splice(index, 1) }
      }
      this.context_menu_row = {}
      this.checkbox = []
    },

    find_by_id(id) { return this.data.find(obj => obj.id === id) }
  },

  components: {
    'context-menu': require('vue-context-menu')
  }
}
</script>
