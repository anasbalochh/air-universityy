<template>
    <div :class="['flex flex-col w-full', wrapperClass]">
        <div
            :class="[
                'relative w-full transition-opacity duration-150 cursor-text',
                disabled ? 'opacity-60 pointer-events-none' : 'opacity-100',
            ]"
            data-component="ui-input-field"
            @click="focusInput"
        >
            <!-- Floating Label -->
            <label
                v-if="label"
                :for="fieldId"
                :class="[
                    'absolute left-4 z-1 inline-flex items-center gap-1 text-[#7B7B7B] transform -translate-y-[50%] transition-all duration-200 pointer-events-none',
                    {
                        'top-[20%] text-xs': isActive,
                        'top-[50%] text-base': !isActive,
                    },
                    labelClass,
                ]"
            >
                {{ label }}
                <span v-if="required" class="text-rose-500">*</span>
            </label>

            <!-- Input Wrapper -->
            <div
                :class="[
                    'flex w-full min-h-14 items-center gap-3 border-2 rounded-lg px-4 transition-all duration-200 py-3 pb-2.5',
                    error
                        ? 'border-rose-500 focus-within:border-rose-500 focus-within:ring-2 focus-within:ring-rose-200'
                        : 'border-[#B59B5A] focus-within:ring-2 focus-within:ring-[#B59B5A]/30',
                    inputWrapperClass,
                ]"
                data-slot="input-wrapper"
            >
                <!-- Left Icon -->
                <component
                    v-if="hasLeftIcon"
                    :is="isLeftIconInteractive ? 'button' : 'span'"
                    class="flex shrink-0 items-center justify-center text-xl text-slate-500"
                    :class="[
                        leftIconClass,
                        isLeftIconInteractive
                            ? 'ui-field__icon-btn cursor-pointer focus:outline-none'
                            : '',
                    ]"
                    v-bind="leftIconAttrsComputed"
                    @click="handleLeftIconWrapperClick"
                >
                    <slot
                        name="icon-left"
                        :password-visible="passwordVisibility"
                        :toggle-password="handleTogglePassword"
                        :focus-input="focusInput"
                    >
                        <component
                            v-if="leftIconContent && isComponent(leftIconContent)"
                            :is="leftIconContent"
                            v-bind="leftIconPropsData"
                        />
                        <span
                            v-else-if="typeof leftIconContent === 'string'"
                            v-html="leftIconContent"
                        />
                        <span
                            v-else-if="icon && iconPosition === 'left'"
                            v-html="icon"
                        />
                    </slot>
                </component>

                <!-- Input -->
                <input
                    ref="inputRef"
                    :id="fieldId"
                    :name="name"
                    :type="resolvedType"
                    class="ui-field__control h-full w-full bg-transparent text-base text-black outline-none placeholder-transparent"
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
                    @focus="onFocus"
                    @blur="onBlur"
                />

                <!-- Toggle Password -->
                <button
                    v-if="shouldShowPasswordToggle"
                    class="ui-field__icon-btn flex h-10 w-10 items-center justify-center rounded-full text-slate-500 transition hover:text-slate-700 focus:outline-none"
                    :class="togglePasswordClass"
                    v-bind="togglePasswordPropsComputed"
                    @click.stop="handleTogglePassword($event, 'toggle-button')"
                    :aria-pressed="passwordVisibility"
                    :aria-label="
                        passwordVisibility
                            ? togglePasswordAriaLabelsComputed.hide
                            : togglePasswordAriaLabelsComputed.show
                    "
                >
                    <slot
                        name="toggle-password"
                        :password-visible="passwordVisibility"
                        :toggle-password="handleTogglePassword"
                    >
                        <component
                            v-if="currentToggleIcon && isComponent(currentToggleIcon)"
                            :is="currentToggleIcon"
                            v-bind="togglePasswordIconPropsData"
                        />
                        <span
                            v-else-if="typeof currentToggleIcon === 'string'"
                            v-html="currentToggleIcon"
                        />
                        <template v-else>
                            <svg
                                v-if="!passwordVisibility"
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
                        </template>
                    </slot>
                </button>

                <!-- Right Icon -->
                <component
                    v-if="hasRightIcon"
                    :is="isRightIconInteractive ? 'button' : 'span'"
                    class="flex shrink-0 items-center justify-center text-xl text-slate-500"
                    :class="[
                        rightIconClass,
                        isRightIconInteractive
                            ? 'ui-field__icon-btn cursor-pointer focus:outline-none'
                            : '',
                    ]"
                    v-bind="rightIconAttrsComputed"
                    @click="handleRightIconWrapperClick"
                >
                    <slot
                        name="icon-right"
                        :password-visible="passwordVisibility"
                        :toggle-password="handleTogglePassword"
                        :focus-input="focusInput"
                    >
                        <component
                            v-if="rightIconContent && isComponent(rightIconContent)"
                            :is="rightIconContent"
                            v-bind="rightIconPropsData"
                        />
                        <span
                            v-else-if="typeof rightIconContent === 'string'"
                            v-html="rightIconContent"
                        />
                        <span
                            v-else-if="icon && iconPosition === 'right'"
                            v-html="icon"
                        />
                    </slot>
                </component>
            </div>
        </div>

        <!-- Error / Hint -->
        <p v-if="error" class="mt-2 text-sm font-medium text-rose-600">
            {{ error }}
        </p>
        <p v-else-if="hint" class="mt-2 text-sm text-slate-500">
            {{ hint }}
        </p>
    </div>
</template>

<script setup>
import { computed, ref, useSlots, watch } from "vue";

const props = defineProps({
    id: String,
    label: String,
    modelValue: [String, Number, Boolean],
    type: { type: String, default: "text" },
    placeholder: String,
    disabled: Boolean,
    readonly: Boolean,
    required: Boolean,
    hint: String,
    error: String,
    autocomplete: String,
    name: String,
    icon: String,
    iconPosition: { type: String, default: "right" },
    togglePassword: Boolean,
    inputClass: [String, Array, Object],
    inputWrapperClass: [String, Array, Object],
    labelClass: [String, Array, Object],
    wrapperClass: [String, Array, Object],
    inputAttrs: { type: Object, default: () => ({}) },
    leftIcon: [String, Object, Function],
    rightIcon: [String, Object, Function],
    leftIconProps: { type: Object, default: () => ({}) },
    rightIconProps: { type: Object, default: () => ({}) },
    leftIconAttrs: { type: Object, default: () => ({}) },
    rightIconAttrs: { type: Object, default: () => ({}) },
    leftIconClass: [String, Array, Object],
    rightIconClass: [String, Array, Object],
    leftIconClick: Function,
    rightIconClick: Function,
    leftIconTogglePassword: Boolean,
    rightIconTogglePassword: Boolean,
    leftIconInteractive: { type: Boolean, default: undefined },
    rightIconInteractive: { type: Boolean, default: undefined },
    passwordVisible: { type: Boolean, default: undefined },
    initialPasswordVisible: { type: Boolean, default: false },
    togglePasswordClass: [String, Array, Object],
    togglePasswordProps: { type: Object, default: () => ({}) },
    togglePasswordAriaLabels: {
        type: Object,
        default: () => ({
            show: "Show password",
            hide: "Hide password",
        }),
    },
    togglePasswordIcons: {
        type: Object,
        default: () => ({
            show: null,
            hide: null,
        }),
    },
    togglePasswordIconProps: { type: Object, default: () => ({}) },
});

const emit = defineEmits([
    "update:modelValue",
    "input",
    "change",
    "focus",
    "blur",
    "icon-left-click",
    "icon-right-click",
    "toggle-password",
    "toggle-password-click",
    "update:passwordVisible",
]);

const slots = useSlots();
const inputRef = ref(null);
const focused = ref(false);

const generatedId = `ui-input-${Math.random().toString(36).slice(2, 10)}`;
const fieldId = computed(() => props.id || generatedId);

const passwordVisibleState = ref(
    props.passwordVisible ?? props.initialPasswordVisible
);

watch(
    () => props.passwordVisible,
    (value) => {
        if (value !== undefined) {
            passwordVisibleState.value = value;
        }
    }
);

const passwordVisibility = computed({
    get: () =>
        props.type === "password"
            ? props.passwordVisible ?? passwordVisibleState.value
            : false,
    set: (value) => {
        if (props.passwordVisible === undefined) {
            passwordVisibleState.value = value;
        }
        emit("update:passwordVisible", value);
    },
});

const resolvedType = computed(() => {
    if (props.type === "password") {
        return passwordVisibility.value ? "text" : "password";
    }
    return props.type;
});

const normalizedValue = computed(() => props.modelValue ?? "");

const hasLeftIcon = computed(
    () =>
        !!slots["icon-left"] ||
        !!props.leftIcon ||
        (props.icon && props.iconPosition === "left")
);

const hasRightIcon = computed(
    () =>
        !!slots["icon-right"] ||
        !!props.rightIcon ||
        (props.icon && props.iconPosition === "right")
);

const leftIconContent = computed(() => {
    if (slots["icon-left"]) return null;
    if (props.leftIcon) return props.leftIcon;
    if (props.icon && props.iconPosition === "left") return props.icon;
    return null;
});

const rightIconContent = computed(() => {
    if (slots["icon-right"]) return null;
    if (props.rightIcon) return props.rightIcon;
    if (props.icon && props.iconPosition === "right") return props.icon;
    return null;
});

const isLeftIconInteractive = computed(() => {
    if (props.leftIconInteractive !== undefined) {
        return props.leftIconInteractive;
    }
    return (
        props.leftIconTogglePassword || typeof props.leftIconClick === "function"
    );
});

const isRightIconInteractive = computed(() => {
    if (props.rightIconInteractive !== undefined) {
        return props.rightIconInteractive;
    }
    return (
        props.rightIconTogglePassword || typeof props.rightIconClick === "function"
    );
});

const leftIconAttrsComputed = computed(() => {
    const attrs = { ...(props.leftIconAttrs || {}) };
    if (isLeftIconInteractive.value) {
        attrs.type = attrs.type ?? "button";
        if (props.disabled || props.readonly) {
            attrs.disabled = true;
        }
    }
    return attrs;
});

const rightIconAttrsComputed = computed(() => {
    const attrs = { ...(props.rightIconAttrs || {}) };
    if (isRightIconInteractive.value) {
        attrs.type = attrs.type ?? "button";
        if (props.disabled || props.readonly) {
            attrs.disabled = true;
        }
    }
    return attrs;
});

const leftIconPropsData = computed(() => props.leftIconProps || {});
const rightIconPropsData = computed(() => props.rightIconProps || {});
const togglePasswordIconPropsData = computed(
    () => props.togglePasswordIconProps || {}
);

const togglePasswordAriaLabelsComputed = computed(() => ({
    show: props.togglePasswordAriaLabels?.show ?? "Show password",
    hide: props.togglePasswordAriaLabels?.hide ?? "Hide password",
}));

const shouldShowPasswordToggle = computed(
    () => props.type === "password" && props.togglePassword
);

const currentToggleIcon = computed(() => {
    const icons = props.togglePasswordIcons || {};
    return passwordVisibility.value ? icons.hide ?? null : icons.show ?? null;
});

const togglePasswordPropsComputed = computed(() => {
    const attrs = { ...(props.togglePasswordProps || {}) };
    attrs.type = attrs.type ?? "button";
    if (props.disabled || props.readonly) {
        attrs.disabled = true;
    }
    return attrs;
});

const isActive = computed(() => {
    if (focused.value) return true;
    const value = normalizedValue.value;
    if (Array.isArray(value)) {
        return value.length > 0;
    }
    return value !== "" && value !== null && value !== undefined;
});

function isComponent(value) {
    return value && (typeof value === "object" || typeof value === "function");
}

function onInput(e) {
    let value = e.target.value;
    if (props.type === "number") {
        value = value === "" ? "" : Number(value);
    }
    emit("update:modelValue", value);
    emit("input", e);
}

function onChange(e) {
    emit("change", e);
}

function onFocus(e) {
    focused.value = true;
    emit("focus", e);
}

function onBlur(e) {
    focused.value = false;
    emit("blur", e);
}

function focusInput() {
    if (props.disabled) return;
    if (inputRef.value && typeof inputRef.value.focus === "function") {
        inputRef.value.focus();
    }
}

function handleTogglePassword(event, source = "toggle-button") {
    if (props.disabled || props.readonly || props.type !== "password") {
        return;
    }
    if (event?.preventDefault) {
        event.preventDefault();
    }
    emit("toggle-password-click", { event, source });
    const next = !passwordVisibility.value;
    passwordVisibility.value = next;
    emit("toggle-password", { visible: next, event, source });
    focusInput();
}

function handleLeftIconClick(event) {
    emit("icon-left-click", event);
    if (typeof props.leftIconClick === "function") {
        props.leftIconClick(event);
    }
    if (props.leftIconTogglePassword && props.type === "password") {
        handleTogglePassword(event, "left-icon");
    } else if (isLeftIconInteractive.value) {
        focusInput();
    }
}

function handleRightIconClick(event) {
    emit("icon-right-click", event);
    if (typeof props.rightIconClick === "function") {
        props.rightIconClick(event);
    }
    if (props.rightIconTogglePassword && props.type === "password") {
        handleTogglePassword(event, "right-icon");
    } else if (isRightIconInteractive.value) {
        focusInput();
    }
}

function handleLeftIconWrapperClick(event) {
    if (props.disabled) {
        event?.preventDefault?.();
        return;
    }
    if (isLeftIconInteractive.value) {
        event?.preventDefault?.();
        event?.stopPropagation?.();
        handleLeftIconClick(event);
    }
}

function handleRightIconWrapperClick(event) {
    if (props.disabled) {
        event?.preventDefault?.();
        return;
    }
    if (isRightIconInteractive.value) {
        event?.preventDefault?.();
        event?.stopPropagation?.();
        handleRightIconClick(event);
    }
}
</script>

<style scoped>
.ui-field__control::-ms-reveal,
.ui-field__control::-ms-clear {
    display: none;
}

.ui-field__icon-btn {
    border: none;
    background: transparent;
    padding: 0;
}

.ui-field__icon-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}
</style>
