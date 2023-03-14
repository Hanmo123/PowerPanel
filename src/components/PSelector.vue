<script lang="ts" setup>
import {computed, nextTick} from 'vue';

const props = defineProps(['modelValue', 'list', 'readonly', 'onSelect']);
const emits = defineEmits(['update:modelValue']);

const model = computed({
    get() {
        return '#' + props.modelValue;
    },
    set(v) {
        if (props.readonly) return;
        if (v.lastIndexOf('#') > 0) return;
        emits('update:modelValue', v.slice(1));
        nextTick(() => emits('update:modelValue', v.slice(1).replace(' ', '')));
    }
});
</script>

<template>
    <n-mention v-model:value="model" :options="list" default-value="#" prefix="#" :on-select="onSelect"/>
</template>