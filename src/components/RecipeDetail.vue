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
  <div class="min-vh-100 d-flex justify-content-center py-5 px-3 px-md-5">
    <div class="bg-white p-4 mx-auto shadow w-100" style="max-width: 1140px">
      <div v-if="isLoading && !error" class="d-flex justify-content-center my-5">
        <div class="spinner-border text-secondary" role="status" style="width: 3rem; height: 3rem">
          <span class="visually-hidden">Loading...</span>
        </div>
      </div>
      <div v-if="error" class="error">{{ error }}</div>
      <div class="recipe" v-if="recipe">
        <h2 class="font-cursive display-4 display-md-3 text-center mt-4">{{ recipe.title }}</h2>
        <h4 class="font-cursive text-center mb-4">By {{ recipe.sourceName }}</h4>
        <div class="row mb-4 align-items-center recipe-card rounded">
          <div
            class="col-12 col-md-4 col-lg-3 d-flex flex-column justify-content-center align-items-center align-items-md-end text-end px-3"
          >
            <p>
              Vegetarian <span>{{ recipe.vegetarian ? '✅' : '❌' }}</span>
            </p>
            <p>
              Vegan <span>{{ recipe.vegan ? '✅' : '❌' }}</span>
            </p>
            <p>
              Ketogenic <span>{{ recipe.ketogenic ? '✅' : '❌' }}</span>
            </p>
            <p>
              Whole 30 <span>{{ recipe.whole30 ? '✅' : '❌' }}</span>
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
          </div>
          <div class="col-12 col-md-4 col-lg-6 text-center">
            <img :src="recipe.image" :alt="recipe.title" class="img-fluid rounded shadow mx-auto" />
          </div>
          <div
            class="col-12 col-md-4 col-lg-3 d-flex flex-column justify-content-center align-items-md-start px-3"
          >
            <p v-if="recipe.servings">🍽️ {{ recipe.servings }} servings</p>
            <p v-if="recipe.readyInMinutes">⏱️ Ready in {{ recipe.readyInMinutes }} minutes</p>
            <p v-if="recipe.cookingMinutes">🔥 Cooked in {{ recipe.cookingMinutes }} minutes</p>
            <p v-if="recipe.preparationMinutes">
              🥣 Prepared in {{ recipe.preparationMinutes }} minutes
            </p>
            <p v-if="recipe.pricePerServing">💰 ${{ recipe.pricePerServing }} per serving</p>
          </div>
        </div>

        <hr />

        <div class="row mt-4">
          <div class="col-12 col-md-4">
            <h2 class="font-cursive">Ingredients</h2>
            <ul class="ps-3 bullets">
              <li v-for="ingredient in ingredients" :key="ingredient.id">
                {{ ingredient.measures.us.amount }} {{ ingredient.measures.us.unitShort }}
                <span
                  v-if="ingredient.measures.metric.unitShort != ingredient.measures.us.unitShort"
                >
                  ({{ ingredient.measures.metric.amount
                  }}{{ ingredient.measures.metric.unitShort }})
                </span>

                {{ ingredient.name }}
              </li>
            </ul>
          </div>

          <div class="col-12 col-md-8 border-start border-secondary-subtle">
            <h2 class="font-cursive">Instructions</h2>
            <p v-if="recipe.instructions" v-html="recipe.instructions" class="text-break"></p>
            <p v-else>No Instructions found.</p>
          </div>
        </div>
      </div>

      <div v-else-if="error">
        <p class="error">{{ error }}</p>
      </div>
      <hr />

      <div class="d-flex justify-content-end mt-4">
        <RouterLink to="/" class="return-btn">
          <button class="btn btn-outline-secondary">Go Home</button>
        </RouterLink>
      </div>
    </div>
  </div>
</template>
