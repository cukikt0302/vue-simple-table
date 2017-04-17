<template>
	<div class="row">
		<div class="col-md-12">
			<div class="box box-primary">
				<div class="box-body no-padding table-responsive">
					
					<my-table ref="my_table"
						:header="header" :body="body" :data="data" :action="action"
						:config="{
							table_class: 'table table-striped', 
							checkbox: true, 
							edit_row: true,
							sort: true
						}" 
						@edit="edit" @set_checkbox="set_checkbox">
						
						<template slot="edit_row" scope="res">
							<form @submit.prevent="submit_row(res.row)" role="form" 
							@keyup.esc="$refs.my_table.cancel_edit_row">

								<div class="col-md-4 col-md-offset-2">
									<div class="form-group">
										<label for="title">Title</label>
										<input type="text" class="form-control input-sm" v-focus  id="title" 
										placeholder="Title" v-model="action.edit_row.title = res.row.title">
									</div>
								
									<div class="form-group">
										<label for="create">Created</label>
										<input type="text" class="form-control input-sm" id="create" 
										placeholder="Created" v-model="action.edit_row.create = res.row.create">
									</div>
								</div>
								<div class="col-md-4">
								
									<div class="form-group">
										<label for="category">Category</label>
										<select name="category" id="category" class="form-control input-sm" 
										v-model="action.edit_row.category = res.row.category">
											<option :value="'Default'">Default</option>
											<option :value="'Internet'">Internet</option>
											<option :value="'Cooking'">Cooking</option>
										</select>
									</div>

									<div class="form-group">
										<label for="author">Author</label>
										<input type="text" class="form-control input-sm" id="author" placeholder="Author"
										:value="res.row.author" v-model="action.edit_row.author = res.row.author">
									</div>
								</div>
							
								<div class="col-md-8 col-md-offset-2">
									<button type="submit" class="btn btn-flat btn-sm btn-primary">Submit</button>
									<button type="button" class="btn btn-flat btn-sm btn-default" 
									@click.prevent="$refs.my_table.cancel_edit_row()">Cancel</button>
								</div>
							</form>
						</template>

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

						<template slot="textarea" scope="res">
							<textarea rows="1" v-focus class="input-sm form-control" 
							@blur="reset_edit(res)" @keydown.enter="submit(res)"
							@keydown.esc="reset_edit(res)" v-model="action.edit_value"></textarea>
						</template>

					</my-table>

					<div class="col-md-12 text-right">
						<my-pagi :total_page="5" :per_page="2" @current_page="current_page">
							<span slot="nextPage">&rsaquo;</span>
							<span slot="prevPage">&lsaquo;</span>
							<span slot="firstPage">&laquo;</span>
							<span slot="lastPage">&raquo; 5</span>
						</my-pagi>
					</div>

					<div class="col-md-12">
						<pre>{{ action }}</pre>
					</div>

				</div>
			</div>
		</div>
	</div>
</template>

<style>
	.cursor-pointer { cursor: pointer; }
</style>

<script>
/*
 * create: Nguyen Ke Nhat
 * Date: 04/14/2017
 * Github: https://github.com/cukikt0302/vue-simple-table
*/
	export default {
		name: 'post-general',

		data() {
			return {
				header: [// key: name of column for sort, not sort not set this
					{ title: 'ID', class: 'text-center' },
					{ title: 'Title', key: 'title', class: 'cursor-pointer'},
					{ title: 'Category', key: 'category', class: 'cursor-pointer'},
					{ title: 'Create', key: 'create', class: 'text-center cursor-pointer'},
					{ title: 'Author', key: 'author', class: 'cursor-pointer'}
				],

				body: [
					{ class: 'text-center' },
					{ method: {dblclick: 'edit', slot: 'textarea'}  },
					{ method: {dblclick: 'edit', slot: 'select'} },
				 	{ class: 'text-center', method: {dblclick: 'edit', slot: 'input'} },
					{ method: {dblclick: 'edit', slot: 'input'} }
				],

				data: [
					{
						id: 1,
						title: 'Lorem ipsum dolor sit amet',
						category: 'Internet',
						create: '2017-02-03',
						author: 'administrator'
					},
					{
						id: 2,
						title: 'Esse explicabo, beatae accusantium odit ipsa velit atque',
						category: 'Cooking',
						create: '2017-09-15',
						author: 'moderator'
					},
					{
						id: 3,
						title: 'Lorem ipsum dolor sit amet',
						category: 'Default',
						create: '2016-11-21',
						author: 'guest'
					},
					{
						id: 4,
						title: 'Esse explicabo, beatae accusantium odit ipsa velit atque',
						category: 'Internet',
						create: '2016-11-29',
						author: 'alibaba'
					}
				],

				action: {
					r: false,// row index
					c: false,// col index
					edit_value: '',// v-model for all edit model
					checkbox: [],
					edit_row: {// value of edit_row in my-table component
						title: '',
						category: '',
						create: '',
						author: ''
					},
					pagination: {
						current_page: 1
					}
				}
			}
		},

		mounted() {
			this.$store.commit('set_page_header', {
	      title: 'Post',
	      desc: 'General'
	    })
		},

		methods: {

			edit(row, r, c, k) {
				this.action.r = r;
				this.action.c = c;
				this.action.edit_value = row[k];
			},

			submit(res) {
				if (res.row[res.k] != this.action.edit_value) {
					res.row[res.k] = this.action.edit_value;
				}
				this.reset_edit()
			},

			reset_edit() {
				this.action.r = false;
				this.action.c = false;
			},

			set_checkbox(item) { this.action.checkbox = item },

			submit_row(row) {
				for (let item in this.action.edit_row) {
					row[item] = this.action.edit_row[item]
				}
				this.$refs.my_table.cancel_edit_row(true)
			},

			current_page(num) {
				this.action.pagination.current_page = num
			}

		},

		components: {
			'my-table': require('../../general/table.vue'),
			'my-pagi': require('../../general/pagination.vue')
		}
	}
</script>
