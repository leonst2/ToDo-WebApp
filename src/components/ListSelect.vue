<template>
    <div>
        <n-flex>
            <TheList 
            :list-sel-func="selectList" 
            v-for="list in lists" 
            :name="list.ListName.valueOf()" 
            :description="list.Description.valueOf()"></TheList>
            <n-card size="small" style="width: 300px; height: 150px; margin-top: 28px; margin-bottom: 20px; margin-left: 25px; margin-right: 40px; border: 1px solid rgba(255, 255, 255, 0.24); text-align: left;"   title="">
                <n-space vertical>
                <n-input v-model:value="newListName" maxlength="30" show-count clearable placeholder="List Name"/>
                <n-input size="tiny" type="textarea" maxlength="80" show-count clearable placeholder="Description" :autosize="{
        minRows: 2,
        maxRows: 2
      }" v-model:value="newListDescr"/>
      <n-button type="primary" @click="addList(lists)">Create New List</n-button>
                </n-space>
            </n-card>
        </n-flex>
    </div>
</template>

<script setup lang="ts">
    import { ref, watch } from 'vue';
import TheList from './TheList.vue';
    import { NFlex, NCard, NButton,NSpace, NInput, useMessage} from 'naive-ui';


const emit = defineEmits(['listSelect'])

    const message = useMessage()

    

    const newListName = ref('')
    const newListDescr = ref('')

    const props = defineProps({
        lists: Array<{ListName: String, Description: String, ListId: Number}>,
        curPage: Number,
        apiData: Object,
    })

    // Method to add new List
    const addList = (lists: any) => {
      // Check if List already exists
        let listExists = false
    for (let index = 0; index < lists.length; index++) {
        if (newListName.value === lists[index].name) {
            listExists = true
        } 
    }
    if (!listExists) {
      // API Call
         fetch(props.apiData?.Url + '/addlist?pageid=' + props.curPage + '&description=' + newListDescr.value + '&listname=' + newListName.value + '&username=' + props.apiData?.username, {method: 'GET'})
              .then((response) => {
                if (response.status === 200) {
                  return response.json()
                }
                }
            )
            .then(
              (json) => {
                lists.push({ListName: newListName.value, Description: newListDescr.value, ListId: json.newList})
                message.success('List has been created sucessfully.')
                newListName.value = ''
                newListDescr.value = ''
              }
              
            )
              .catch(error => {
                console.error('Error:', error);
              });
            }
            else{
                message.error('List with this name already exists. Please choose another name!')
                newListName.value = ''
                newListDescr.value = ''
            }
        
    }

// Function to run on List being selected, so that List Entries can be loaded
const selectList = (name: String) => {
    console.log(name)
       props.lists?.forEach(list => {
    if (list.ListName === name) {
      emit('listSelect', {ID: list.ListId, Name: name})
    }
  })
}
             

            
        

</script>

<style scoped>

</style>