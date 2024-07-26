<script setup lang="ts">
import TheNavbar from './components/TheNavbar.vue';
import ListSelect from './components/ListSelect.vue';
import ListView from './components/ListView.vue'
import { NConfigProvider, darkTheme, NDialogProvider,NMessageProvider } from 'naive-ui';
import { ref } from 'vue';
import { onBeforeMount } from 'vue';

// apiData Object for easy Access to the base Api Url and the username of the User that is currently logged in
const apiData = ref({Url: 'http://leon-stiffel.de:5000/api/todo', username: 'first'})

const theme = darkTheme

// refs with Object Arrays to render Lists and ListEntries
const lists = ref([{ListName: 'Test1234', Description: 'Something', ListId: 1},{ListName: 'Test1234', Description: 'Something', ListId: 1}])
const entries = ref(new Array<{Entry_ID: Number, FinishDate: String, Status: Boolean, Title: String}>)
// refs for saving which Page and List are currently open (Also used for conditional Rendering)
const currentPage = ref(1)
const currentList = ref(NaN)
const currentListName = ref('')

// Gets all Lists under Page with given ID
function loadLists(ID: Number){
  // API Request
  fetch(apiData.value.Url  + '/getalllists?username=' + apiData.value.username + '&pageid=' + ID, {method: 'GET'})
              .then((response) => {
                    return response.json()
              })
              .then(
                (json) => {
                  lists.value = json.lists
                  console.log(lists.value)
                }
              )
              .catch(error => {
                console.error('Error:', error);
              });
}
// Gets all Entries for List with given ID
function loadEntries(ID: Number){
  fetch(apiData.value.Url + '/getalllistentries?listid=' + ID + '&username=' + apiData.value.username, {method: 'GET'})
              .then((response) => {
                    return response.json()
                }
            )
            .then(
              (json) => {
                entries.value = json.listEntries
                console.log(entries.value)
              }
            )
              .catch(error => {
                console.error('Error:', error);
              });
}
// On page reload get currently logged in User
onBeforeMount(() => {
  let data = localStorage.getItem('ApiUsername')
  if (data !== null) {
    apiData.value.username = JSON.parse(data)
  }
  
})
// Function to save username in localStorage
function saveApiData(){
  localStorage.setItem('ApiUsername', JSON.stringify(apiData.value.username))
}
</script>

<template>
  <!-- Switch Theme here -->
  <n-config-provider :theme="theme">
    <div id="main">
      <!-- Message Provider needed to display Naive UI Message -->
      <n-message-provider>
        <header>
          <!-- Dialog Provider needed to display Naive UI Dialogs -->
          <n-dialog-provider>
            <TheNavbar @user-login="(username) => {
              apiData.username = username
              saveApiData()
            }" 
            @user-logout="() => {
              apiData.username = ''
              saveApiData()
              currentPage = NaN
              currentList = NaN
            }"
            :api-data="apiData" 
            @pageSelect="(pageID)=>{
              currentPage = pageID
              currentList = NaN
            loadLists(pageID)
            }">
            </TheNavbar>
    </n-dialog-provider>
    
  </header>

  <main>
    <!-- v-if in here to switch between the ListSelect and the ListView depending on if a List is selected -->
  <ListSelect 
    :api-data="apiData" 
    v-if="currentPage >= 0 && isNaN(currentList)" 
    :lists="lists"
    :curPage="currentPage" 
    @listSelect="(listData) => {
      currentList = listData.ID
      currentListName = listData.Name
      loadEntries(listData.ID)
   }">
  </ListSelect>
  <ListView 
   :api-data="apiData" 
   v-if="currentList >= 0" 
   :entries="entries" 
   :listID="currentList" 
   :listName="currentListName">
  </ListView>
  </main>
  </n-message-provider>
  </div>
  </n-config-provider>
</template>

<style scoped>
  #main{
    height: 100vh;
  }
</style>
