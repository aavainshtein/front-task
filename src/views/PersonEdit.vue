<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { store } from '@/store'
import AVPerson from '@/components/AVPerson.vue'

const route = useRoute()

const person = computed(() => {
  const id = Number(route.params.id)
  return store.people.find((p) => p.id === id)
})

function updateAge(value: string) {
  if (person.value) {
    person.value.ageInHours = Number(value) || 0
  }
}
</script>

<template>
  <div v-if="person" class="flex flex-col gap-4">
    <router-link to="/" class="text-violet-600 hover:underline text-sm">&larr; Back</router-link>
    <AVPerson
      :avatar-src="'/cat.jpg'"
      :value="person.ageInHours"
      :label="`${person.name.toUpperCase()} IS`"
      caption="hours old"
      @change="updateAge($event ? Number($event).toString() : '0')"
      :debounce="200"
    />
  </div>

  <div v-else>
    <p class="text-gray-600">Person not found</p>
    <router-link to="/" class="text-violet-600 hover:underline text-sm">Back to list</router-link>
  </div>
</template>
