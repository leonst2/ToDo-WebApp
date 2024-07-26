<template>
    <div>
        <h1>{{ listName }}</h1>
        <n-divider></n-divider>
        <n-list hoverable bordered>
        <ListItem 
            v-for="entry in entries" 
            :id="entry.Entry_ID.valueOf()" 
            @entry-updated="(updatedData) => updateEntry(updatedData, entries)" 
            @entry-deleted="(ID) => deleteEntry(ID, entries)" 
            :text="entry.Title.valueOf()" 
            @compl-change="(complete) => {entry.Status = !complete}" 
            :done="entry.Status.valueOf()"
            :date="entry.FinishDate.valueOf()">
        </ListItem>
            <n-list-item > <n-space vertical><n-input v-model:value="newToDo"/><n-date-picker value-format="yyyy-MM-dd" v-model:formatted-value="newDate"/></n-space>
                <template #suffix>
        <n-button @click="createEntry(entries)" ghost type="primary">Create</n-button>
        
      </template>
            </n-list-item>
        </n-list>
    </div>
</template>

<script setup lang="ts">
    import { NList, NListItem, NDivider,NButton, NInput, NDatePicker,NSpace } from 'naive-ui';
    import { ref } from 'vue';
    import ListItem from './ListItem.vue';

const newToDo = ref('')
const newDate = ref('2024-01-01')

    const props = defineProps({
        entries: Array<{Entry_ID: Number, FinishDate: String, Status: Boolean, Title: String}>,
        listID: Number,
        listName: String,
        apiData: Object,
    })
     // Method to delete exisitng Entry
    const deleteEntry = (ID: Number, entries: any) => {
        // API Call to delete List Entry with given Entry ID.
        fetch(props.apiData?.Url + '/removelistentry?entryid=' + ID + '&username=' + props.apiData?.username, {method: 'GET'})
              .then((response) => {
                if (response.ok === true) {    
                    if (entries) {
                for (let index = 0; index < entries.length; index++) {
                console.log(index)
                if (entries[index].Entry_ID === ID) {
                    entries.splice(index, 1)
                }
            }
        } 
                }})
              .catch(error => {
                console.error('Error:', error);
              });
    }
    // Method to update exisitng Entry
    const updateEntry = (updateData: {Entry_ID: Number, FinishDate: String, Status: Boolean, Title: String}, entries: any)=>{
        // API Call to update List Entry with given Entry ID
        fetch(props.apiData?.Url + '/updatelistentry?entryid=' + updateData.Entry_ID + '&status=' + updateData.Status + '&title=' + updateData.Title +'&finishdate=' + updateData.FinishDate + '&username=' + props.apiData?.username, {method: 'GET'})
              .then((response) => {
                if (response.ok === true) { 
                    if (entries) {
                for (let index = 0; index < entries.length; index++) {
                if (entries[index].Entry_ID === updateData.Entry_ID) {
                    entries.splice(index, 1, updateData)
                }
            }
        } 
                }
        })
              .catch(error => {
                console.error('Error:', error);
              });
    }
    // Method to create new Entry
    const createEntry = (entries: any) => {
        // Check if List with given Name already exists
        let entryExists = false
        for (let index = 0; index < entries.length; index++) {
            if (newToDo.value === entries[index].name) {
                entryExists = true
            } 
        }
        if (!entryExists) {
        // API Call
         fetch(props.apiData?.Url + '/addlistentry?title=' + newToDo.value + '&finishdate=' + newDate.value 
         + '&listid=' + props.listID + '&username=' + props.apiData?.username, {method: 'GET'})
              .then((response) => {
                if (response.status === 200) {
                  return response.json()
                }
                }
            )
            .then(
              (json) => {
                // Update Website after sucessful response
                entries.push({Title: newToDo.value, FinishDate: newDate.value, Entry_ID: json.newEntry, Status: true})
                newToDo.value = ''
                newDate.value = '2024-01-01'
              }
              
            )
              .catch(error => {
                console.error('Error:', error);
              });
            }
    }
</script>

<style scoped>
    h1{
        color: rgba(255, 255, 255, 0.82);
        margin-top: 20px;
        margin-left: 30px;
    }
    
</style>