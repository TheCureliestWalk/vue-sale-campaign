<template>
  <main class="container">
    <h1>Sale Campaign</h1>
    <!-- Product List  -->
    <div>
      <h3>Product:</h3>
      <table>
        <thead>
          <tr>
            <th>Item</th>
            <th>Price</th>
            <th>Category</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in items" :key="item.id">
            <td>{{ item.name }}</td>
            <td>{{ item.price }}</td>
            <td>{{ item.category }}</td>
            <td><button @click="addItem(item)">Add item</button></td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="row">
      <!-- Selected Items -->
      <div class="column">
        <h3>Shopping Cart</h3>
        <p v-if="selectedItems.length === 0">~ There's no items in cart ~</p>
        <ul v-else>
          <li v-for="item in selectedItems" :key="item.id">
            {{ item.name }} - {{ item.price }} THB
          </li>
        </ul>
      </div>

      <!-- Discount Rule -->
      <div class="column">
        <h3>Discount</h3>

        <!-- 1. Coupon -->
        <div>
          <h4>1. Coupon</h4>
          <div class="flex items-center pl">
            <label for="fixed">
              <input
                type="radio"
                id="fixed"
                name="fixed"
                value="fixed"
                v-model="couponDiscountType"
              />
              <span>Fixed Rate</span></label
            >

            <label for="percentage">
              <input
                type="radio"
                id="percentage"
                name="percentage"
                value="percentage"
                v-model="couponDiscountType"
              />
              <span>Percentage Discount</span></label
            >
          </div>
          <input type="number" v-model="couponDiscount" />
        </div>

        <!-- 2. On Top -->
        <div>
          <h4>2. On Top</h4>
          <p>15% off on Clothing || Or points (1 pts = 1 THB, max 20%)</p>
          <p>TODO</p>
        </div>

        <!-- 3. Special Campaign (Seasonal) -->
        <div>
          <h4>3. Special Campaign (Seasonal)</h4>
          <small>Example Discount: 40 THB at every 300 THB</small>
          <p>
            Every <input type="number" v-model="seasonalDiscountEveryPrice" /> THB, get
            <input type="number" v-model="seasonalDiscountPrice" /> THB discount
          </p>

          <p>
            Seasonal discounted <b>{{ seasonalDiscount }} THB</b>
          </p>
        </div>
      </div>

      <!-- Final Price -->
      <div class="column">
        <h4>
          Total Price: <b>{{ totalPrice }} THB</b>
        </h4>
        <h4>
          Final Price:
          <b>{{ finalPrice }} THB</b>
        </h4>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { items } from '@/data/items.json'

const selectedItems = ref([])
const couponDiscountType = ref('fixed')
const totalPrice = ref(0)

const couponDiscount = ref(0)
const onTopDiscount = ref(0)

const seasonalDiscount = ref(0)
const seasonalDiscountEveryPrice = ref(300)
const seasonalDiscountPrice = ref(40)

// Calculation of discount price

// First Priority
const calculateCouponDiscount = (type, couponDiscount) => {
  if (type == 'percentage') {
    return (finalPrice.value = totalPrice.value - totalPrice.value * (couponDiscount / 100))
  } else {
    return (finalPrice.value = totalPrice.value - couponDiscount)
  }
}

// Third Priority
const calculateSeasonalDiscount = (seasonalDiscountPrice, seasonalDiscountEveryPrice) => {
  const discount = computed(
    () => Math.floor(finalPrice.value / seasonalDiscountEveryPrice) * seasonalDiscountPrice
  ) // Discount 40 THB at every 300 THB
  return (seasonalDiscount.value = discount.value)
}

// Final Price Calc.
const finalPrice = computed(() => {
  return (
    selectedItems.value.reduce((acc, item) => acc + item.price, 0) -
    couponDiscount.value -
    onTopDiscount.value -
    seasonalDiscount.value
  )
})

watch(() => {
  calculateCouponDiscount(couponDiscountType.value, couponDiscount.value)
  calculateSeasonalDiscount(seasonalDiscountPrice.value, seasonalDiscountEveryPrice.value)
})

const addItem = (item) => {
  selectedItems.value.push(item)
  totalPrice.value += item.price
  console.log('Added item', item)
}
</script>

<style>
.flex {
  display: flex;
}

.items-center {
  align-items: center;
}

.pl {
  padding-left: 1rem;
}
</style>
