<template>
  <v-container>
    <!-- Filtres et recherche -->
    <v-row class="mb-4">
      <v-col cols="12" lg="6">
        <v-text-field
          v-model="search"
          label="Rechercher une carte"
          prepend-inner-icon="mdi-magnify"
          variant="outlined"
          density="compact"
          clearable
          hide-details
        ></v-text-field>
      </v-col>
      <v-col cols="12" sm="6" lg="3">
        <v-select
          v-model="selectedFactions"
          :items="factions"
          item-title="name"
          item-value="value"
          label="Factions"
          variant="outlined"
          density="compact"
          multiple
          clearable
          hide-details
        >
          <template v-slot:selection="{ item }">
            <v-img :src="item.raw.image" height="24" width="24" contain></v-img>
          </template>
          <template v-slot:item="{ item, props }">
            <v-list-item v-bind="props">
              <template v-slot:prepend>
                <v-img :src="item.raw.image" height="24" width="24" contain class="mr-2"></v-img>
              </template>
            </v-list-item>
          </template>
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" lg="3">
        <v-select
          v-model="sortBy"
          :items="sortOptions"
          label="Trier par"
          variant="outlined"
          density="compact"
          hide-details
        ></v-select>
      </v-col>
    </v-row>

    <v-data-iterator :items="filteredCards" :items-per-page="12" :search="search">
      <template v-slot:default="{ items }">
        <v-container class="pa-2" fluid>
          <v-row dense>
            <v-col v-for="item in items" :key="item.raw.id" cols="12" sm="6" md="4" lg="2">
              <v-card
                :href="`https://www.altered.gg/fr-fr/cards/${item.raw.id}/offers`"
                target="_blank"
                hover
                class="pb-3"
                border
                flat
              >
                <v-img :src="item.raw.pictureUrl" class="flex-grow-1">
                  <template v-slot:placeholder>
                    <div class="d-flex align-center justify-center fill-height">
                      <v-progress-circular indeterminate color="primary"></v-progress-circular>
                    </div>
                  </template>
                </v-img>

                <v-card-text class="text-center pa-2">
                  <div class="text-h6 text-green">{{ item.raw.price }}€</div>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </template>

      <template v-slot:footer="{ page, pageCount, prevPage, nextPage }">
        <v-toolbar density="compact" color="transparent" flat>
          <v-spacer></v-spacer>
          <v-btn
            :disabled="page === 1"
            density="comfortable"
            icon="mdi-arrow-left"
            variant="tonal"
            rounded
            @click="prevPage"
          ></v-btn>

          <div class="mx-2 text-body-1">Page {{ page }} / {{ pageCount }}</div>

          <v-btn
            :disabled="page >= pageCount"
            density="comfortable"
            icon="mdi-arrow-right"
            variant="tonal"
            rounded
            @click="nextPage"
          ></v-btn>
          <v-spacer></v-spacer>
        </v-toolbar>
      </template>
    </v-data-iterator>
  </v-container>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useDisplay } from 'vuetify'

interface UniqueCard {
  id: string
  name: string
  pictureUrl: string
  price: number
  faction: string
  vendu: boolean
}

const page = ref(1)
const search = ref('')
const selectedFactions = ref<string[]>(['axiom', 'bravos', 'lyra', 'muna', 'ordis', 'yzmir'])
const uniqueCards = ref<UniqueCard[]>([])
const sortBy = ref('price-desc')

const sortOptions = [
  { value: 'price-asc', title: 'Prix croissant' },
  { value: 'price-desc', title: 'Prix décroissant' },
  { value: 'name-asc', title: 'Nom A-Z' },
  { value: 'name-desc', title: 'Nom Z-A' },
]
const { mdAndUp, lgAndUp, smAndUp } = useDisplay()

// Calculer le nombre d'items par page en fonction du breakpoint
// sm=6 (2 cols) -> 2*2=4, md=4 (3 cols) -> 3*2=6, lg=3 (4 cols) -> 4*2=8
const itemsPerPageComputed = computed(() => {
  if (lgAndUp.value) return 8 // 4 colonnes * 2 lignes
  if (mdAndUp.value) return 6 // 3 colonnes * 2 lignes
  if (smAndUp.value) return 4 // 2 colonnes * 2 lignes
  return 2 // 1 colonne * 2 lignes (xs)
})

const loadCardsFromCSV = async () => {
  try {
    const response = await fetch('/uniques.csv')
    const text = await response.text()
    const lines = text.split('\n').slice(1) // Skip header
    uniqueCards.value = lines
      .filter((line) => line.trim())
      .map((line) => {
        const [id, name, price, pictureUrl, faction, vendu] = line.split(';')
        return {
          id: id?.trim() || '',
          name: name?.trim() || '',
          price: parseInt(price?.trim() || '0'),
          pictureUrl: pictureUrl?.trim() || '',
          faction: faction?.trim() || '',
          vendu: vendu?.trim() === 'true',
        }
      })
  } catch (error) {
    console.error('Error loading CSV:', error)
  }
}

onMounted(() => {
  loadCardsFromCSV()
})

const factions = [
  {
    value: 'axiom',
    name: 'Axiom',
    image: new URL('@/assets/AXIOM-faction-icon.png', import.meta.url).href,
  },
  {
    value: 'bravos',
    name: 'Bravos',
    image: new URL('@/assets/BRAVOS-faction-icon.png', import.meta.url).href,
  },
  {
    value: 'lyra',
    name: 'Lyra',
    image: new URL('@/assets/LYRA-faction-icon.png', import.meta.url).href,
  },
  {
    value: 'muna',
    name: 'Muna',
    image: new URL('@/assets/MUNA-faction-icon.png', import.meta.url).href,
  },
  {
    value: 'ordis',
    name: 'Ordis',
    image: new URL('@/assets/ORDIS-faction-icon.png', import.meta.url).href,
  },
  {
    value: 'yzmir',
    name: 'Yzmir',
    image: new URL('@/assets/YZMIR-faction-icon.png', import.meta.url).href,
  },
]

const filteredCards = computed(() => {
  let cards = uniqueCards.value

  // Filtrer par faction
  if (selectedFactions.value.length > 0) {
    cards = cards.filter((card) => selectedFactions.value.includes(card.faction))
  }

  // Trier
  const sorted = [...cards]
  switch (sortBy.value) {
    case 'price-asc':
      sorted.sort((a, b) => a.price - b.price)
      break
    case 'price-desc':
      sorted.sort((a, b) => b.price - a.price)
      break
    case 'name-asc':
      sorted.sort((a, b) => a.name.localeCompare(b.name))
      break
    case 'name-desc':
      sorted.sort((a, b) => b.name.localeCompare(a.name))
      break
  }

  return sorted
})
</script>

<style scoped>
.v-card:hover {
  transform: translateY(-4px);
  transition: transform 0.2s;
}
</style>
