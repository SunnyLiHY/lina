<template>
  <TicketListTable :url="url" />
</template>

<script>
import TicketListTable from './TicketListTable'
export default {
  name: 'MyTicketList',
  components: {
    TicketListTable
  },
  data() {
    return {
      url: '/api/v1/tickets/tickets/?assign=0'
    }
  }
}
</script>

<style scoped>

</style>
