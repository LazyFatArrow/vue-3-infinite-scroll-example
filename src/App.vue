<template>
  <div id="app">
    <div v-if="isInitialRequestLoading">
      Loading users...
    </div>
    <ul class="users">
      <li
        v-for="user in users"
        :key="user.id"
        class="user"
      >
        <img :src="user.avatar" />
        <h3>{{ user.name }}</h3>
      </li>
    </ul>

    <div
      v-if="isLoading"
      class="loading-spinner"
    >
      Loading moar users
    </div>
    <div
      v-if="hasFetchedAllData"
      class="loading-spinner"
    >
      No More Data
    </div>
  </div>
</template>

<script>
  import { ref, onBeforeMount, onMounted, onBeforeUnmount, computed } from 'vue'

  export default {
    setup() {
      const users = ref([])
      const currentPage = ref(0)
      const totalPages = ref()
      const limit = 10
      const isInitialRequestLoading = ref(true)
      const isLoading = ref(false)

      const hasFetchedAllData = computed(() => {
        return currentPage.value === totalPages.value && !isLoading.value
      })

      const fetchUsers = async () => {
        currentPage.value++

        try {
          const response = await fetch(`http://localhost:3000/api/v1/users?limit=${limit}&page=${currentPage.value}`)
          const parsedResponse = await response.json()
  
          users.value = [
            ...users.value,
            ...parsedResponse.data
          ]
          totalPages.value = parsedResponse.totalPages
        } catch(err) {
          console.log(err)
        }
      }

      const handleScroll = async () => {
        if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) {
          console.log(hasFetchedAllData.value || isLoading.value)
          if (hasFetchedAllData.value || isLoading.value) {
            return
          }

          isLoading.value = true
          
          await fetchUsers()

          isLoading.value = false
        }
      }

      onBeforeMount(async () => {
        await fetchUsers()
        isInitialRequestLoading.value = false
      })

      onMounted(() => {
        window.addEventListener('scroll', handleScroll)
      })

      onBeforeUnmount(() => {
        window.removeEventListener('scroll', handleScroll)
      })

      return {
        users,
        isInitialRequestLoading,
        isLoading,
        hasFetchedAllData,
      }
    }
  }
</script>

<style scoped>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }

  .users {
    padding-bottom: 5rem;
  }

  .user {
    list-style: none;
  }

  .loading-spinner {
    padding-bottom: 3rem;
  }
</style>
