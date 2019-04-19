<template>
  <div class="vuetify-table">
    <h1>Vuetify Version of the Data Table</h1>
    <VuetifyTable></VuetifyTable>
  </div>
</template>

<script>
  // @ is an alias to /src
  import VuetifyTable from '../components/VuetifyTable'

  export default {
    name: 'vuetify',
    components: {
      VuetifyTable
    }
  }
</script>