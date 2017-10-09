<template>
<div :class="wrap_class">
	<table :class="config.table_class">
		
		<thead :class="config.head_class">
			<tr>
				<th v-show="config.show_increment">Icrement</th>
				<th class="text-capitalize" v-for="(thead, index) in header" 
				:class="thead.class" @click="sort_colum(thead)" 
				v-show="hide_col.indexOf(index) === -1">

					<span v-if="config.checkbox && index === 0 && !toggle_edit_row">
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

				<th v-show="config.hasOwnProperty('undo_row') && config.undo_row"></th>
			</tr>
		</thead>

		<tbody :class="config.body_class">
			<tr v-for="(row, r) in data" @contextmenu="open_contextmenu($event, row)"
			:class="{'tr-active': context_menu_row && context_menu_row.id === row.id}"
			v-if="row.id !== toggle_edit_row">

				<td v-show="config.show_increment">{{ r+1 }}</td>

				<td v-for="(item, k, c) in row" :key="`${k}--${c}`" 
					:class="body[c] && body[c].class" v-show="hide_col.indexOf(c) === -1">

					<div v-if="body[c] && body[c].hasOwnProperty('method') && has_ev(body[c].method) && !toggle_edit_row">
						
						<!-- Click -->
						<div v-if="check_ev(body[c].method, 'click')" 
						@click="click(check_ev(body[c].method, 'click'), row, r, c, k)">
							<span v-if="v === r && h === c">
								<slot v-if="check_slot(body[c].method)" 
								:name="check_slot(body[c].method)" v-bind="{row, r, c, k}"></slot>

								<!-- <span v-else style="color: red">Can't find slot property: "body.method.slot"</span> -->
								<span v-else>{{ item || default_cell }}</span>
							</span>

							<span v-else>{{ item || default_cell }}</span>
						</div>
						<!-- Click -->
						
						<!-- Dblclick -->
						<div v-else-if="check_ev(body[c].method, 'dblclick')"
						@dblclick="dblclick(check_ev(body[c].method, 'dblclick'), row, r, c, k)">
							<span v-if="v === r && h === c">
								<slot v-if="check_slot(body[c].method)" 
								:name="check_slot(body[c].method)" v-bind="{row, r, c, k}"></slot>

								<span v-else style="color: red">Can't find SLOT in PROP: "body.method.slot"</span>
							</span>

							<span v-else>{{ item || default_cell }}</span>
						</div>
						<!-- Dblclick -->
						
						<!-- Hover -->
						<div v-else-if="check_ev(body[c].method, 'hover')"
						@mouseover="hover(check_ev(body[c].method, 'hover'), row, r, c, k)">
							<span v-if="v === r && h === c">
								<slot v-if="check_slot(body[c].method)" 
								:name="check_slot(body[c].method)" v-bind="{row, r, c, k}"></slot>

								<span v-else>Can't find SLOT in PROP: "body.method.slot"</span>
							</span>

							<span v-else style="color: red">{{ item }}</span>
						</div>
						<!-- Hover -->
						
						<!-- Undo Cell -->
						<span class="simple-table-undo">
							<button v-show="show_undo_cell(row.id, r, c)"
							class="btn btn-xs btn-info" @click.prevent="set_data_row_undo(r)">
								<span class="fa fa-undo"></span>
							</button>
						</span>
						<!-- Undo Cell -->

					</div>

					<div v-else>
						<span v-if="config.checkbox && k == 'id' && c === 0 && !toggle_edit_row">
							<input type="checkbox" :value="item" v-model="checkbox">
						</span>
						<span v-else>{{ item }}</span>
					</div>

				</td>
				
				<!-- Undo row -->
				<td v-show="config.hasOwnProperty('undo_row') && config.undo_row">
					<span v-show="show_undo_row(row.id)">
						<button class="btn btn-xs btn-info" @click.prevent="set_data_row_undo(r)">
							<span class="fa fa-undo"></span>
						</button>
					</span>
				</td>
				<!-- Undo Row -->

			</tr>

			<tr v-else class="tr-active">
				<td :colspan="header.length + 1">
					<slot name="edit_row" :row="find_by_id(toggle_edit_row)"></slot>
				</td>
			</tr>
		</tbody>

	</table>

	<context-menu v-if="config.context_menu" id="context-menu" ref="ctxMenu"
	@ctx-open="onCtxOpen" @ctx-cancel="resetCtxLocals" @ctx-close="onCtxClose">
	  <li v-if="config.edit_row" @click.prevent="set_edit_row">
	  	<span class="pull-right">
	  		<i class="fa fa-wrench"></i>
	  	</span>
  	 	<span class="context-menu-right">{{ contextmenu_edit_text }}</span>
	  </li>
	  <li v-if="config.delete_row" @click.prevent="remove_row">
	  	<span class="pull-right">
	  		<i class="fa fa-remove"></i>
	  	</span>
	  	<span class="context-menu-right">{{ contextmenu_remove_text }}</span>
	  </li>
	  <slot name="list_context" :row="context_menu_row"></slot>
	</context-menu>

	<!-- <context-menu v-else style="display: none" id="context-menu" ref="ctxMenu"></context-menu> -->
</div>
</template>

<style>
	.simple-table-undo {
    float: right;
    position: relative;
    margin-top: -20px;
    display: block;
	}
	.tr-active { background-color: rgba(148, 149, 158, 0.38) !important; }
	.ctx-menu { font-size: 1em !important; min-width: 200px !important; }
	#context-menu li {
		padding: 5px 10px;
		font-size: .9em;
		font-weight: 600;
    line-height: 1.5;
    cursor: context-menu;
	}
	#context-menu li:not(:last-child) { border-bottom: 1px dotted #e3e3e3; }
	#context-menu li > i.fa { padding: 0 5px; }
	#context-menu li:hover { 
		color: #2b2d2f;
    text-decoration: none;
    background-color: #f5f5f5;
	}
</style>

<script>
/*
 * Author: Nguyen Ke Nhat
 * Create at: 04/14/2017
 * Github: https://github.com/cukikt0302/vue-simple-table
*/
export default {
	name: 'vue-simple-table',

	props: {
		config: { default() { return false } },
		header: { default() { return {} } },// header key (sort property), class
		body: { default() { return {} } },// body event, class, slot
		data: { default() { return {} } },// body data
		contextmenu_edit_text: { type: String, default() { return 'Edit' } },
		contextmenu_remove_text: { type: String, default() { return 'Remove' } },
		hide_col: { default() { return [] } },// vertical index want hide column
		wrap_class: { type: String, default() { return '' } },
		default_cell: { type: String, default() { return '' } }
	},

	data() {
		return {
			checkbox: [],
			context_menu_row: false,
			toggle_edit_row: false,
			toggle_sort: -1,
			column_sort: '',
			row_undo: false,
			data_row_undo: false,
			uv: false,// undo vertical index
			h: false,// horizontal index
			v: false// vertical index
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
			this.$emit('get_checkbox', newVal)
		}
	},

	methods: {
		open_contextmenu(e, row) {
			if (this.config && this.config.context_menu) {
				e.preventDefault();
				this.$refs.ctxMenu.open(e, row)
			}
		},

		sort_colum(item) {
			if ( !this.config.sort || !item.hasOwnProperty('key') 
				|| this.toggle_edit_row || this.v || this.h) return
			if (item.key == 'id' && this.config.checkbox) return
			this.toggle_sort = -(1) * this.toggle_sort
			this.column_sort = item.key
			this.data.sort((a, b) => {
				if (typeof a[item.key] == 'string' && typeof b[item.key] == 'string') {
			    var nameA = a[item.key].toLowerCase(), nameB = b[item.key].toLowerCase()
			  } else {
			  	var nameA = a[item.key], nameB = b[item.key]
			  }
        return (nameA == nameB ? 0 : nameA < nameB ? -1 : 1) * (this.toggle_sort)
			})
		},

		// Event
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

		click(Emit, row, vertical_index, horizontal_index, k) {
			this.v = vertical_index;
			this.h = horizontal_index;
			this.$emit(Emit, row[k])
		},
		dblclick(Emit, row, vertical_index, horizontal_index, k) {
			this.v = vertical_index;
			this.h = horizontal_index;
			this.$emit(Emit, row[k])
		},
		hover(Emit, row, vertical_index, horizontal_index, k) {
			this.v = vertical_index;
			this.h = horizontal_index;
			this.$emit(Emit, row[k])
		},
		// Event

		onCtxOpen(row) { this.context_menu_row = row },
    onCtxClose(row) {},
    resetCtxLocals(row) { this.context_menu_row = false },

    set_edit_row() {// context_menu_row when right click context menu onCtxOpen()
    	this.toggle_edit_row = this.context_menu_row.id
    	this.$emit('edit_row_emit', this.context_menu_row)
    },

    cancel_edit_row(clearContext) { 
    	this.toggle_edit_row = false
    	if ( !clearContext) this.context_menu_row = false
    },

  	cancel_edit_cell() {
  		this.h = false
  		this.v = false
  	},

  	close(clearContext) {
  		this.toggle_edit_row = false
    	this.h = false
  		this.v = false
  		// set tr-active class, after edit row
  		if ( !clearContext) this.context_menu_row = false
  	},

  	show_undo_cell(id, r, c) {
  		if (this.config.hasOwnProperty('undo_cell') && this.config.undo_cell === true)
	  		return (this.row_undo && this.row_undo === id
	  		 	&& this.uv === c && this.v !== r && this.h !== c)
	  	else
	  		return false
  	},

  	show_undo_row(id) {
  		if (this.config.hasOwnProperty('undo_row') && this.config.undo_row === true)
  			return this.row_undo && this.row_undo === id && this.uv === false
  		else
  			return false
  	},

  	set_row_undo(row, v) {
  		this.data_row_undo = row
  		this.row_undo = row.id || false
  		if (v != undefined) {
  			this.uv = v
  		} else {
  			this.uv = false
  		}
  	},

  	set_data_row_undo(index) {
  		this.$set(this.data, index, this.data_row_undo)
  		this.$emit('undo_row_emit', JSON.parse(JSON.stringify(this.data_row_undo)))
  		this.data_row_undo = false
  		this.row_undo = false
  	},

    remove_row() {
    	let remove_id = [], is_confirm = false, obj_id = []
    	if (this.checkbox.length > 0) {
    		if (confirm(
    			`Remove ${this.checkbox.length} ${(
    				this.checkbox.length === 1 ? 'row' : 'rows'
  				)} has checked ?`
  			)) {
    			for (let i = this.checkbox.length -1; i >= 0; i--) {
    				let obj = this.data.filter(ob => ob.id === this.checkbox[i])[0]
    				remove_id.push(i)
    				this.data.splice(this.data.indexOf(obj), 1)
      		}
      		is_confirm = true
      		obj_id = JSON.parse(JSON.stringify(this.checkbox))
    		}
    	} else {
    		let index = this.data.indexOf(this.context_menu_row),
    				confirmStr = `Remove row: ${(
    					index + 1 === 1 ? '1 ST' 
    						: index + 1 === 2 ? '2 ND' 
    						: index + 1 === 3 ? '3 RD' 
    						: (index + 1) + ' TH'
  					)}`;
				remove_id.push(index)
    		if (confirm(confirmStr)) { 
    			this.data.splice(index, 1) 
    			is_confirm = true
    			obj_id.push(this.context_menu_row.id)
    		}
    	}
    	this.context_menu_row = false
    	this.checkbox = []
    	if (is_confirm) {
    		this.$emit('remove_row', remove_id, obj_id)
    	}
    },

    find_by_id(id) { return this.data.find(obj => obj.id === id) }
	},

	components: {
		'context-menu': require('vue-context-menu')
	}
}
</script>
