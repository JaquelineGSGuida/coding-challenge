<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { RouterLink } from 'vue-router'
import { cuisines } from '@/constants/cuisines'

const query = ref('')
const recipes = ref([])
const error = ref(null)
const isLoading = ref(false)

const recipesPerPage = 5
const currentPage = ref(1)
const totalPages = ref(1)
const hasSearched = ref(false)

const selectedCuisine = ref('')

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
        cuisine: selectedCuisine.value,
        number: recipesPerPage,
        offset: (page - 1) * recipesPerPage,
      },
    })

    recipes.value = response.data.results
    totalPages.value = Math.ceil(response.data.totalResults / recipesPerPage)
  } catch (err) {
    error.value = 'Error fetching recipes. Please try again.'
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
  hasSearched.value = true
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
  <div class="min-vh-100 d-flex justify-content-center py-5 px-3 px-md-5">
    <div class="bg-white p-4 mx-auto shadow w-100" style="max-width: 1140px">
      <h1 class="font-cursive display-4 display-md-3 text-center my-4">
        Welcome! Search for a recipe:
      </h1>
      <div class="row gy-3 align-items-center mb-4">
        <div class="col-12 col-md-6">
          <div class="dropdown w-100">
            <button
              class="btn btn-outline-secondary text-start dropdown-toggle w-100"
              type="button"
              data-bs-toggle="dropdown"
            >
              {{ selectedCuisine || 'All Cuisines' }}
            </button>
            <ul class="dropdown-menu mt-1 shadow-sm w-100">
              <li>
                <button
                  class="dropdown-item w-100 text-start"
                  :class="{ active: selectedCuisine === '' }"
                  @click="selectedCuisine = ''"
                >
                  All Cuisines
                </button>
              </li>

              <li v-for="cuisine in cuisines" :key="cuisine">
                <button
                  class="dropdown-item w-100 text-start"
                  :class="{ active: selectedCuisine === cuisine }"
                  @click="selectedCuisine = cuisine"
                >
                  {{ cuisine }}
                </button>
              </li>
            </ul>
          </div>
        </div>

        <div
          class="col col-12 col-md-6 d-flex flex-column flex-md-row align-items-start align-items-md-center"
        >
          <input
            v-model="query"
            type="text"
            name="search"
            placeholder="Search recipe"
            class="form-control me-md-3 mb-2 mb-md-0"
            @keyup.enter="searchRecipes"
          />
          <button
            class="btn btn-outline-secondary w-100 w-md-auto"
            @click="searchRecipes"
            :disabled="isLoading"
          >
            Search
          </button>
        </div>
      </div>

      <div v-if="isLoading && !error" class="d-flex justify-content-center my-5">
        <div class="spinner-border text-secondary" role="status" style="width: 3rem; height: 3rem">
          <span class="visually-hidden">Loading...</span>
        </div>
      </div>
      <div v-if="error" class="error">{{ error }}</div>

      <div v-if="recipes.length > 0" class="recipes">
        <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 g-3 mb-3">
          <RouterLink :to="`/recipe/${recipe.id}`" v-for="recipe in recipes" :key="recipe.id">
            <div class="col mb-3 mb-sm-0 h-100">
              <div class="card h-100">
                <img :src="recipe.image" class="card-img-top" :alt="recipe.title" />
                <div class="card-body">
                  <h4>{{ recipe.title }}</h4>
                </div>
              </div>
            </div>
          </RouterLink>
        </div>

        <nav aria-label="Page navigation" class="d-flex justify-content-center mt-4">
          <ul class="pagination">
            <li class="page-item" :class="{ disabled: currentPage === 1 }">
              <button
                class="page-link"
                @click="changePage(currentPage - 1)"
                :disabled="currentPage === 1"
              >
                Previous
              </button>
            </li>

            <li class="page-item disabled">
              <span class="page-link"> Page {{ currentPage }} of {{ totalPages }} </span>
            </li>

            <li class="page-item" :class="{ disabled: currentPage === totalPages }">
              <button
                class="page-link"
                @click="changePage(currentPage + 1)"
                :disabled="currentPage === totalPages"
              >
                Next
              </button>
            </li>
          </ul>
        </nav>
      </div>
      <div v-else-if="!isLoading && hasSearched">
        <span>No recipes found.</span>
      </div>
    </div>
  </div>
</template>

<style></style>
