# vue-simple-table
simple table inline edit

Contain component
https://github.com/vmaimone/vue-context-menu

> full example in general.vue

Demo https://cukikt0302.github.io/vue-simple-table/index.html

FULL example

```html
<template>
	<div class="row">
		<div class="col-md-12">
			<div class="box box-primary">
				<div class="box-body no-padding table-responsive">
					<my-table ref="my_table"
						:header="header" :body="body" :data="data" :action="action"
						:config="{table_class: 'table table-striped', checkbox: true, edit_row: true}" 
						@edit="edit" checkbox_method="set_checkbox" @set_checkbox="set_checkbox">
						
						<template slot="edit_row" scope="res">
							<form @submit.prevent="" role="form" @keyup.esc="$refs.my_table.cancel_edit_row">

								<div class="col-md-6">
									<div class="form-group">
										<label for="title">Title</label>
										<input type="text" class="form-control input-sm" v-focus 
										id="title" placeholder="Title" :value="res.row.title">
									</div>
								
									<div class="form-group">
										<label for="def">Def</label>
										<input type="text" class="form-control input-sm" 
										id="def" placeholder="Input field">
									</div>
								</div>
								<div class="col-md-6">
									<div class="form-group">
										<label for="gha">Gha</label>
										<input type="text" class="form-control input-sm" 
										id="gha" placeholder="Title">
									</div>
								
									<div class="form-group">
										<label for="category">Category</label>
										<select name="category" id="category" class="form-control input-sm">
											<option value="Default">Default</option>
											<option value="Internet">Internet</option>
											<option value="Cooking">Cooking</option>
										</select>
									</div>
								</div>
							
								<div class="col-md-12">
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

					<pre>{{ $data.action }}</pre>

				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		name: 'post-general',
		data() {
			return {
				header: [
					{ title: 'ID', class: 'text-center' },
					{ title: 'Title' },
					{ title: 'Category' },
					{ title: 'Create', class: 'text-center' },
					{ title: 'Author' }
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
						create: '03/02/2017',
						author: 'administrator'
					},
					{
						id: 2,
						title: 'Esse explicabo, beatae accusantium odit ipsa velit atque',
						category: 'Cooking',
						create: '15/09/2017',
						author: 'moderator'
					},
					{
						id: 3,
						title: 'Lorem ipsum dolor sit amet',
						category: 'Default',
						create: '01/04/2014',
						author: 'guest'
					},
					{
						id: 4,
						title: 'Esse explicabo, beatae accusantium odit ipsa velit atque',
						category: 'Internet',
						create: '28/11/1986',
						author: 'alibaba'
					}
				],
				action: {
					r: false,// row index
					c: false,// col index
					edit_value: '',// v-model for all edit model
					checkbox: [],
					edit_row: {// value from edit_row my-table component
						title: '',
						category: '',
						create: '',
						author: ''
					}
				}
			}
		},
		methods: {
			edit(row, r, c, k) {
				this.action.r = r;
				this.action.c = c;
				this.action.edit_value = row[k];
			},
			submit(res) {
				if (res.row[res.k] != this.action.edit_value)
				{
					res.row[res.k] = this.action.edit_value;
				}
				this.action.r = false;
				this.action.c = false;
			},
			reset_edit(res) {
				this.action.r = false;
				this.action.c = false;
			},
			check_ev(item, str) {
				return Object.keys(item)[0] == str 
					? item[Object.keys(item)]
					: false;
			},
			set_checkbox(item) {
				this.action.checkbox = item
			}
		},
		components: {
			'my-table': require('table.vue')
		}
	}
</script>

```
