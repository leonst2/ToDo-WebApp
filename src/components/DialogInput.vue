<template>
    <div>
       <n-input ghost v-model:value="text" 
        :placeholder="placeholder"
        :type="isPassword?'password':'text'"/>
    </div>
</template>

<script setup lang="ts">
    import { watch,ref } from 'vue';
    import { NInput } from 'naive-ui';

// Just a small vmodel Implementation because vmodel does not work in NaiveUI dialogs

    const text = ref('')
    
    const props = defineProps({
        placeholder: String,
        isPassword: Boolean,
    })

    const emit = defineEmits(['newText'])

    watch(text, (newText)=> {
        emit('newText', newText)
    })
</script>

<style scoped>

</style>