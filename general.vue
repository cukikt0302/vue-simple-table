<template>
  <div class="row">
    <loading>

      <div class="col-md-12">
        <div class="box">
          <div class="box-body">

            <div class="row">
              <my-toolbar :header="header" @hidden_column="hidden_column" @search="search"></my-toolbar>
            </div>

          </div>
        </div>
      </div>

      <div class="col-md-12">

        <div class="box">

          <div class="box-body no-padding table-responsive">

            <my-table ref="my_table" :hidden_column="action.hidden_column"
              :header="header" :body="body" :data="data" :h="action.h" :v="action.v" :config="{
                table_class: 'table table-striped', 
                head_class: '',
                body_class: '',
                checkbox: true, edit_row: true, sort: true
              }" 
              @edit="edit" @set_checkbox="set_checkbox" @edit_row="edit_row" 
              contextmenu_edit_text="Edit text" contextmenu_remove_text="Delete text">

              <!-- Slot for Edit Row -->
              <template slot="edit_row" scope="res">
                <form @submit.prevent="submit_row(res.row)" role="form" 
                @keyup.esc="$refs.my_table.cancel_edit_row">

                  <div class="col-md-5 col-md-offset-1">
                    <div class="form-group">
                      <label for="title">Title</label>
                      <input type="text" class="form-control input-sm" v-focus id="title" 
                      placeholder="Title" v-model="action.edit_row.title">
                    </div>

                    <div class="form-group">
                      <label for="create">Created</label>

                      <dropdown class="form-group">
                        <div class="input-group col-xs-12">
                          <input type="text" class="form-control input-sm" id="create" placeholder="Created" 
                          v-model="action.edit_row.create" data-role="trigger">
                          <label for="create" class="input-group-addon">
                            <i class="fa fa-calendar-o"></i>
                          </label>
                        </div>
                        <template slot="dropdown">
                          <li>
                            <date-picker v-model="action.edit_row.create"></date-picker>
                          </li>
                        </template>
                      </dropdown>
                    </div>
                  </div>

                  <div class="col-md-5">

                    <div class="form-group">
                      <label for="category">Category</label>
                      <div class="input-group col-xs-12">
                        <select name="category" id="category" class="form-control input-sm" 
                        v-model="action.edit_row.category">
                          <option :value="'Default'">Default</option>
                          <option :value="'Internet'">Internet</option>
                          <option :value="'Cooking'">Cooking</option>
                        </select>
                        <label for="category" class="input-group-addon">
                          <i class="fa fa-folder-open-o"></i>
                        </label>
                      </div>
                    </div>

                    <div class="form-group">
                      <label for="author">Author</label>
                      <div class="input-group col-xs-12">
                        <input type="text" class="form-control input-sm" 
                        id="author" placeholder="Author" 
                        v-model="action.edit_row.author">
                        <label for="author" class="input-group-addon">
                          <i class="fa fa-user-o"></i>
                        </label>
                      </div>
                    </div>
                  </div>

                  <div class="col-md-10 col-md-offset-1">
                    <div class="input-group">
                      <button type="submit" class="btn btn-flat btn-sm btn-primary">Submit</button>
                      <button type="button" class="btn btn-flat btn-sm btn-default" 
                      @click.prevent="$refs.my_table.cancel_edit_row()">Cancel</button>
                      <a :href="`#/post/edit/${action.edit_row.title}`" 
                      class="btn btn-flat btn-sm btn-success">Full edit</a>
                    </div>
                  </div>
                </form>
              </template>
              <!-- Slot for Edit Row -->

              <!-- Slot for Edit Cell -->
              <template slot="input" scope="res">
                <input type="text" v-focus v-selected v-model="action.edit_value"
                class="input-sm form-control" @blur="reset_edit(res)" 
                @keydown.enter="submit(res)" @keydown.esc="reset_edit(res)">
              </template>

              <template slot="select" scope="res">
                <select v-focus class="input-sm form-control" 
                v-model="action.edit_value" @blur="submit(res)"
                @keydown.enter="submit(res)" @keydown.esc="reset_edit(res)">
                  <option :value="'Default'">Default</option>
                  <option :value="'Internet'">Internet</option>
                  <option :value="'Cooking'">Cooking</option>
                </select>
              </template>

              <!-- Edit cell calendar -->
              <template slot="calendar" scope="res">
                <dropdown class="form-group">
                  <div class="input-group">
                    <input type="text" v-focus v-selected v-model="action.edit_value"
                    class="form-control input-sm" placeholder="Created" 
                    @keydown.enter="submit(res)" @keydown.esc="reset_edit(res)" data-role="trigger">
                  </div>
                  <template slot="dropdown">
                    <li>
                      <date-picker v-model="action.edit_value"></date-picker>
                    </li>
                  </template>
                </dropdown>
              </template>
              <!-- Edit cell calendar -->

              <template slot="textarea" scope="res">
                <textarea rows="1" v-focus class="input-sm form-control" 
                @blur="reset_edit(res)" @keydown.enter="submit(res)"
                @keydown.esc="reset_edit(res)" v-model="action.edit_value"></textarea>
              </template>
              <!-- /Slot for Edit Cell -->

              <!-- Slot for Context Menu -->
              <template slot="list_context" scope="res">
                <li @click.prevent="say(res.row)">
                  <i class="fa fa-th"></i> <span>Your crazy menu</span>
                </li>
              </template>
              <!-- /Slot for Context Menu -->

            </my-table>

            <div class="col-md-12 text-right">
              <!-- <my-pagination ul_class="pagination" :total_page="5" 
              :per_page="2" @current_page="current_page" placement="top">
                <span slot="nextPage">&rarr;</span>
                <span slot="prevPage">&larr;</span>
                <span slot="firstPage">&laquo;</span>
                <span slot="lastPage" title="5">&raquo;</span>
              </my-pagination> -->
              <pagination 
                v-model="pagination.currentPage" 
                :totalPage="pagination.totalPage" 
                :maxSize="pagination.maxSize">
              </pagination>
            </div>

          </div>
        </div>

      </div>

    </loading>
  </div>
</template>

<style>
  .cursor-pointer { cursor: pointer; }
  .btn-app { margin: 0; }
</style>

<script>
/*
 * create: Nguyen Ke Nhat
 * Date: 04/14/2017
 * Github: https://github.com/cukikt0302/vue-simple-table
*/
  import { Dropdown, DatePicker, Pagination } from 'uiv'

  export default {
    name: 'post-general',

    data() {
      return {
        header: [// key: name of column for sort, not sort not set this
          { title: 'ID', key: 'id', class: 'text-center cursor-pointer' },
          { title: 'Title', key: 'title', class: 'cursor-pointer'},
          { title: 'Category', key: 'category', class: 'cursor-pointer'},
          { title: 'Create At', key: 'create', class: 'text-center cursor-pointer'},
          { title: 'Author', key: 'author', class: 'cursor-pointer'}
        ],

        body: [
          { class: 'text-center' },
          { method: {dblclick: 'edit', slot: 'textarea'}  },
          { method: {dblclick: 'edit', slot: 'select'} },
          { class: 'text-center', method: {dblclick: 'edit', slot: 'calendar'} },
          { method: {dblclick: 'edit', slot: 'input'} }
        ],

        data: [// required named is id property - if using checkbox
          {
            id: 1,
            title: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit.',
            category: 'Internet',
            create: '2017-02-03',
            author: 'administrator'
          },
          {
            id: 2,
            title: 'Sed hendrerit ex quis erat commodo condimentum.',
            category: 'Cooking',
            create: '2017-09-15',
            author: 'moderator'
          },
          {
            id: 3,
            title: 'Maecenas egestas ligula et est accumsan suscipit.',
            category: 'Default',
            create: '2016-11-21',
            author: 'guest'
          },
          {
            id: 4,
            title: 'Quisque sit amet nibh venenatis, mollis metus a, malesuada nibh.',
            category: 'Internet',
            create: '2016-04-29',
            author: 'alibaba'
          },
          {
            id: 5,
            title: 'Maecenas egestas ligula et est accumsan suscipit.',
            category: 'Default',
            create: '2014-05-22',
            author: 'cristano_ronaldo'
          },
          {
            id: 6,
            title: 'Nullam vel dui faucibus, venenatis odio sed, ultrices velit.',
            category: 'Cooking',
            create: '2017-08-15',
            author: 'leonel_messi'
          },
          {
            id: 7,
            title: 'Integer dictum elit quis metus convallis, eget euismod arcu tristique.',
            category: 'Internet',
            create: '2016-06-13',
            author: 'ricardo_kaka'
          },
          {
            id: 8,
            title: 'Suspendisse ac nulla sit amet nisl egestas pharetra ac eu arcu.',
            category: 'Internet',
            create: '2016-11-01',
            author: 'ronaldinho'
          },
          {
            id: 9,
            title: 'Vivamus facilisis felis sed ante euismod congue.',
            category: 'Default',
            create: '2015-11-21',
            author: 'zlatan_ibrahimovic'
          },
          {
            id: 10,
            title: 'Pellentesque non ipsum id felis fringilla aliquam sit amet id diam.',
            category: 'Cooking',
            create: '2017-11-21',
            author: 'andrey_arshavin'
          }
        ],

        typeahead_search: '',

        action: {
          h: false,// row index
          v: false,// col index
          edit_value: '',// v-model for all edit model
          checkbox: [],
          edit_row: {// value of edit_row in my-table component
            id: '',
            title: '',
            category: '',
            create: '',
            author: ''
          },
          hidden_column: []
        },

        pagination: {
          currentPage: 1,
          totalPage: 20,
          maxSize: 5
        }
      }
    },

    mounted() {
      //this.$store.commit('set_page_header', {
      //  title: 'Post',
      //  desc: 'General'
      //})

      this.set_data()
    },

    methods: {
      set_data() {
        setTimeout(() => {
          this.$store.state.loading = true
        }, 2000)
        this.$store.state.loading = false
      },

      say(row) {
        alert('Open console and looking for response object!')
        console.log(row);
      },

      edit(row, v, h, k) {
        this.action.v = v;// set edit vertical index
        this.action.h = h;// set edit horizontal index
        this.action.edit_value = row[k];// set value for model
      },

      submit(res) {// console.log(res) => current row object
        if (res.row[res.k] != this.action.edit_value) {
          let after_update = JSON.parse(JSON.stringify(res.row))
          this.$refs.my_table.set_row_undo(after_update, res.c)
          res.row[res.k] = this.action.edit_value
        }
        this.reset_edit()
      },

      reset_edit() {
        this.action.v = false;
        this.action.h = false;
      },

      set_checkbox(item) { this.action.checkbox = item },

      edit_row(row) {
        for (let k in row) {
          this.action.edit_row[k] = row[k]
        }
      },

      submit_row(row) {
        let after_update = JSON.parse(JSON.stringify(row))
        for (let key in this.action.edit_row) {
          row[key] = this.action.edit_row[key]
        }
        this.$refs.my_table.set_row_undo(after_update)
        this.$refs.my_table.cancel_edit_row(true)// close edit slot
      },

      hidden_column(res) {
        this.action.hidden_column = res
      },

      search(res) {
        this.typeahead_search = res
      },

      current_page(num) { this.action.pagination.current_page = num }

    },

    components: {
      'my-table': require('../../general/table.vue'),
      'my-pagination': require('../../general/pagination.vue'),
      'my-toolbar': require('./general/toolbar.vue'),
      Dropdown, DatePicker, Pagination
    }
  }
</script>
