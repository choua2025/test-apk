<template>
  <div class="pa-10">
   <div class="d-flex align-center justify-space-between mb-4">
    <h1 class="text-red">table</h1>
    <v-btn color="primary" @click="openCreate">Create Customer</v-btn>
   </div>

   <CustomerForm v-model="dialog" :customer="selected" @saved="getuser" />

   <v-card class="pa-10 shadow-xl" >
    <v-data-table :headers="header" :items="items" >

  
     <template v-slot:item.actions="{ item }">
      <div class="d-flex align-center">
        <v-btn color="primary" size="small" @click="openEdit(item)">Edit</v-btn>
        <v-btn color="primary" size="small" @click="getuserIdDelete(item)">Delete</v-btn>
      </div>
    </template>
 

    </v-data-table>
   </v-card>
   
  </div>
</template>

<script setup lang="ts">
import axios from 'axios'
import { ref } from 'vue'
const data = {}
const header = ref([
  { title: 'Name', key: 'name' },
  { title: 'Email', key: 'email' },
  { title: 'City', key: 'city' },
  { title: 'Status', key: 'status' },
  { title: 'product', key: 'product_name' },
  { title: 'Price', key: 'price' },
  {title: 'Quantity', key: 'quantity' },
  {title: "Actions", key: "actions"}
])

onMounted(() => {
  getuser()
})

const items = ref([])
const getuser = async () => {
  const data = await axios.get('http://localhost:5000/api/customers')
  items.value = data?.data
}

const dialog = ref(false)
const selected = ref<any>(null)

const openCreate = () => {
  selected.value = null
  dialog.value = true
}

const openEdit = (item: any) => {
  selected.value = item
  dialog.value = true
}

const getuserIdDelete = async (item: any) => {
  const userId = item.id
  console.log(userId)
  await axios.delete(`http://localhost:5000/api/customers/${userId}`).then((response) => {
    getuser() 
  }).catch((error) => {
    console.error('Error deleting user:', error)
  })
}

</script>