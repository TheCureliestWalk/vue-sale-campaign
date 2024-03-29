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
        <label>
          <input type="checkbox" v-model="isCouponEnable" />
          <span>Coupon</span>
        </label>
        <div v-if="isCouponEnable">
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
          <input type="number" v-model="couponDiscountAmount" />
          <code
            >Discounted: <b>{{ couponDiscount }} THB</b></code
          >
        </div>

        <!-- 2. On Top -->
        <label>
          <input type="checkbox" v-model="isOnTopEnable" />
          <span>On Top</span>
        </label>
        <div v-if="isOnTopEnable">
          <h4>2. On Top</h4>
          <!-- On Top Type Selection -->
          <div class="flex items-center pl">
            <label for="onTopByCategoryPercentage">
              <input
                type="radio"
                id="onTopByCategoryPercentage"
                name="onTopByCategoryPercentage"
                value="categoryPercentage"
                v-model="onTopDiscountType"
              />
              <span>By Category Percentage</span>
            </label>
            <label for="onTopByPoint">
              <input
                type="radio"
                id="onTopByPoint"
                name="onTopByPoint"
                value="point"
                v-model="onTopDiscountType"
              />
              <span>By Point</span>
            </label>
          </div>

          <!-- On Top Category Selection -->
          <div v-if="onTopDiscountType === 'categoryPercentage'">
            <label for="onTopCategorySelection">Category: </label>
            <select id="categorySelection" name="categorySelection" v-model="onTopDiscountCategory">
              <option value="clothing">Clothing</option>
              <option value="electronics">Electronics</option>
              <option value="accessories">Accessories</option>
            </select>
          </div>
          <div>
            <label for="onTopDiscountAmount">{{
              onTopDiscountType === 'categoryPercentage'
                ? 'Percentage: '
                : 'Point: (max 20% of Total Price)'
            }}</label>
            <input
              id="onTopDiscountAmount"
              name="onTopDiscountAmount"
              type="number"
              v-model="onTopDiscountAmount"
            />
          </div>
          <code v-if="onTopDiscountType === 'categoryPercentage'"
            >onTopDiscountCategory: {{ onTopDiscountCategory }}</code
          >
          <code>onTopDiscount: {{ onTopDiscount }}</code>
        </div>

        <!-- 3. Special Campaign (Seasonal) -->
        <label>
          <input type="checkbox" v-model="isSeasonalEnable" />
          <span>Special Campaign (Seasonal)</span>
        </label>
        <div v-if="isSeasonalEnable">
          <small
            >Example Discount: {{ seasonalDiscountPrice }} THB at every
            {{ seasonalDiscountEveryPrice }} THB</small
          >
          <div>
            <!-- I don't know how to call variable for this speicial deal -->
            <label for="seasonalDiscountEveryPrice">Every (THB)</label>
            <input
              type="number"
              id="seasonalDiscountEveryPrice"
              name="seasonalDiscountEveryPrice"
              v-model="seasonalDiscountEveryPrice"
            />
            <label for="seasonalDiscountEveryPrice">Will Get (THB) off</label>
            <input
              type="number"
              id="seasonalDiscountPrice"
              name="seasonalDiscountPrice"
              v-model="seasonalDiscountPrice"
            />
          </div>

          <code>
            Seasonal discounted: <b>{{ seasonalDiscount }} THB</b>
          </code>
        </div>
      </div>

      <!-- Final Price -->
      <div class="column">
        <h4>
          Total Price: <b>{{ totalPrice }} THB</b>
        </h4>
        <h4 style="color: green">
          Final Price:
          <b>{{ finalPrice }} THB</b>
        </h4>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, watchEffect } from 'vue'
import { items } from '@/data/items.json'

const selectedItems = ref([])

// Discount type enable handling

const isCouponEnable = ref(false)
const isOnTopEnable = ref(false)
const isSeasonalEnable = ref(false)

const couponDiscount = ref(0)

// Form handling variables
const couponDiscountType = ref('fixed')
const couponDiscountAmount = ref(0)

const onTopDiscount = ref(0)
const onTopDiscountType = ref('categoryPercentage') // 'point' or 'categoryPercentage'
const onTopDiscountCategory = ref('clothing') // 'clothing' or 'electronics' or 'accessories'
const onTopDiscountAmount = ref(0)

const seasonalDiscount = ref(0)

// Form handling variables
const seasonalDiscountEveryPrice = ref(100000)
const seasonalDiscountPrice = ref(40)

const totalPrice = ref(0)

// Calculation of discount price logic

// First Priority
const calculateCouponDiscount = (type, discount) => {
  // Check if input is a negative value or not a number
  if (discount < 0) {
    couponDiscountAmount.value = 0
  }
  // Percentage Discount
  if (type == 'percentage') {
    return (couponDiscount.value = totalPrice.value * (discount / 100))
  }
  // Fixed Rate discount
  return (couponDiscount.value = discount)
}

// Second Priority
const calculateOnTopDiscount = (type, discount) => {
  // Check if input is a negative value
  if (discount <= 0) {
    discount = 0
  }
  // Category Percentage Discount
  if (type === 'categoryPercentage') {
    // Filter only selected category items (make it lowercase for comparison)
    const _categoryItems = selectedItems.value.filter(
      (item) => item.category.toLowerCase() === onTopDiscountCategory.value
    )

    // Accumulate total price of selected category items
    const _categoryTotalPrice = _categoryItems.reduce((acc, item) => acc + item.price, 0)
    return (onTopDiscount.value = _categoryTotalPrice * (discount / 100))
  }
  // ELSE: Point Discount (max 20% of Total Price)
  return (onTopDiscount.value =
    discount > totalPrice.value * 0.2 ? totalPrice.value * 0.2 : discount)
}

// Third Priority
const calculateSeasonalDiscount = (discount, discountEvery) => {
  // Check if input is a negative value
  if (discount <= 0 || typeof discount !== 'number') {
    seasonalDiscountPrice.value = 0
  }
  if (discountEvery <= 0 || typeof discountEvery !== 'number') {
    seasonalDiscountEveryPrice.value = 1
  }
  // Check to prevent the divider is 0, otherwise will get NaN
  if (
    typeof seasonalDiscountPrice.value !== 'number' ||
    typeof seasonalDiscountEveryPrice.value !== 'number'
  ) {
    seasonalDiscountEveryPrice.value = 0
    seasonalDiscountPrice.value = 0
  }
  const _discount = computed(() => Math.floor(totalPrice.value / discountEvery) * discount) // Discount 40 THB at every 300 THB
  return (seasonalDiscount.value = _discount.value)
}

// Final Price Calc.
const finalPrice = computed(() => {
  const _finalPrice =
    totalPrice.value - couponDiscount.value - onTopDiscount.value - seasonalDiscount.value
  // Check if value is negative number and return 0 THB
  if (_finalPrice <= 0) return 0
  return _finalPrice
})

// Watch and update every changes
watchEffect(() => {
  calculateCouponDiscount(couponDiscountType.value, couponDiscountAmount.value)
  calculateOnTopDiscount(onTopDiscountType.value, onTopDiscountAmount.value)
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
  gap: 1rem;
}

.items-center {
  align-items: center;
}

.pl {
  padding-left: 1rem;
}

label > span {
  padding-left: 0.5rem;
}
</style>
