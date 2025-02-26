<script setup>
import {ref} from 'vue'
import * as yup from 'yup'

const task = ref({
  title: '',
  description: '',
  deliveryDate: '',
  priority: '',
  category: '',
  duration: '',
  acceptTerms: false,
})

const initialTask = JSON.parse(JSON.stringify(task.value));

const errors = ref({
  title: '',
  description: '',
  deliveryDate: '',
  priority: '',
  category: '',
  duration: '',
  acceptTerms: '',
})

const taskCreated = ref(false)

const schema = yup.object().shape({
  title: yup.string().required('Title is required'),
  description: yup.string().required('Description is required'),
  deliveryDate: yup.date().required('Delivery date is required').typeError('Must be a valid date'),
  priority: yup.string().required('You must select a priority'),
  category: yup.string().required('You must choose a category'),
  duration: yup.number()
      .required('Duration is required')
      .min(1, 'Minimum 1 hour')
      .max(12, 'Maximum 12 hours')
      .typeError('Must be a number'),
  acceptTerms: yup.boolean()
      .oneOf([true], 'You must accept the terms and conditions'),
})

const validateTask = async () => {
  try {
    await schema.validate(task.value, {abortEarly: false})
    errors.value = {}
    return true
  } catch (err) {
    errors.value = err.inner.reduce((acc, error) => {
      acc[error.path] = error.message
      return acc
    }, {})
    return false
  }
}

const submitTask = async () => {
  if (await validateTask()) {
    taskCreated.value = true
  }
}

const resetForm = () => {
  task.value = JSON.parse(JSON.stringify(initialTask));
  errors.value = {};
  taskCreated.value = false;
};
</script>

<template>
  <div class="w-full max-w-lg bg-white p-6 rounded-lg shadow-md">
    <h2 class="text-xl font-semibold py-4 text-black py-2">Isning test form</h2>
    <form @submit.prevent="submitTask" class="w-full">
      <div class="py-4">
        <label class="block text-gray-700 text-sm font-bold mb-2" for="title">Title</label>
        <input v-model="task.title" id="title" type="text"
               class="w-full px-3 py-2 border border-blue-300 rounded-lg focus:outline-none focus:ring focus:border-blue-300 text-black"/>
        <p v-if="errors.title" class="text-red-500 text-sm">{{ errors.title }}</p>
      </div>
      <div class="py-4">
        <label class="block text-gray-700 text-sm font-bold mb-2" for="description">Description</label>
        <textarea v-model="task.description" id="description" rows="3"
                  class="w-full px-3 py-2 border border-blue-300 rounded-lg focus:outline-none focus:ring focus:border-blue-300 text-black"></textarea>
        <p v-if="errors.description" class="text-red-500 text-sm">{{ errors.description }}</p>
      </div>

      <div class="py-4">
        <label class="block text-gray-700 text-sm font-bold mb-2" for="deliveryDate">Delivery Date</label>
        <input v-model="task.deliveryDate" id="deliveryDate" type="date"
               class="w-full px-3 py-2 border border-blue-300 rounded-lg focus:outline-none focus:ring focus:border-blue-300 text-black"/>
        <p v-if="errors.deliveryDate" class="text-red-500 text-sm">{{ errors.deliveryDate }}</p>
      </div>

      <div class="py-4">
        <label class="block text-gray-700 text-sm font-bold mb-2">Priority</label>
        <div class="flex gap-4">
          <label class="text-black py-1"><input type="radio" v-model="task.priority" value="Low"/> Low</label>
          <label class="text-black py-1"><input type="radio" v-model="task.priority" value="Medium"/> Medium</label>
          <label class="text-black py-1"><input type="radio" v-model="task.priority" value="High"/> High</label>
        </div>
        <p v-if="errors.priority" class="text-red-500 text-sm">{{ errors.priority }}</p>
      </div>
      <div class="py-4">
        <label class="block text-gray-700 text-sm font-bold mb-2" for="category">Category</label>
        <select v-model="task.category" id="category"
                class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring focus:border-blue-300 text-black border-blue-300">
          <option value="" class="text-black">Select a category</option>
          <option value="Work" class="text-black">Work</option>
          <option value="Personal" class="text-black">Personal</option>
          <option value="Study" class="text-black">Study</option>
        </select>
        <p v-if="errors.category" class="text-red-500 text-sm">{{ errors.category }}</p>
      </div>
      <div class="py-4">
        <label class="block text-gray-700 text-sm font-bold mb-2" for="duration">Duration (hours)</label>
        <input v-model="task.duration" id="duration" type="number" min="1" max="12"
               class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring focus:border-blue-300 text-black border-blue-300"/>
        <p v-if="errors.duration" class="text-red-500 text-sm">{{ errors.duration }}</p>
      </div>

      <div class="py-4">
        <label class="flex items-center gap-x-2 text-black">
          <input type="checkbox" v-model="task.acceptTerms" class="mr-2 text-black"/>
          I accept the terms and conditions
        </label>
        <p v-if="errors.acceptTerms" class="text-red-500 text-sm">{{ errors.acceptTerms }}</p>
      </div>

      <button type="submit"
              class="w-full bg-blue-500 text-white py-2 px-4 rounded-lg hover:bg-blue-600 transition mb-6">
        Create Task
      </button>
    </form>

    <div v-if="taskCreated" class="relative space-y-4 p-4 bg-green-100 border border-green-400 text-green-700 rounded">
      <p><strong>✅ Task created successfully!</strong></p>
      <p><strong>Title:</strong> {{ task.title }}</p>
      <p><strong>Description:</strong> {{ task.description }}</p>
      <p><strong>Delivery Date:</strong> {{ task.deliveryDate }}</p>
      <p><strong>Priority:</strong> {{ task.priority }}</p>
      <p><strong>Category:</strong> {{ task.category }}</p>
      <p><strong>Duration:</strong> {{ task.duration }} hours</p>
      <p><strong>Accepted Terms:</strong> {{ task.acceptTerms ? 'Yes' : 'No' }}</p>

      <button @click="resetForm"
              class="w-full bg-red-500 text-white py-2 px-4 rounded-lg hover:bg-blue-600 transition">
        Reset Form
      </button>
    </div>
  </div>
</template>