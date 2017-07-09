<template>
<div>
	<table :class="config.table_class">
		
		<thead :class="config.head_class">
			<tr>
				<th v-show="config.show_increment">Icrement</th>
				<th class="text-capitalize" v-for="(thead, index) in header" 
				:class="thead.class" @click="sort_colum(thead)" 
				v-show="hidden_column.indexOf(index) === -1">

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

				<!-- <th></th> -->
			</tr>
		</thead>

		<tbody :class="config.body_class">
			<tr v-for="(row, r) in data" :class="{'tr-active': context_menu_row && context_menu_row.id === row.id}"
			@contextmenu="open_contextmenu($event, row)"
			v-if="row.id !== toggle_edit_row">

				<td v-show="config.show_increment">{{ r+1 }}</td>

				<td v-for="(item, k, c) in row" :key="item.id" 
					:class="body[c] && body[c].class" v-show="hidden_column.indexOf(c) === -1">

					<div v-if="body[c] && body[c].hasOwnProperty('method') && has_ev(body[c].method) && !toggle_edit_row">
						
						<!-- Click -->
						<div v-if="check_ev(body[c].method, 'click')" 
						@click="click(check_ev(body[c].method, 'click'), row, r, c, k)">
							{{ item }}
						</div>
						<!-- Click -->
						
						<!-- Dblclick -->
						<div v-else-if="check_ev(body[c].method, 'dblclick')"
						@dblclick="dblclick(check_ev(body[c].method, 'dblclick'), row, r, c, k)">
							<span v-if="v === r && h === c">

								<slot v-if="check_slot(body[c].method)" 
								:name="check_slot(body[c].method)"
								:row="row" :r="r" :c="c" :k="k"></slot>

								<span v-else>Can't find <b>Slot</b></span>

							</span>

							<span v-else>{{ item }}</span>
						</div>
						<!-- Dblclick -->
						
						<!-- Hover -->
						<div v-else-if="check_ev(body[c].method, 'hover')"
						@mouseover="hover(check_ev(body[c].method, 'hover'), row, r, c, k)">
							{{ item }}
						</div>
						<!-- Hover -->

						<span class="simple-table-undo">
							<tooltip text="Undo" v-show="row_undo && row_undo === row.id && uv === c">
								<button class="btn btn-xs btn-info" @click.prevent="set_data_row_undo(r)">
									<span class="fa fa-undo"></span>
								</button>
							</tooltip>
						</span>

					</div>

					<div v-else>
						<span v-if="config.checkbox && k == 'id' && c === 0 && !toggle_edit_row">
							<input type="checkbox" :value="item" v-model="checkbox">
						</span>
						<span v-else>{{ item }}</span>
					</div>

				</td>

				<td v-show="row_undo && row_undo === row.id && uv === false">
					<tooltip text="Undo">
						<button class="btn btn-xs btn-info" @click.prevent="set_data_row_undo(r)">
							<span class="fa fa-undo"></span>
						</button>
					</tooltip>
				</td>

			</tr>

			<tr v-else class="tr-active">
				<td :colspan="header.length + 1">
					<slot name="edit_row" :row="find_by_id(toggle_edit_row)"></slot>
				</td>
			</tr>
		</tbody>

	</table>

	<context-menu v-if="config.edit_row" id="context-menu" ref="ctxMenu"
	@ctx-open="onCtxOpen" @ctx-cancel="resetCtxLocals" @ctx-close="onCtxClose">
	  <li @click.prevent="set_edit_row">
	  	<i class="fa fa-wrench"></i> <span>{{ contextmenu_edit_text }}</span>
	  </li>
	  <li @click.prevent="remove_row">
	  	<i class="fa fa-remove"></i> <span>{{ contextmenu_remove_text }}</span>
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
import { Tooltip } from 'uiv'

export default {
	name: 'vue-simple-table',

	props: {
		config: { default() { return false } },
		header: { default() { return {} } },
		body: { default() { return {} } },
		h: { default() { return false } },
		v: { default() { return false } },
		data: { default() { return {} } },
		contextmenu_edit_text: { default() { return 'Edit' } },
		contextmenu_remove_text: { default() { return 'Remove' } },
		hidden_column: { default() { return [] } }
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
			uv: false// undo v
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
			if (this.config && this.config.edit_row) {
				e.preventDefault();
				this.$refs.ctxMenu.open(e, row)
			}
			
		},

		sort_colum(item) {
			if ( !this.config.sort || !item.hasOwnProperty('key') || this.toggle_edit_row) return
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

		click(Emit, row, r, c, k) { this.$emit(Emit, row, r, c, k) },
		dblclick(Emit, row, r, c, k) { this.$emit(Emit, row, r, c, k) },
		hover(Emit, row, r, c, k) { this.$emit(Emit, row, r, c, k) },
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
  		this.data_row_undo = false
  		this.row_undo = false
  	},

    remove_row() {
    	let remove_id = []
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
    		if (confirm(confirmStr)) { this.data.splice(index, 1) }
    	}
    	this.context_menu_row = false
    	this.checkbox = []
    	this.$emit('remove_row', remove_id)
    },

    find_by_id(id) { return this.data.find(obj => obj.id === id) }
	},

	components: {
		'context-menu': require('vue-context-menu'),
		Tooltip
	}
}
</script>
