<script setup>
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'
import { useRoute } from 'vue-router'

const route = useRoute()
const recipe = ref(null)
const ingredients = ref([])
const isLoading = ref(false)
const error = ref(null)

/**
 * This calls the API and fetches the recipe
 * @param id - identifies the recipe to be fetched
 */
const fetchRecipe = async (id) => {
  error.value = null
  isLoading.value = true

  try {
    const apiKey = import.meta.env.VITE_SPOONACULAR_API_KEY

    const response = await axios.get(`https://api.spoonacular.com/recipes/${id}/information`, {
      params: {
        apiKey,
      },
    })

    recipe.value = response.data
    ingredients.value = response.data.extendedIngredients
  } catch (err) {
    error.value = 'Error fetching recipes.'
    console.error('Error fetching recipes: ', err)
  } finally {
    isLoading.value = false
  }
}

onMounted(() => {
  fetchRecipe(route.params.id)
})

watch(
  () => route.params.id,
  (newId) => {
    fetchRecipe(newId)
  },
)
</script>

<template>
  <div v-if="isLoading" class="loading">Loading...</div>
  <div class="recipe" v-if="recipe">
    <h2>{{ recipe.title }}</h2>
    <img :src="recipe.image" alt="" />
    <p>
      Vegetarian <span>{{ recipe.vegetarian ? '✅' : '❌' }}</span>
    </p>
    <p>
      Vegan <span>{{ recipe.vegan ? '✅' : '❌' }}</span>
    </p>
    <p>
      Gluten-Free <span>{{ recipe.glutenFree ? '✅' : '❌' }}</span>
    </p>
    <p>
      Dairy-Free <span>{{ recipe.dairyFree ? '✅' : '❌' }}</span>
    </p>
    <p>
      Very Healthy <span>{{ recipe.veryHealthy ? '✅' : '❌' }}</span>
    </p>
    <p>
      Low Fodmap <span>{{ recipe.lowFodmap ? '✅' : '❌' }}</span>
    </p>

    <h2>Ingredients</h2>
    <div v-for="ingredient in ingredients" :key="ingredient.id">
      {{ ingredient.measures.us.amount }} {{ ingredient.measures.us.unitShort }}
      <span v-if="ingredient.measures.metric.unitShort != ingredient.measures.us.unitShort">
        ({{ ingredient.measures.metric.amount }}{{ ingredient.measures.metric.unitShort }})
      </span>

      {{ ingredient.name }}
    </div>
    <h2>Instructions</h2>
    <p v-if="recipe.instructions" v-html="recipe.instructions"></p>
    <p v-else>No Instructions found.</p>
  </div>

  <div v-else-if="error">
    <p class="error">{{ error }}</p>
  </div>

  <RouterLink to="/" class="return-btn">
    <button>Go Home</button>
  </RouterLink>
</template>

<style></style>
