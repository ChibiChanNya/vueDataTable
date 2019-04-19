<template>
  <div class="vuetify-table">
    <h1>Chibi's Version of the Data Table</h1>
    <ChibiTable
            :headers="headers"
            :table_data="table_data"
    >

    </ChibiTable>
  </div>
</template>

<script>
  import data from '@/assets/data/sample-data'

  // @ is an alias to /src
  import ChibiTable from '../components/ChibiTable'

  export default {
    name: 'vuetify',
    components: {
      ChibiTable
    },

    data() {
      return {
        headers: [
          {title: 'ID', sortable: false, value: 'ID', type: "text"},
          {title: 'Name', value: 'Name', sortable: true, type: "text"},
          {title: 'Description', value: 'Description', sortable: true, editable: true, type: "text"},
          {title: 'Date', value: 'Date', sortable: true, type:"date"},
          {title: 'Amount', value: 'Amount', sortable: true, type: "number"},
        ],
        table_data: data,


      }
    },


  }
</script>