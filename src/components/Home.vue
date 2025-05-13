<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { RouterLink } from 'vue-router'

const query = ref('')
const recipes = ref([])
const error = ref(null)
const isLoading = ref(false)

const recipesPerPage = 5
let currentPage = ref(1)
let totalPages = ref(1)

/**
 * This calls the API and fetches the recipes already considering the limit
 * of recipes per page
 * @param page - used to calculate the offset in the API call
 */
const fetchRecipes = async (page = 1) => {
  error.value = null
  isLoading.value = true

  try {
    const apiKey = import.meta.env.VITE_SPOONACULAR_API_KEY

    const response = await axios.get(`https://api.spoonacular.com/recipes/complexSearch`, {
      params: {
        apiKey,
        query: query.value,
        number: recipesPerPage,
        offset: (page - 1) * recipesPerPage,
      },
    })

    recipes.value = response.data.results
    totalPages.value = Math.ceil(response.data.totalResults / recipesPerPage)
  } catch (err) {
    error.value = 'Error fetching recipes.'
    console.error('Error fetching recipes: ', err)
  } finally {
    isLoading.value = false
  }
}

/**
 * This reset the currentPage value and calls the fetchRecipes function
 */
const searchRecipes = () => {
  currentPage.value = 1
  fetchRecipes(currentPage.value)
}

/**
 * This changes the current page and calls the fetchRecipes function again
 * @param page - the page number to be changed to
 */
const changePage = (page) => {
  if (page > 0 && page <= totalPages.value) {
    currentPage.value = page
    fetchRecipes(currentPage.value)
  }
}
</script>

<template>
  <h1>Welcome! Search for a recipe</h1>
  <div class="search-area">
    <input
      v-model="query"
      type="text"
      name="search"
      placeholder="Search recipe"
      @keyup.enter="searchRecipes"
    />
    <button @click="searchRecipes" :disabled="isLoading">Search</button>
  </div>

  <div v-if="isLoading" class="loading">Loading...</div>
  <div v-if="error" class="error">{{ error }}</div>

  <div v-if="recipes.length > 0" class="recipes">
    <RouterLink :to="`/recipe/${recipe.id}`" v-for="recipe in recipes" :key="recipe.id">
      <img :src="recipe.image" alt="" />
      {{ recipe.title }}
    </RouterLink>

    <div class="pagination">
      <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">Previous</button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages">
        Next
      </button>
    </div>
  </div>
  <div v-else-if="!isLoading">
    <span>No recipes found.</span>
  </div>
</template>

<style></style>
