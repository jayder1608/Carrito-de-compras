<template>
  <q-layout view="lHh Lpr lFf" class="bg-green-1">


    <q-header elevated class="bg-green text-white">
      <q-toolbar>
        <q-toolbar-title class="text-weight-bold">
          <q-icon name="shopping_cart" class="q-mr-sm" />
          Carrito de Compras
        </q-toolbar-title>
        <q-badge color="orange" rounded floating v-if="totalItems > 0">
          {{ totalItems }} items
        </q-badge>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-lg">
        <q-banner rounded class="bg-green-3 text-green-10 text-center q-pa-md q-mb-md">
          <div class="text-h6 text-weight-bold">Bienvenido a tu Carrito de Compras</div>
          <div class="text-caption">Gestiona tus productos de forma fácil y rápida</div>
          <template v-if="savedMessage">
            <q-icon name="check_circle" color="positive" class="q-ml-sm" />
            <span class="text-positive text-weight-medium">¡Carrito guardado!</span>
          </template>
        </q-banner>
        <div class="row q-col-gutter-lg">
          <div class="col-12 col-md-6">
            <q-card class="q-pa-md bg-white text-grey-9 full-height">
              <q-card-section class="text-center">
                <div class="text-h5 text-weight-bold text-green flex items-center justify-center">
                  <q-icon name="inventory_2" class="q-mr-sm" />
                  Productos Disponibles
                </div>
              </q-card-section>
              
              <q-separator spaced />
              
              <q-card-section class="q-gutter-md">
                <div v-for="product in products" :key="product.id" 
                  class="row items-center justify-between q-pa-md rounded-borders"
                  :class="{'bg-green-2': cart.find(item => item.id === product.id), 
                          'bg-grey-1': !cart.find(item => item.id === product.id)}">
                  <div class="col row items-center">
                    <q-img 
                      :src="product.image" 
                      :alt="product.name"
                      style="width: 100px; height: 100px;"
                      class="rounded-borders q-mr-md"
                    />
                    <div>
                      <span class="text-weight-medium text-dark text-h6">{{ product.name }}</span>
                      <div class="text-h5 text-green text-weight-bold q-mt-sm">
                        {{ formatCurrency(product.price) }}
                      </div>
                    </div>
                  </div>
                  <div class="col-auto">
                    <q-btn 
                      color="green" 
                      icon="add" 
                      round 
                      dense 
                      size="lg"
                      @click="addToCart(product)"
                    />
                  </div>
                </div>
              </q-card-section>
            </q-card>
          </div>
          <div class="col-12 col-md-6">
            <q-card class="q-pa-md bg-white text-grey-9 full-height" bordered v-if="cart.length > 0">
              <q-card-section>
                <div class="text-h5 text-weight-bold text-green flex items-center justify-center">
                  <q-icon name="shopping_bag" class="q-mr-sm" />
                  Tu Carrito
                  <q-badge color="orange" class="q-ml-sm">
                    {{ totalItems }}
                  </q-badge>
                </div>
                <div class="text-caption text-grey-7 text-center">
                  Gestiona los productos en tu carrito
                </div>
              </q-card-section>
              <q-separator spaced />
              <q-card-section style="max-height: 500px; overflow-y: auto;" class="q-gutter-sm">
                <div v-for="item in cart" :key="item.id" 
                  class="row items-center justify-between q-pa-md rounded-borders bg-green-2 q-mb-sm">
                  <div class="col row items-center">
                    <q-img 
                      :src="item.image" 
                      :alt="item.name"
                      style="width: 80px; height: 80px;"
                      class="rounded-borders q-mr-md"
                    />
                    <div>
                      <span class="text-weight-medium text-dark text-h6">{{ item.name }}</span>
                      <div class="text-caption text-grey-7">
                        {{ formatCurrency(item.price) }} c/u
                      </div>
                      <div class="text-h6 text-green text-weight-bold q-mt-xs">
                        Total: {{ formatCurrency(item.price * item.qty) }}
                      </div>
                    </div>
                  </div>
                  <div class="col-auto row items-center q-gutter-sm">
                    <q-btn 
                      color="negative" 
                      icon="remove" 
                      round 
                      dense 
                      size="md"
                      @click="decrement(item.id)"
                    />
                    <span class="text-weight-bold q-px-md bg-white rounded-borders border text-h6" style="min-width: 40px; text-align: center;">
                      {{ item.qty }}
                    </span>
                    <q-btn 
                      color="positive" 
                      icon="add" 
                      round 
                      dense 
                      size="md"
                      @click="increment(item.id)"
                    />
                  </div>
                </div>
              </q-card-section>
              <q-separator spaced />
              <q-card-section class="bg-grey-1 rounded-borders q-pa-md">
                <div class="row justify-between text-dark q-pb-sm">
                  <span class="flex items-center">
                    <q-icon name="receipt" class="q-mr-sm" />
                    Subtotal ({{ totalItems }} items)
                  </span>
                  <span class="text-weight-medium text-h6">{{ formatCurrency(subtotal) }}</span>
                </div>
                <div class="row justify-between text-grey-8 q-pb-sm">
                  <span class="flex items-center">
                    <q-icon name="account_balance" class="q-mr-sm" />
                    Impuesto (16%)
                  </span>
                  <span class="text-h6">{{ formatCurrency(impuesto) }}</span>
                </div>
                <q-separator spaced />
                <div class="row justify-between text-h4 text-green text-weight-bold q-pt-sm">
                  <span class="flex items-center">
                    <q-icon name="payments" class="q-mr-sm" />
                    Total a Pagar
                  </span>
                  <span>{{ formatCurrency(totalFinal) }}</span>
                </div>
                <q-banner v-if="totalFinal > 1000" class="bg-orange text-white q-mt-md rounded-borders">
                  <div class="flex items-center">
                    <q-icon name="local_shipping" class="q-mr-sm" />
                    <div>
                      <div class="text-weight-bold">¡Felicidades!</div>
                      <div>Tu compra supera los $1000 y califica para ENVÍO GRATIS</div>
                    </div>
                  </div>
                </q-banner>
              </q-card-section>

              <q-card-actions align="center" class="q-pt-md">
                <q-btn 
                  color="green" 
                  label="Pagar Ahora" 
                  glossy 
                  unelevated 
                  icon="credit_card"
                  size="lg"
                  class="full-width q-mb-sm"
                  :disable="cart.length === 0"
                />
                <q-btn 
                  color="grey-7" 
                  label="Limpiar Carrito" 
                  flat 
                  icon="delete"
                  class="full-width"
                  @click="cart = []"
                />
              </q-card-actions>
            </q-card>
            <q-card v-else class="q-pa-xl text-center bg-white full-height flex flex-center">
              <div>
                <q-icon name="shopping_cart" size="xl" color="grey-5" class="q-mb-md" />
                <div class="text-h5 text-grey-7 q-mb-sm text-weight-medium">Tu carrito está vacío</div>
                <div class="text-caption text-grey-6">Agrega productos desde la columna izquierda</div>
              </div>
            </q-card>
          </div>
        </div>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import {
  QLayout,
  QHeader,
  QToolbar,
  QToolbarTitle,
  QPageContainer,
  QPage,
  QBanner,
  QCard,
  QCardSection,
  QCardActions,
  QBtn,
  QIcon,
  QSeparator,
  QBadge,
  QImg
} from 'quasar'

import { useQuasar } from 'quasar'
import { ref, computed, watch, onMounted } from 'vue'

const $q = useQuasar()

const products = ref([
  { 
    id: 1, 
    name: 'Laptop Dell XPS', 
    price: 899,
    image: 'https://m.media-amazon.com/images/I/81Vep45DQ4L._AC_SL1500_.jpg'
  },
  { 
    id: 2, 
    name: 'Mouse Logitech MX', 
    price: 25,
    image: 'https://media.falabella.com/falabellaCO/43700767_1/w=800,h=800,fit=pad'
  },
  { 
    id: 3, 
    name: 'Teclado Mecánico RGB', 
    price: 89,
    image: 'https://i.blogs.es/ce6ee4/katboard3/1200_900.jpeg'
  },
  { 
    id: 4, 
    name: 'Monitor 24" 4K', 
    price: 199,
    image: 'https://www.lg.com/content/dam/channel/wcms/co/images/monitores/24mp400-b_awp_escb_co_c/gallery/DZ-2.jpg'
  },
  { 
    id: 5, 
    name: 'Webcam HD Profesional', 
    price: 59,
    image: 'https://http2.mlstatic.com/D_NQ_NP_2X_865030-MCO82363355023_022025-T.webp'
  }
])

const cart = ref([])
const savedMessage = ref(false)

onMounted(() => {
  const saved = localStorage.getItem('carrito_quasar')
  if (saved) {
    try {
      cart.value = JSON.parse(saved)
    } catch (e) {
      console.error('Error cargando carrito:', e)
      cart.value = []
    }
  }
})

const totalItems = computed(() => 
  cart.value.reduce((sum, item) => sum + item.qty, 0)
)

const subtotal = computed(() => 
  cart.value.reduce((sum, item) => sum + (item.price * item.qty), 0)
)

const impuesto = computed(() => 
  +(subtotal.value * 0.16).toFixed(2)
)

const totalFinal = computed(() => 
  +(subtotal.value + impuesto.value).toFixed(2)
)

watch(cart, (newCart) => {
  localStorage.setItem('carrito_quasar', JSON.stringify(newCart))
  savedMessage.value = true
  setTimeout(() => (savedMessage.value = false), 1500)
}, { deep: true })

watch(totalFinal, (newTotal, oldTotal) => {
  if (newTotal > 1000 && oldTotal <= 1000) {
    $q.notify({
      color: 'orange',
      message: '¡Tu total superó $1000 — Podrías calificar para envío gratis! 🎉',
      icon: 'local_shipping',
      position: 'top',
      timeout: 3000
    })
  }
})

function addToCart(product) {
  const existingItem = cart.value.find(item => item.id === product.id)
  
  if (existingItem) {
    existingItem.qty++
  } else {
    cart.value.push({ 
      id: product.id, 
      name: product.name, 
      price: product.price, 
      image: product.image,
      qty: 1 
    })
  }
}

function increment(id) {
  const item = cart.value.find(item => item.id === id)
  if (item) item.qty++
}

function decrement(id) {
  const itemIndex = cart.value.findIndex(item => item.id === id)
  if (itemIndex > -1) {
    cart.value[itemIndex].qty--
    if (cart.value[itemIndex].qty <= 0) {
      cart.value.splice(itemIndex, 1)
    }
  }
}

function formatCurrency(value) {
  return value.toLocaleString('en-US', { 
    style: 'currency', 
    currency: 'USD',
    minimumFractionDigits: 2
  })
}
</script>

<style scoped>
.full-height {
  height: 100%;
}

.row > .col {
  display: flex;
}
</style>