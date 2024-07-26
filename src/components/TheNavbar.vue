<template>
    <div id="flex">  
          <!-- Button to create new Page -->
          <n-button v-if="apiData?.username!==''" :on-click="addPage">New Page</n-button> 
          <!-- Scrollbar for if Menu gets to big -->
          <n-scrollbar x-scrollable>
          <!-- Menu to display the Pages-->
          <n-menu responsive :options="menuOptions" mode="horizontal" /></n-scrollbar>   
          <!-- UserMenu -->
          <n-dropdown :options="dropdownOptions" placement="bottom-start"><n-button ghost style=" align-self: center; margin-right: 10px;">Profile/PageSharing</n-button></n-dropdown>
    </div>
</template>

<script setup lang="ts">
// Imports
import {NMenu, type MenuOption, NButton,NSpace, useDialog, useMessage, NScrollbar, NDropdown, type DropdownOption} from 'naive-ui'
import { h, ref, type Ref } from 'vue';
import DialogInput from './DialogInput.vue';
import { onBeforeMount } from 'vue';
import { getSHA256Hash } from "boring-webcrypto-sha256"; 
import { onUpdated } from 'vue';

const props = defineProps({
  apiData: Object,
})

const emit = defineEmits(['pageSelect', 'userLogin', 'userLogout'])
// Dialog object to create Dialogs
const dialog = useDialog()
// message object to create mes
const message = useMessage()
// menuOptions Ref for the Menu on the Top
const menuOptions: Ref<MenuOption[]> = ref([{
                label: () =>
                  h(
                    'n-button',
                    {
                      onclick() {
                        selectPage(this)
                      }
                    },
                    'page1'
                  ),
                  key: 1
                },{
                label: () =>
                  h(
                    'n-button',
                    {
                      onclick() {
                        selectPage(this)
                      }
                    },
                    'page2'
                  ),
                  key: 2
                }])
// DropDownOptions Ref for Login/Logout Button 
const dropdownOptions: Ref<DropdownOption[]> = ref(props.apiData?.username===''?[ 
  {
          label: 'Login',
          key: 'login',
          props: {
            onClick: () => {
              login()
            }
          }
        },
        {
          label: 'Register',
          key: 'register',
          props: {
            onClick: () => {
              registerUser()
            }
          }
        },
]:[ 
  {
          label: 'Logout',
          key: 'logout',
          props: {
            onClick: () => {
              logout()
            }
          }
        },
]
)
// List of Page Names for checking if page name already exists
let pages = ref([{PageName: 'Page1', PageId: 1},{PageName: 'Page2', PageId: 2}])

// addPage func to create new Page
const addPage = () => {
  let newName = ''
  // create Input Dialog to enter new Name
  dialog.info({
          title: 'Enter page name:',
          content: ()=>h(DialogInput, {onNewText:(event)=>newName = event}),
          positiveText: 'Create',
          negativeText: 'Cancel',
          onPositiveClick: () => {
            // Check if page can be created
            let pageExists = false
            for (let index = 0; index < pages.value.length; index++) {
                if (newName === pages.value[index].PageName) {
                    pageExists = true
                } 
            }
    if (!pageExists) {
      // API Call
      fetch(props.apiData?.Url + '/addpage?username=' + props.apiData?.username + '&pagename=' + newName, {method: 'GET'})
              .then((response) => {
                if (response.status === 200) {
                  return response.json()
                }
                }
            )
            .then(
              (json) => {
                pages.value.splice(pages.value.length-1,0,{PageName: newName, PageId: json.newPage})
                // Update the options Array for the Menu to display new Page
                updatePages()
                message.success('Page has been created sucessfully.')
              }
            )
              .catch(error => {
                console.error('Error:', error);
              });
            }
            else{
                  message.error("Page with this name already exists. Please choose another name!")
                }
    },
          onNegativeClick: () => {
            
          }
        })
}

// Log User out trough API
const logout = () => {
    console.log('User logged out')
    fetch(props.apiData?.Url + '/logout?username=' + props.apiData?.username, {method: 'GET'})
              .then((response) => {
                if (response.ok === true) {
                  message.success('User has been logged out sucessfully')
                }
              })
              .catch(error => {
                console.error('Error:', error);
              });

    dropdownOptions.value = [ 
  {
          label: 'Login',
          key: 'login',
          props: {
            onClick: () => {
              login()
            }
          }
        },
        {
          label: 'Register',
          key: 'register',
          props: {
            onClick: () => {
              registerUser()
            }
          }
        },
]
// Emit to make App delete username because User logged out
emit('userLogout')
}

// Log User in trough API
const login = () => {
  let username = ''
  let password = ''
  dialog.info({
          title: 'Login',
          content: ()=>h(NSpace,[h(DialogInput, {onNewText:(event)=>username = event, placeholder:'Username',}),h(DialogInput, {onNewText:(event)=>password = event, placeholder:'Password',isPassword:true}),]),
          positiveText: 'Login',
          negativeText: 'Cancel',
          onPositiveClick: () => {
            getSHA256Hash(password)
            .then((pw) => {
              fetch(props.apiData?.Url + '/login?username=' + username + '&password=' + pw , {method: 'GET'})
              .then((response) => {
                if (response.status === 200) {
                  message.success('Logged in as ' + username)

                  emit('userLogin', username)
                  // Update DropDownOptions to display logout button
                  dropdownOptions.value = [ 
  {
          label: 'Logout',
          key: 'logout',
          props: {
            onClick: () => {
              logout()
            }
          }
        },
]
                }
                else{
                  message.error('Could not log you in. Check your username and password.')
                }
                }
            )
              .catch(error => {
                console.error('Error:', error);
              });
            }) 
          },
          onNegativeClick: () => {}})
}
// Register User trough API Call
const registerUser = () => {
  let firstname = ''
  let lastname = ''
  let email = ''
  let username = ''
  let password = ''
    dialog.info({
          title: 'Register User',
          content: ()=>h(NSpace,[h(DialogInput, {onNewText:(event)=>firstname = event, placeholder:'First Name'}),h(DialogInput, {onNewText:(event)=>lastname = event, placeholder:'LastName',}),h(DialogInput, {onNewText:(event)=>email = event, placeholder:'E-Mail',}),h(DialogInput, {onNewText:(event)=>username = event, placeholder:'Username',}),h(DialogInput, {onNewText:(event)=>password = event, placeholder:'Password',}),]),
          positiveText: 'Register',
          negativeText: 'Cancel',
          onPositiveClick: () => {
            getSHA256Hash(password)
            .then((pw) => {
              fetch(props.apiData?.Url + '/register?username=' + username + '&firstname=' + firstname + '&lastname=' + lastname + '&email=' + email + '&password=' + pw , {method: 'GET'})
              .then((response) => {
                if (response.status === 200) {
                  message.success('User has been created. Use your username and password to login.' )
                }
                else{
                  message.error('User with this username already exists.')
                }
                }
            )
              .catch(error => {
                console.error('Error:', error);
              });
            }) 
          },
          onNegativeClick: () => {}})
}
// Update the menuOptions Object to display all Pages as MenuItems
const updatePages = () => {
  menuOptions.value = []
    pages.value.forEach(page => {
      menuOptions.value.push( {
                label: () =>
                  h(
                    'n-button',
                    {
                      onclick() {
                        selectPage(this)
                      }
                    },
                    page.PageName
                  ),
                  key: page.PageId.valueOf()
                })
    });
}
// Function to run on slection of Page, emits pageId so Lists can be loaded
const selectPage = (btn: any) => {
  pages.value.forEach(page => {
    if (page.PageName === btn.innerHTML) {
      emit('pageSelect', page.PageId)
    }
  })
}
//Reload all pages on first open
onBeforeMount(() => {
  fetch(props.apiData?.Url + '/getallpages?username=' + props.apiData?.username, {method: 'GET'})
              .then((response) => {
                    return response.json()
                }
            )
            .then(
              (json) => {
                pages.value = json.pages
                updatePages()
              }
            )
              .catch(error => {
                console.error('Error:', error);
              });
})
// Reload all pages on Property Update aka. User Login
onUpdated(() => {
  fetch(props.apiData?.Url + '/getallpages?username=' + props.apiData?.username, {method: 'GET'})
              .then((response) => {
                    return response.json()
                }
            )
            .then(
              (json) => {
                pages.value = json.pages
                updatePages()
              }
            )
              .catch(error => {
                console.error('Error:', error);
              });
})
</script>

<style scoped>
    #flex{
      display: flex;
      border-bottom: 2px rgba(84, 84, 84, 0.65) solid;
      height: fit-content;
      padding: 10px 0px;
    }
    
</style>