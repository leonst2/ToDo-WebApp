<template>
    <div style="text-align: left;">  
            <n-card style="width: 300px; height: 150px; margin-top: 20px; margin-bottom: 20px; margin-left: 25px; margin-right: 40px; border: none; text-align: left;" :title="name">
                <n-button   @click="listSelFunc?.(name)" style="width: 300px; height: 150px; margin-left: -20px; margin-top: -60px; position: absolute; "></n-button>
                {{ description }}
            </n-card>
            
          
    </div>
</template>

<script setup lang="ts">
    import { NButton,NCard } from 'naive-ui';

    // Simple Component just displays the passedDown Props
    // listSelFunc is used to load ListItems on clicking on a List

    const props = defineProps({
        name: String,
        description: String,
        listSelFunc: Function,
    })
</script>

<style scoped>

</style>