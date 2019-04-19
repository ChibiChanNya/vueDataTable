<template>
    <v-card>
        <v-card-title>
            <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ on }">
                    <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
                </template>
                <v-card>
                    <v-card-title>
                        <span class="headline">{{ formTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                        <v-container grid-list-md>
                            <v-layout wrap>
                                <v-flex xs12 sm6 md4>
                                    <v-text-field v-model="editedItem.name" label="Dessert name"></v-text-field>
                                </v-flex>
                                <v-flex xs12 sm6 md4>
                                    <v-text-field v-model="editedItem.calories" label="Calories"></v-text-field>
                                </v-flex>
                                <v-flex xs12 sm6 md4>
                                    <v-text-field v-model="editedItem.fat" label="Fat (g)"></v-text-field>
                                </v-flex>
                                <v-flex xs12 sm6 md4>
                                    <v-text-field v-model="editedItem.carbs" label="Carbs (g)"></v-text-field>
                                </v-flex>
                                <v-flex xs12 sm6 md4>
                                    <v-text-field v-model="editedItem.protein" label="Protein (g)"></v-text-field>
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
            <v-text-field
                    v-model="search"
                    append-icon="search"
                    label="Search"
                    single-line
                    hide-details
            ></v-text-field>
        </v-card-title>
        <v-data-table
                :headers="headers"
                :items="table_data"
                :search="search"
                item-key="ID"
                class="elevation-1"
                select-all
                v-model="selected"
        >
            <template v-slot:items="props">
                <td >
                    <v-checkbox
                            v-model="props.selected"
                            primary
                            hide-details
                    ></v-checkbox>
                </td>
                <td class="text-xs-right">{{ props.item.ID }}</td>
                <td class="text-xs-right">{{ props.item.Name }}</td>
                <td class="text-xs-right">
                    <v-edit-dialog
                            :return-value.sync="props.item.Description"
                            lazy
                            @save="save"
                            @cancel="close"

                    > {{ props.item.Description }}
                        <template v-slot:input>
                            <v-text-field
                                    v-model="props.item.Description"
                                    label="Edit"
                            ></v-text-field>
                        </template>
                    </v-edit-dialog>
                </td>
                <td class="text-xs-right">{{ new Date(props.item.Date).toLocaleDateString() }}</td>
                <td class="text-xs-right">$ {{ props.item.Amount }}</td>
            </template>
            <template v-slot:no-data>
                <v-btn color="primary" @click="initialize">Reset</v-btn>
            </template>
            <v-alert v-slot:no-results :value="true" color="error" icon="warning">
                Your search for "{{ search }}" found no results.
            </v-alert>
        </v-data-table>
    </v-card>
</template>

<script>
  import data from '../assets/data/sample-data'

  export default {
    name: "VuetifyTable",

    data() {
      return {
        headers: [
          {
            text: 'ID',
            align: 'left',
            sortable: false,
            value: 'ID'
          },
          {text: 'Name', value: 'Name'},
          {text: 'Description', value: 'Description'},
          {text: 'Date', value: 'Date'},
          {text: 'Amount', value: 'Amount'},
        ],
        table_data: null,
        search: null,
        selected: [],
        dialog: false,
        editedIndex: -1,
        editedItem: {
          name: '',
          description: 0,
          date: 0,
          amount: 0,
        },
        defaultItem: {
          name: '',
          description: 0,
          date: 0,
          amount: 0,
        }
      }
    },

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      }
    },

    watch: {
      dialog (val) {
        val || this.close()
      }
    },

    created () {
      this.initialize()
    },

    methods:{

      initialize(){
        this.table_data = data;
      },

      close () {
        this.dialog = false;
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem);
          this.editedIndex = -1
        }, 300)
      },

      save () {
        if (this.editedIndex > -1) {
          Object.assign(this.table_data[this.editedIndex], this.editedItem);
        } else {
          this.desserts.push(this.editedItem)
        }
        this.close()
      }
    }
  }
</script>

<style scoped>

</style>