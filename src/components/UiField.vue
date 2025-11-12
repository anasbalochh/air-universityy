<template>
  <div :class="['ui-field flex flex-col w-full', wrapperClass]">
    <div
      :class="[
        'relative w-full transition-opacity duration-150',
        disabled ? 'opacity-60 pointer-events-none' : 'opacity-100'
      ]"
      :data-variant="variant"
    >
      <label
        v-if="label"
        :for="fieldId"
        :class="[
          'absolute -top-3 left-6 z-[1] inline-flex items-center gap-1 rounded-full border border-transparent bg-white px-2 text-xs font-semibold text-slate-500',
          labelClass
        ]"
      >
        {{ label }}
        <span v-if="required" class="text-rose-500">*</span>
      </label>

      <div
        :class="[
          'flex w-full items-center gap-3 rounded-[28px] border-2 bg-slate-50 px-6 transition-all duration-200',
          error ? 'border-rose-500 focus-within:border-rose-500 focus-within:ring-rose-100' : 'border-amber-500 focus-within:border-blue-500 focus-within:ring-blue-100',
          'focus-within:ring-4',
          variant === 'textarea' ? 'py-4' : 'py-3',
          inputWrapperClass
        ]"
        data-slot="input-wrapper"
      >
        <span
          v-if="hasLeftIcon"
          class="flex shrink-0 items-center justify-center text-xl text-slate-500"
          data-slot="icon-left"
        >
          <slot name="icon-left">
            <span v-if="icon" v-html="icon"></span>
          </slot>
        </span>

        <textarea
          v-if="variant === 'textarea'"
          :id="fieldId"
          :name="name"
          :rows="rows"
          class="ui-field__control h-full w-full resize-none bg-transparent text-base text-slate-900 outline-none placeholder:text-slate-400"
          :class="inputClass"
          :placeholder="placeholder"
          :autocomplete="autocomplete"
          :disabled="disabled"
          :readonly="readonly"
          :required="required"
          v-bind="inputAttrs"
          :value="normalizedValue"
          @input="onInput"
          @change="onChange"
          @focus="event => emit('focus', event)"
          @blur="event => emit('blur', event)"
        />

        <select
          v-else-if="variant === 'select'"
          v-model="selectBinding"
          :id="fieldId"
          :name="name"
          class="ui-field__control h-full w-full appearance-none bg-transparent text-base text-slate-900 outline-none placeholder:text-slate-400"
          :class="inputClass"
          :multiple="multiple"
          :disabled="disabled"
          :required="required"
          :autocomplete="autocomplete"
          v-bind="inputAttrs"
          @change="event => emit('change', event)"
          @focus="event => emit('focus', event)"
          @blur="event => emit('blur', event)"
        >
          <option
            v-if="placeholder && !multiple"
            disabled
            value=""
          >
            {{ placeholder }}
          </option>
          <template v-for="option in options" :key="optionKey(option)">
            <optgroup v-if="isGroup(option)" :label="option.label">
              <option
                v-for="child in option.options"
                :key="optionKey(child)"
                :value="child.value"
                :disabled="child.disabled"
              >
                {{ child.label }}
              </option>
            </optgroup>
            <option
              v-else
              :value="option.value"
              :disabled="option.disabled"
            >
              {{ option.label }}
            </option>
          </template>
        </select>

        <svg
          v-if="variant === 'select' && !multiple && !hasRightIcon"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 20 20"
          fill="currentColor"
          class="h-5 w-5 text-slate-500"
        >
          <path
            fill-rule="evenodd"
            d="M10 12a1 1 0 01-.707-.293l-3-3a1 1 0 111.414-1.414L10 9.586l2.293-2.293a1 1 0 111.414 1.414l-3 3A1 1 0 0110 12z"
            clip-rule="evenodd"
          />
        </svg>

        <input
          v-else
          :id="fieldId"
          :name="name"
          :type="resolvedType"
          class="ui-field__control h-full w-full bg-transparent text-base text-slate-900 outline-none placeholder:text-slate-400"
          :class="inputClass"
          :placeholder="placeholder"
          :autocomplete="autocomplete"
          :disabled="disabled"
          :readonly="readonly"
          :required="required"
          v-bind="inputAttrs"
          :value="normalizedValue"
          @input="onInput"
          @change="onChange"
          @focus="event => emit('focus', event)"
          @blur="event => emit('blur', event)"
        />

        <button
          v-if="togglePassword"
          type="button"
          class="flex h-10 w-10 items-center justify-center rounded-full text-slate-500 transition hover:text-slate-700 focus:outline-none"
          @click="showPassword = !showPassword"
          :aria-label="showPassword ? 'Hide password' : 'Show password'"
        >
          <svg
            v-if="!showPassword"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="h-5 w-5"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M1.5 12s3.75-7.5 10.5-7.5S22.5 12 22.5 12s-3.75 7.5-10.5 7.5S1.5 12 1.5 12z"
            />
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M12 15.75a3.75 3.75 0 100-7.5 3.75 3.75 0 000 7.5z"
            />
          </svg>
          <svg
            v-else
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="h-5 w-5"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M3.98 8.223A10.477 10.477 0 001.5 12s3.75 7.5 10.5 7.5a10.48 10.48 0 005.632-1.72M6.228 6.228A10.45 10.45 0 0112 4.5c6.75 0 10.5 7.5 10.5 7.5a10.499 10.499 0 01-4.477 4.688M6.228 6.228L3 3m3.228 3.228l11.544 11.544M17.772 17.772L21 21"
            />
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M9.755 9.755a3.75 3.75 0 014.49 4.49"
            />
          </svg>
        </button>

        <span
          v-else-if="hasRightIcon"
          class="flex shrink-0 items-center justify-center text-xl text-slate-500"
          data-slot="icon-right"
        >
          <slot name="icon-right">
            <span v-if="icon" v-html="icon"></span>
          </slot>
        </span>
      </div>
    </div>

    <p v-if="error" class="mt-2 text-sm font-medium text-rose-600">
      {{ error }}
    </p>
    <p v-else-if="hint" class="mt-2 text-sm text-slate-500">
      {{ hint }}
    </p>
  </div>
</template>

<script setup>
import { computed, ref, useSlots } from 'vue';

const props = defineProps({
  id: { type: String, default: null },
  label: { type: String, default: '' },
  modelValue: {
    type: [String, Number, Boolean, Array, Object, null],
    default: ''
  },
  type: { type: String, default: 'text' },
  variant: {
    type: String,
    default: 'input',
    validator: value => ['input', 'textarea', 'select'].includes(value)
  },
  placeholder: { type: String, default: '' },
  options: {
    type: Array,
    default: () => []
  },
  disabled: { type: Boolean, default: false },
  readonly: { type: Boolean, default: false },
  required: { type: Boolean, default: false },
  hint: { type: String, default: '' },
  error: { type: String, default: '' },
  rows: { type: Number, default: 4 },
  multiple: { type: Boolean, default: false },
  autocomplete: { type: String, default: '' },
  name: { type: String, default: '' },
  icon: { type: String, default: '' },
  iconPosition: {
    type: String,
    default: 'right',
    validator: value => ['left', 'right'].includes(value)
  },
  togglePassword: { type: Boolean, default: false },
  inputClass: {
    type: [String, Array, Object],
    default: ''
  },
  inputWrapperClass: {
    type: [String, Array, Object],
    default: ''
  },
  labelClass: {
    type: [String, Array, Object],
    default: ''
  },
  wrapperClass: {
    type: [String, Array, Object],
    default: ''
  },
  inputAttrs: {
    type: Object,
    default: () => ({})
  }
});

const emit = defineEmits(['update:modelValue', 'input', 'change', 'focus', 'blur']);

const slots = useSlots();

const generatedId = `ui-field-${Math.random().toString(36).slice(2, 10)}`;

const fieldId = computed(() => props.id || generatedId);

const showPassword = ref(false);

const resolvedType = computed(() => {
  if (props.type === 'password' && props.togglePassword) {
    return showPassword.value ? 'text' : 'password';
  }
  return props.type;
});

const normalizedValue = computed(() => props.modelValue ?? '');

const selectBinding = computed({
  get: () => {
    if (props.multiple) {
      return Array.isArray(props.modelValue) ? props.modelValue : [];
    }
    return props.modelValue ?? '';
  },
  set: value => {
    emit('update:modelValue', value);
  }
});

const hasLeftIcon = computed(() => {
  if (props.icon && props.iconPosition === 'left') return true;
  return Boolean(slots['icon-left']);
});

const hasRightIcon = computed(() => {
  if (props.icon && props.iconPosition === 'right') return true;
  return Boolean(slots['icon-right']);
});

function onInput(event) {
  let value = event.target.value;

  if (props.type === 'number') {
    value = value === '' ? '' : Number(value);
  }

  emit('update:modelValue', value);
  emit('input', event);
}

function onChange(event) {
  emit('change', event);
}

function optionKey(option) {
  return option?.value ?? option?.label ?? JSON.stringify(option);
}

function isGroup(option) {
  return Array.isArray(option?.options);
}
</script>

<style scoped>
.ui-field__control::-ms-reveal,
.ui-field__control::-ms-clear {
  display: none;
}
</style>
