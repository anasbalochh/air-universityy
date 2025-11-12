<template>
  <div class="space-y-6 rounded-3xl bg-slate-100 p-8">
    <UiField
      label="Email"
      v-model="form.email"
      placeholder="liamoliver@gmail.com"
      autocomplete="email"
      appearance="filled"
      :inputWrapperClass="'bg-[#F8F6F0]'"
    />

    <UiField
      label="Password"
      v-model="form.password"
      type="password"
      :togglePassword="true"
      placeholder="********"
      autocomplete="current-password"
      hint="At least 8 characters."
      appearance="filled"
      :inputWrapperClass="'bg-[#F8F6F0]'"
    />

    <UiField
      label="Username"
      v-model="form.username"
      placeholder="Choose a username"
      appearance="filled"
      :inputWrapperClass="'bg-[#F8F6F0]'"
    >
      <template #icon-left>
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
          fill="currentColor"
          class="h-5 w-5"
        >
          <path
            fill-rule="evenodd"
            d="M12 2.25a4.5 4.5 0 00-3.58 7.26A7.5 7.5 0 004.5 16.5v1.125a2.625 2.625 0 002.625 2.625h9.75a2.625 2.625 0 002.625-2.625V16.5a7.5 7.5 0 00-3.92-6.99A4.5 4.5 0 0012 2.25zm-2.25 4.5a2.25 2.25 0 114.5 0 2.25 2.25 0 01-4.5 0zM7.5 16.5a4.5 4.5 0 019 0v1.125a.375.375 0 01-.375.375h-8.25a.375.375 0 01-.375-.375V16.5z"
            clip-rule="evenodd"
          />
        </svg>
      </template>
    </UiField>

    <UiField
      label="About you"
      variant="textarea"
      v-model="form.about"
      rows="4"
      placeholder="Tell us about yourself"
      appearance="filled"
      :inputWrapperClass="'bg-[#F8F6F0]'"
    />

    <UiField
      label="Department"
      variant="select"
      v-model="form.department"
      placeholder="Select a department"
      :options="departmentOptions"
      appearance="filled"
      :inputWrapperClass="'bg-[#F8F6F0]'"
    />
  </div>
</template>

<script setup>
import { reactive } from 'vue';
import UiField from './UiField.vue';

const form = reactive({
  email: 'liamoliver@gmail.com',
  password: 'password123',
  username: '',
  about: '',
  department: ''
});

const departmentOptions = [
  { label: 'Product Design', value: 'design' },
  { label: 'Engineering', value: 'engineering' },
  { label: 'Marketing', value: 'marketing' },
  {
    label: 'Leadership',
    options: [
      { label: 'Operations', value: 'operations' },
      { label: 'People', value: 'people' }
    ]
  }
];
</script>
