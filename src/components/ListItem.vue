<template>
    <div>
<n-list-item v-if="!update":class="{ notDone: complete }"> {{ text }}
                <template #suffix>
        <n-button-group><n-button ghost type="primary" ><n-checkbox v-model:checked="complete"></n-checkbox></n-button>
        <n-button @click="updateSwitch" ghost type="info">...</n-button>
        <n-button @click="emit('entryDeleted', props.id)" ghost type="error">X</n-button></n-button-group>
      </template>
            </n-list-item>
            <n-list-item v-if="update"> <n-space vertical><n-input v-model:value="updateToDo"/><n-date-picker value-format="yyyy-MM-dd" v-model:formatted-value="updateDate"/></n-space>
                <template #suffix>
                    <n-button-group>
                        <n-button @click="() => {
                            emit('entryUpdated', {Entry_ID: id, FinishDate: updateDate, Status: complete, Title: updateToDo})
                            updateSwitch()
                        }" ghost type="primary">Update</n-button>
                        <n-button @click="updateSwitch" ghost type="error">Cancel</n-button>
                    </n-button-group>
        
        
      </template>
            </n-list-item>
    </div>
</template>

<script setup lang="ts">
    import { NListItem, NButton, NButtonGroup, NCheckbox, NSpace, NInput, NDatePicker } from 'naive-ui';
import { onUpdated } from 'vue';
import { onMounted } from 'vue';
import { watch } from 'vue';
    import { ref } from 'vue';

    const props = defineProps({
        text:String,
        done: Boolean,
        id: Number,
        date: String,
    })

    const emit = defineEmits(['complChange', 'entryDeleted', 'entryUpdated'])

    const update = ref(false)
    const complete = ref(false)
    const updateToDo = ref('')
    const updateDate = ref('2024-01-01')

    // Needs to be both OnMounted and onUpdated here else only the old Array is loaded (don't know why this happens)
    onMounted(() => {
        console.log(props.done)
        complete.value = !props.done
    })
    onUpdated(() => {
        console.log(props.done)
        complete.value = !props.done
    })

    // Automatically update ListEntry if it is completed/uncompleted
    watch(complete,(complete) =>{
         emit('complChange', complete)
        emit('entryUpdated', {Entry_ID: props.id, FinishDate: props.date, Status: !complete, Title: props.text})
    })
    
    const updateSwitch = () => {
        updateDate.value = '2024-01-01'
        updateToDo.value = ''
        update.value = !update.value
    }
    
</script>

<style scoped>
    .notDone {
  text-decoration: line-through;
}
</style>