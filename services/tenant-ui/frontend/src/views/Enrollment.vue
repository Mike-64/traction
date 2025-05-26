<template>
  <div>
    <Enrollment v-if="!selectedRecord" @select="handleSelect" />
    <EnrollmentForm v-else :enrollment="selectedRecord" @back="handleBack" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Enrollment from '@/components/enrollment/Enrollment.vue';
import EnrollmentForm from '@/components/enrollment/EnrollmentForm.vue';

const selectedRecord = ref(null);

function handleSelect(data) {
  console.log('Selected record:', data);
  selectedRecord.value = data;
}

function handleBack() {
  selectedRecord.value = null;
}
</script>
