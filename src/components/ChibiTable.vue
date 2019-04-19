<template>
    <v-card v-sheet>
        <v-card-title>
            <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ on }">
                    <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
                </template>
                <v-card>
                    <v-card-title>
                        <span class="headline">New Item</span>
                    </v-card-title>

                    <v-card-text>
                        <v-container grid-list-md>
                            <v-layout wrap justify-space-around>
                                <v-flex xs12 sm6 md6 v-for="item in headers" :key="item.title">
                                    <div v-if="item.type ==='text' || item.type === 'number' ">
                                        <v-text-field v-model="editedItem[item.value]"
                                                      :label="item.value"></v-text-field>
                                    </div>
                                    <div v-else-if="item.type ==='date' ">
                                        <v-menu
                                                v-model="date_menu"
                                                :close-on-content-click="false"
                                                :nudge-right="40"
                                                lazy
                                                transition="scale-transition"
                                                offset-y
                                                full-width
                                                min-width="290px"
                                        >
                                            <template v-slot:activator="{ on }">
                                                <v-text-field
                                                        v-model="editedItem[item.value]"
                                                        :label="item.value"
                                                        prepend-icon="event"
                                                        readonly
                                                        v-on="on"
                                                ></v-text-field>
                                            </template>
                                            <v-date-picker v-model="editedItem[item.value]"
                                                           @input="date_menu = false"></v-date-picker>
                                        </v-menu>
                                    </div>

                                    <div v-else-if="item.type ==='boolean' ">
                                        <v-checkbox
                                                v-model="editedItem[item.value]"
                                                primary
                                                :label="item.value"
                                        ></v-checkbox>
                                    </div>

                                </v-flex>

                            </v-layout>
                        </v-container>
                    </v-card-text>

                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
                        <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
            <v-spacer></v-spacer>
            {{sorted_data.length}} Items
            <v-spacer></v-spacer>
            <v-text-field
                    v-model="search"
                    append-icon="search"
                    label="Search"
                    single-line
                    hide-details
            ></v-text-field>
        </v-card-title>

        <div class="v-table__overflow elevation-1">
            <table class="v-datatable v-table v-datatable--select-all theme--light">
                <thead>
                <tr>
                    <th>
                        <v-checkbox
                                v-model="selectAll"
                                primary
                                hide-details
                                :indeterminate="partial_select"
                        ></v-checkbox>
                    </th>
                    <th class="text-xs-left column"
                        v-for="item in headers"
                        :key="item.title"
                        @click="item.sortable && set_sort(item.value)"
                        v-bind:class="{ active: sorting.attribute === item.value, desc: sorting.direction === -1, sortable: item.sortable  }"
                    >
                        {{item.title}}
                        <i v-if="item.sortable" aria-hidden="true" class="v-icon material-icons theme--light"
                           style="font-size: 16px;">arrow_upward</i>
                    </th>
                </tr>
                </thead>
                <transition-group name="table" tag="tbody" mode="in-out">
                    <tr v-for="row in sorted_data" :key="row.ID">
                        <td>
                            <v-checkbox
                                    v-model="selected"
                                    primary
                                    hide-details
                                    :value="row.ID"
                            ></v-checkbox>
                        </td>
                        <td v-for="(item, i) in row" :key="item">
                        <span v-if="header_type(i) === 'date'">
                            {{ new Date(item).toLocaleDateString() }}
                        </span>
                            <span v-else>{{item}}</span>
                        </td>
                    </tr>
                    <tr v-if="!sorted_data.length">
                        <td colspan="6" class="text-xs-center">No matching records found</td>
                    </tr>
                </transition-group>
            </table>
        </div>

    </v-card>
</template>

<script>
  export default {
    name: "ChibiTable",

    props: {
      headers: {
        type: Array,
        default: () => []
      },
      table_data: {
        type: Array,
        default: () => []
      },

    },

    data() {
      return {
        selected: [],
        search: "",
        dialog: false,
        date_menu: false,
        sorting: {attribute: "", direction: 1},
        editedIndex: -1,
        editedItem: {
          ID: '',
          Name: '',
          Description: 0,
          Date: 0,
          Amount: 0,
        },
        defaultItem: {
          ID: '',
          Name: '',
          Description: 0,
          Date: 0,
          Amount: 0,
        }
      }
    },

    computed: {
      selectAll: {
        get: function () {
          return this.sorted_data ? this.selected.length === this.sorted_data.length : false;
        },
        set: function (value) {
          let selected = [];

          if (value) {
            this.sorted_data.forEach(function (item) {
              selected.push(item.ID);
            });
          }

          this.selected = selected;
        }
      },

      partial_select: function () {
        return this.selected.length > 0 && this.selected.length < this.sorted_data.length;
      },

      filtered_data: function () {
        if (this.search.length === 0) {
          return this.table_data;
        }
        return this.table_data.filter((item) => Object.keys(item).some((key) => {
              if (item.hasOwnProperty(key)) {
                return item[key].toString().toLowerCase().indexOf(this.search.toLowerCase()) >= 0;
              } else return false;
            })
        );
      },

      sorted_data: function () {
        if (!this.sorting.attribute) {
          return this.filtered_data;
        }
        let attr = this.sorting.attribute;
        return [...this.filtered_data].sort((a, b) => {
          if (a[attr] > b[attr]) {
            return this.sorting.direction;
          }
          if (a[attr] < b[attr]) {
            return -1 * this.sorting.direction;
          }
          // a must be equal to b
          return 0;
        })

      },

    },

    methods: {

      close() {
        this.dialog = false;
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem);
          this.editedIndex = -1
        }, 300)
      }
      ,

      save() {
        if (this.editedIndex > -1) {
          Object.assign(this.table_data[this.editedIndex], this.editedItem);
        } else {
          this.table_data.push(this.editedItem)
        }
        this.close()
      },

      set_sort(attr) {
        if (this.sorting.attribute === attr && this.sorting.direction === 1)
          this.sorting.direction = -1;
        else {
          this.sorting.direction = 1;
          this.sorting.attribute = attr;
        }
      },

      header_type(header){
        let h = this.headers.find( (item) => item.title === header);
        return h && h.type;
      }
    }
  }
</script>

<style lang="sass" scoped>

    th, td
        max-width: 200px
        overflow: hidden

    thead th.column.sortable

        &.active
            color: rgba(0, 0, 0, 0.87)

    .table-enter-active, .table-leave-active
        transition: all 1s

    .table-enter, .table-leave-to
        opacity: 0
        transform: translateX(30px)

    .table-move
        transition: transform 1s


</style>