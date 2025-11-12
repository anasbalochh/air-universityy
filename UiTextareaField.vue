<template>
    <div :class="['flex flex-col w-full', wrapperClass]">
        <div
            :class="[
                'relative w-full transition-opacity duration-150 cursor-text',
                disabled ? 'opacity-60 pointer-events-none' : 'opacity-100',
            ]"
            data-component="ui-textarea-field"
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

            <!-- Textarea Wrapper -->
            <div
                :class="[
                    'flex w-full min-h-14 items-start gap-3 border-2 rounded-lg px-4 transition-all duration-200 py-4',
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
                    class="flex shrink-0 items-center justify-center pt-1 text-xl text-slate-500"
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

                <!-- Textarea -->
                <textarea
                    ref="textareaRef"
                    :id="fieldId"
                    :name="name"
                    :rows="rows"
                    class="ui-field__control h-full min-h-[3.5rem] w-full resize-none bg-transparent text-base text-slate-900 outline-none placeholder-transparent"
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

                <!-- Right Icon -->
                <component
                    v-if="hasRightIcon"
                    :is="isRightIconInteractive ? 'button' : 'span'"
                    class="flex shrink-0 items-center justify-center pt-1 text-xl text-slate-500"
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
import { computed, ref, useSlots } from "vue";

const props = defineProps({
    id: String,
    label: String,
    modelValue: [String, Number],
    placeholder: String,
    rows: { type: Number, default: 4 },
    disabled: Boolean,
    readonly: Boolean,
    required: Boolean,
    hint: String,
    error: String,
    autocomplete: String,
    name: String,
    icon: String,
    iconPosition: { type: String, default: "right" },
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
    leftIconInteractive: { type: Boolean, default: undefined },
    rightIconInteractive: { type: Boolean, default: undefined },
});

const emit = defineEmits([
    "update:modelValue",
    "input",
    "change",
    "focus",
    "blur",
    "icon-left-click",
    "icon-right-click",
]);

const slots = useSlots();
const textareaRef = ref(null);
const focused = ref(false);

const generatedId = `ui-textarea-${Math.random().toString(36).slice(2, 10)}`;
const fieldId = computed(() => props.id || generatedId);

const normalizedValue = computed(
    () => (props.modelValue ?? "").toString()
);

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
    return typeof props.leftIconClick === "function";
});

const isRightIconInteractive = computed(() => {
    if (props.rightIconInteractive !== undefined) {
        return props.rightIconInteractive;
    }
    return typeof props.rightIconClick === "function";
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

const isActive = computed(() => {
    if (focused.value) return true;
    const value = normalizedValue.value;
    return value !== "" && value !== null && value !== undefined;
});

function isComponent(value) {
    return value && (typeof value === "object" || typeof value === "function");
}

function onInput(e) {
    const value = e.target.value;
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
    if (textareaRef.value && typeof textareaRef.value.focus === "function") {
        textareaRef.value.focus();
    }
}

function handleLeftIconClick(event) {
    emit("icon-left-click", event);
    if (typeof props.leftIconClick === "function") {
        props.leftIconClick(event);
    }
    if (isLeftIconInteractive.value) {
        focusInput();
    }
}

function handleRightIconClick(event) {
    emit("icon-right-click", event);
    if (typeof props.rightIconClick === "function") {
        props.rightIconClick(event);
    }
    if (isRightIconInteractive.value) {
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
