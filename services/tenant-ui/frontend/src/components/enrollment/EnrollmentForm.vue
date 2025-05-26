<template>
  <div class="flex justify-content-between align-items-center mb-4">
    <div class="flex align-items-center gap-3">
      <Button
        icon="pi pi-angle-left"
        variant="text"
        rounded
        size="large"
        @click="$emit('back')"
      />
      <h2 class="text-2xl font-bold">{{ enrollment.student_full_name }}</h2>
    </div>
    <div class="flex align-items-center gap-3">
      <!-- Status Dropdown -->
      <Dropdown
        v-model="status"
        :options="enrollmentStatusOptions"
        option-label="label"
        option-value="value"
        class="w-32"
      />
      <!-- Download Button -->
      <Button
        label="Download Transcript"
        icon="pi pi-download"
        severity="secondary"
        class="p-button btn-primary"
        @click="downloadTranscript"
      />
    </div>
  </div>
  <div class="p-6 max-w-4xl mx-auto rounded-lg shadow-md">
    <!-- Header with Status and Download -->
    <!-- <h1 class="text-2xl font-bold mb-4">{{ $t('enrollment.information') }}</h1> -->

    <Panel class="mb-4">
      <template #header>
        <h2 class="text-xl font-semibold mb-2">
          {{ $t('enrollment.studentInformation') }}
        </h2>
      </template>
      <p>
        <strong>{{ $t('enrollment.name') }}</strong>
        {{ enrollment.student_full_name }}
      </p>
      <p>
        <strong>{{ $t('enrollment.dateOfBirth') }}</strong>
        {{ enrollment.student_birth_date }}
      </p>
      <p>
        <strong>{{ $t('enrollment.address') }}</strong>
        {{ enrollment.student_address }}
      </p>
      <p>
        <strong>{{ $t('enrollment.phone') }}</strong>
        {{ enrollment.student_phone }}
      </p>
      <p>
        <strong>{{ $t('enrollment.email') }}</strong>
        {{ enrollment.student_email }}
      </p>
    </Panel>

    <Panel class="mb-4">
      <template #header>
        <h2 class="text-xl font-semibold mb-2">
          {{ $t('enrollment.schoolInfo') }}
        </h2>
      </template>
      <p>
        <strong>{{ $t('enrollment.name') }}</strong>
        {{ enrollment.school_name }}
      </p>
      <p>
        <strong>{{ $t('enrollment.address') }}</strong>
        {{ enrollment.school_address }}
      </p>
      <p>
        <strong>{{ $t('enrollment.graduationYear') }}</strong>
        {{ enrollment.graduation_date }}
      </p>
      <p>
        <strong>{{ $t('enrollment.gpa') }}</strong> {{ enrollment.gpa }}
      </p>
    </Panel>

    <Panel
      v-for="(term, i) in enrollment.terms"
      :key="i"
      class="mb-4"
      toggleable
    >
      <template #header>
        <h2 class="text-xl font-semibold mb-2">
          {{ $t('enrollment.terms') + ' ' + i++ }}
        </h2>
      </template>
      <Splitter class="mb-5" style="height: 400px">
        <SplitterPanel class="flex align-items-center justify-content-center">
          <div class="border p-3 mb-3 rounded">
            <p>
              <strong>{{ $t('enrollment.year') }}</strong> {{ term.termYear }}
            </p>
            <p>
              <strong>{{ $t('enrollment.name') }}</strong>
              {{ term.termSchoolName }}
            </p>
            <p>
              <strong>{{ $t('enrollment.gradeLevel') }}</strong>
              {{ term.termGradeLevel }}
            </p>
            <p>
              <strong>{{ $t('enrollment.credits') }}</strong>
              {{ term.termCredit }}
            </p>
            <p>
              <strong>{{ $t('enrollment.gpa') }}</strong> {{ term.termGpa }}
            </p>
            <h3 class="font-medium mt-2">{{ $t('enrollment.courses') }}</h3>
          </div>
        </SplitterPanel>
        <SplitterPanel
          class="flex align-items-center justify-content-center flex-wrap"
        >
          <h3 class="font-medium mt-2">{{ $t('enrollment.courses') }}</h3>
          <Card v-for="(course, j) in term.courses" :key="j" class="">
            <template #subtitle
              >{{ course.courseCode }} {{ course.courseTitle }}</template
            >
            <template #content>
              {{ $t('enrollment.grade') }} {{ course.grade }}
              {{ $t('common.comma') }} {{ $t('enrollment.credits') }}
              {{ course.creditEarned }}
            </template>
          </Card>
        </SplitterPanel>
      </Splitter>
    </Panel>
    <Panel ref="transcriptRef" class="mb-4">
      <template #header>
        <h2 class="text-xl font-semibold mb-2">
          {{ $t('enrollment.transcript') }}
        </h2>
      </template>
      <p>
        <strong>{{ $t('enrollment.program') }}</strong>
        {{ enrollment.transcript.program }}
      </p>
      <p>
        <strong>{{ $t('enrollment.classRank') }}</strong>
        {{ enrollment.transcript.classRank }}
      </p>
      <p>
        <strong>{{ $t('enrollment.gpa') }}</strong>
        {{ enrollment.transcript.gpa }}
      </p>
      <p>
        <strong>{{ $t('enrollment.unweightedGPA') }}</strong>
        {{ enrollment.transcript.gpaUnweighted }}
      </p>
      <p>
        <strong>{{ $t('enrollment.transcriptDate') }}</strong>
        {{ enrollment.transcript.transcriptDate }}
      </p>
      <p>
        <strong>{{ $t('enrollment.transcriptComments') }}</strong>
        {{ enrollment.transcript.transcriptComments }}
      </p>
    </Panel>
<div class="flex align-items-center gap-3">
    <Button
    class="mt-4 px-4 py-2 rounded p-button btn-primary"
    label="Save"
    @click="saveEnrollment"
    />
    <Button
    class="mt-4 px-4 py-2 rounded p-button btn-primary"
    @click="downloadPDF"
    >
    {{ $t('enrollment.download') }}
</Button>
</div>
</div>
</template>

<script setup>
import { ref, defineProps } from 'vue';
import jsPDF from 'jspdf';
import Button from 'primevue/button';
import Dropdown from 'primevue/dropdown';
import Panel from 'primevue/panel';
import Card from 'primevue/card';
import Splitter from 'primevue/splitter';
import SplitterPanel from 'primevue/splitterpanel';
import { useToast } from 'vue-toastification';
defineEmits(['back']);
const toast = useToast();
const props = defineProps({
  enrollment: {
    type: Object,
    required: true,
  },
});
console.log('enrollment', props.enrollment);
const enrollmentStatusOptions = [
  {
    label: 'Started',
    value: 'started',
  },
  {
    label: 'Completed',
    value: 'completed',
  },
  {
    label: 'Approved',
    value: 'approved',
  },
  {
    label: 'Rejected',
    value: 'rejected',
  },
  {
    label: 'Pending',
    value: 'pending',
  },
];
const status = ref(props.enrollment.enrollment_status || 'pending');
const saveEnrollment = () => {
  // This function would typically send the updated status to the server
  toast.success('Enrollment status updated successfully!');
  console.log('Saving enrollment with status:', status.value);
};
const transcriptRef = ref(null);

function downloadPDF() {
  const doc = new jsPDF();
  let y = 10;

  doc.setFontSize(14);
  doc.text('Transcript', 10, y);
  y += 10;

  const fields = [
    ['Name', props.enrollment.student_full_name],
    ['Program', props.enrollment.transcript.program],
    ['Class Rank', props.enrollment.transcript.classRank],
    ['Weighted GPA', props.enrollment.transcript.gpa],
    ['Unweighted GPA', props.enrollment.transcript.gpaUnweighted],
    ['Transcript Date', props.enrollment.transcript.transcriptDate],
    ['Comments', props.enrollment.transcript.transcriptComments],
  ];

  doc.setFontSize(11);
  fields.forEach(([label, value]) => {
    doc.text(`${label}: ${value || '-'}`, 10, y);
    y += 8;
  });

  props.enrollment.terms.forEach((term, i) => {
    y += 6;
    doc.setFontSize(12);
    doc.text(`Term ${i + 1}: ${term.termYear} - ${term.termSchoolName}`, 10, y);
    y += 6;
    doc.setFontSize(10);
    term.courses.forEach((course) => {
      doc.text(
        `• ${course.courseTitle} (${course.courseCode}) - Grade: ${course.grade}, Credits: ${course.creditEarned}`,
        12,
        y
      );
      y += 6;
      if (y > 280) {
        doc.addPage();
        y = 10;
      }
    });
  });

  doc.save(`${props.enrollment.student_full_name} - transcript.pdf`);
}
</script>

<style scoped lang="scss">
p {
  margin-bottom: 4px;
}

.btn-primary {
  padding: 8px 12px;
  background: $tenant-ui-new-accent-color;
  color: white;
  border: none;
  cursor: pointer;
}
</style>
