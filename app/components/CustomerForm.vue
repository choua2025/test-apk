<template>
  <v-dialog :model-value="modelValue" max-width="500" @update:model-value="emit('update:modelValue', $event)">
    <v-card>
      <v-card-title class="pa-5">{{ isEdit ? 'Update Customer' : 'Create Customer' }}</v-card-title>

      <v-form ref="formRef" @submit.prevent="submit">
        <v-card-text class="d-flex flex-column ga-2">
          <v-text-field v-model="form.name" label="Name" :rules="[required]" />
          <v-text-field v-model="form.email" label="Email" type="email" :rules="[required, emailRule]" />
          <v-text-field v-model="form.city" label="City" :rules="[required]" />
          <v-alert v-if="error" type="error" density="compact">{{ error }}</v-alert>
        </v-card-text>

        <v-card-actions class="pa-5">
          <v-spacer />
          <v-btn @click="close">Cancel</v-btn>
          <v-btn type="submit" color="primary" variant="flat" :loading="loading">
            {{ isEdit ? 'Update' : 'Create' }}
          </v-btn>
        </v-card-actions>
      </v-form>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import axios from 'axios'

const props = defineProps<{
  modelValue: boolean
  customer?: Record<string, any> | null
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void
  (e: 'saved'): void
}>()

const API_URL = 'http://localhost:5000/api/customers'

const emptyForm = () => ({
  name: '',
  email: '',
  city: '',
})

const formRef = ref()
const form = ref(emptyForm())
const loading = ref(false)
const error = ref('')

const isEdit = computed(() => !!props.customer?.id)

const required = (v: any) => (v !== null && v !== '' && v !== undefined) || 'Required'
const emailRule = (v: string) => /.+@.+\..+/.test(v) || 'Invalid email'

// Reset/fill the form each time the dialog opens
watch(
  () => props.modelValue,
  (open) => {
    if (!open) return
    error.value = ''
    form.value = props.customer
      ? { ...emptyForm(), name: props.customer.name, email: props.customer.email, city: props.customer.city }
      : emptyForm()
    formRef.value?.resetValidation()
  },
)

const close = () => emit('update:modelValue', false)

const submit = async () => {
  const { valid } = await formRef.value.validate()
  if (!valid) return

  loading.value = true
  error.value = ''
  const payload = {
    name: form.value.name,
    email: form.value.email,
    city: form.value.city,
  }
  try {
    if (isEdit.value) {
      await axios.put(`${API_URL}/${props.customer!.id}`, payload)
    } else {
      await axios.post(API_URL, payload)
    }
    emit('saved')
    close()
  } catch (e: any) {
    error.value = e?.response?.data?.message || 'Something went wrong. Please try again.'
  } finally {
    loading.value = false
  }
}
</script>
