<script setup>
import { ref, computed } from 'vue'
import PlantCard from './components/PlantCard.vue'
import { plants } from './data/plants.js'

const cart = ref([])
const showCart = ref(false)
const showSignIn = ref(false)
const showSignUp = ref(false)
const isSignedIn = ref(false)
const user = ref(null)
const signInForm = ref({
  email: '',
  password: ''
})
const showCheckout = ref(false)
const checkoutStep = ref(1) // 1: shipping, 2: payment, 3: review, 4: confirmation
const shippingForm = ref({
  firstName: '',
  lastName: '',
  email: '',
  address: '',
  city: '',
  state: '',
  zipCode: '',
  country: 'US'
})
const paymentForm = ref({
  cardNumber: '',
  expiryDate: '',
  cvv: '',
  nameOnCard: ''
})
const orderNumber = ref('')

const addToCart = (plant) => {
  const existingItem = cart.value.find(item => item.id === plant.id)
  if (existingItem) {
    existingItem.quantity += 1
  } else {
    cart.value.push({ ...plant, quantity: 1 })
  }
  showCart.value = true
}

const removeFromCart = (plantId) => {
  cart.value = cart.value.filter(item => item.id !== plantId)
}

const updateQuantity = (plantId, newQuantity) => {
  if (newQuantity <= 0) {
    removeFromCart(plantId)
  } else {
    const item = cart.value.find(item => item.id === plantId)
    if (item) {
      item.quantity = newQuantity
    }
  }
}

const getTotalPrice = () => {
  return cart.value.reduce((total, item) => total + (item.price * item.quantity), 0).toFixed(2)
}

const getTotalItems = () => {
  return cart.value.reduce((total, item) => total + item.quantity, 0)
}

const signIn = (email, password) => {
  // Simple mock sign-in - in a real app, this would call an API
  if (email && password) {
    isSignedIn.value = true
    user.value = { email, name: email.split('@')[0] }
    showSignIn.value = false
  }
}

const handleSignIn = () => {
  signIn(signInForm.value.email, signInForm.value.password)
  signInForm.value = { email: '', password: '' } // Reset form
}

const startCheckout = () => {
  if (cart.value.length === 0) return
  showCart.value = false
  showCheckout.value = true
  checkoutStep.value = 1
}

const nextStep = () => {
  if (checkoutStep.value < 3) {
    checkoutStep.value++
  }
}

const prevStep = () => {
  if (checkoutStep.value > 1) {
    checkoutStep.value--
  }
}

const placeOrder = () => {
  // Mock order placement
  orderNumber.value = 'ORD-' + Date.now().toString().slice(-8)
  checkoutStep.value = 4
  // Clear cart after successful order
  setTimeout(() => {
    cart.value = []
  }, 2000)
}

const closeCheckout = () => {
  showCheckout.value = false
  checkoutStep.value = 1
  shippingForm.value = {
    firstName: '',
    lastName: '',
    email: '',
    address: '',
    city: '',
    state: '',
    zipCode: '',
    country: 'US'
  }
  paymentForm.value = {
    cardNumber: '',
    expiryDate: '',
    cvv: '',
    nameOnCard: ''
  }
}

const isStepValid = computed(() => {
  if (checkoutStep.value === 1) {
    return shippingForm.value.firstName && shippingForm.value.lastName &&
           shippingForm.value.email && shippingForm.value.address &&
           shippingForm.value.city && shippingForm.value.state && shippingForm.value.zipCode
  }
  if (checkoutStep.value === 2) {
    return paymentForm.value.nameOnCard && paymentForm.value.cardNumber &&
           paymentForm.value.expiryDate && paymentForm.value.cvv
  }
  return true
})

const signOut = () => {
  isSignedIn.value = false
  user.value = null
  cart.value = [] // Clear cart on sign out
}
</script>

<template>
  <div class="app-container">
    <header class="hero">
      <div class="hero-content">
        <h1>🌿 Rare Indoor Plants 🌿</h1>
        <p>Discover the most beautiful and rare indoor plants for your home</p>
        <div class="header-actions">
          <button v-if="!isSignedIn" class="signin-btn" @click="showSignIn = true">
            👤 Sign In
          </button>
          <button v-else class="user-btn" @click="signOut">
            👋 {{ user.name }}
          </button>
          <button class="cart-btn" @click="showCart = !showCart">
            🛒 Cart ({{ getTotalItems() }})
          </button>
        </div>
      </div>
    </header>

    <main class="store-container">
      <div class="plants-grid">
        <PlantCard
          v-for="plant in plants"
          :key="plant.id"
          :plant="plant"
          @add-to-cart="addToCart"
        />
      </div>
    </main>

    <!-- Cart Sidebar -->
    <div v-if="showCart" class="cart-overlay" @click="showCart = false">
      <div class="cart-sidebar" @click.stop>
        <div class="cart-header">
          <h2>Your Cart</h2>
          <button class="close-btn" @click="showCart = false">✕</button>
        </div>

        <div v-if="cart.length === 0" class="empty-cart">
          <p>Your cart is empty</p>
          <span class="empty-cart-icon">🛒</span>
        </div>

        <div v-else class="cart-items">
          <div
            v-for="item in cart"
            :key="item.id"
            class="cart-item"
          >
            <img :src="item.image" :alt="item.name" class="cart-item-image" />
            <div class="cart-item-info">
              <h3>{{ item.name }}</h3>
              <p class="cart-item-price">${{ item.price }}</p>
              <div class="quantity-controls">
                <button @click="updateQuantity(item.id, item.quantity - 1)">-</button>
                <span>{{ item.quantity }}</span>
                <button @click="updateQuantity(item.id, item.quantity + 1)">+</button>
              </div>
            </div>
            <button class="remove-btn" @click="removeFromCart(item.id)">🗑️</button>
          </div>

          <div class="cart-total">
            <strong>Total: ${{ getTotalPrice() }}</strong>
            <button class="checkout-btn" @click="startCheckout">
              Proceed to Checkout
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Sign In Modal -->
    <div v-if="showSignIn" class="modal-overlay" @click="showSignIn = false">
      <div class="signin-modal" @click.stop>
        <div class="modal-header">
          <h2>Welcome Back</h2>
          <button class="close-btn" @click="showSignIn = false">✕</button>
        </div>

        <form class="signin-form" @submit.prevent="handleSignIn">
          <div class="form-group">
            <label for="email">Email</label>
            <input
              id="email"
              v-model="signInForm.email"
              type="email"
              required
              placeholder="your@email.com"
            />
          </div>

          <div class="form-group">
            <label for="password">Password</label>
            <input
              id="password"
              v-model="signInForm.password"
              type="password"
              required
              placeholder="Your password"
            />
          </div>

          <button type="submit" class="signin-submit-btn">
            Sign In
          </button>
        </form>

        <div class="modal-footer">
          <p>Don't have an account? <button class="link-btn" @click="showSignUp = true; showSignIn = false">Sign Up</button></p>
        </div>
      </div>
    </div>

    <!-- Checkout Modal -->
    <div v-if="showCheckout" class="modal-overlay" @click="closeCheckout">
      <div class="checkout-modal" @click.stop>
        <div class="checkout-header">
          <div class="checkout-steps">
            <div class="step" :class="{ active: checkoutStep === 1, completed: checkoutStep > 1 }">
              <span class="step-number">1</span>
              <span class="step-label">Shipping</span>
            </div>
            <div class="step" :class="{ active: checkoutStep === 2, completed: checkoutStep > 2 }">
              <span class="step-number">2</span>
              <span class="step-label">Payment</span>
            </div>
            <div class="step" :class="{ active: checkoutStep === 3, completed: checkoutStep > 3 }">
              <span class="step-number">3</span>
              <span class="step-label">Review</span>
            </div>
          </div>
          <button class="close-btn" @click="closeCheckout">✕</button>
        </div>

        <div class="checkout-content">
          <!-- Step 1: Shipping -->
          <div v-if="checkoutStep === 1" class="checkout-step">
            <h2>Shipping Information</h2>
            <form class="checkout-form">
              <div class="form-row">
                <div class="form-group">
                  <label>First Name</label>
                  <input v-model="shippingForm.firstName" type="text" required />
                </div>
                <div class="form-group">
                  <label>Last Name</label>
                  <input v-model="shippingForm.lastName" type="text" required />
                </div>
              </div>
              <div class="form-group">
                <label>Email</label>
                <input v-model="shippingForm.email" type="email" required />
              </div>
              <div class="form-group">
                <label>Address</label>
                <input v-model="shippingForm.address" type="text" required />
              </div>
              <div class="form-row">
                <div class="form-group">
                  <label>City</label>
                  <input v-model="shippingForm.city" type="text" required />
                </div>
                <div class="form-group">
                  <label>State</label>
                  <input v-model="shippingForm.state" type="text" required />
                </div>
                <div class="form-group">
                  <label>ZIP Code</label>
                  <input v-model="shippingForm.zipCode" type="text" required />
                </div>
              </div>
            </form>
          </div>

          <!-- Step 2: Payment -->
          <div v-if="checkoutStep === 2" class="checkout-step">
            <h2>Payment Information</h2>
            <form class="checkout-form">
              <div class="form-group">
                <label>Name on Card</label>
                <input v-model="paymentForm.nameOnCard" type="text" required />
              </div>
              <div class="form-group">
                <label>Card Number</label>
                <input v-model="paymentForm.cardNumber" type="text" placeholder="1234 5678 9012 3456" required />
              </div>
              <div class="form-row">
                <div class="form-group">
                  <label>Expiry Date</label>
                  <input v-model="paymentForm.expiryDate" type="text" placeholder="MM/YY" required />
                </div>
                <div class="form-group">
                  <label>CVV</label>
                  <input v-model="paymentForm.cvv" type="text" placeholder="123" required />
                </div>
              </div>
              <div class="security-notice">
                <span class="lock-icon">🔒</span>
                Your payment information is secure and encrypted
              </div>
            </form>
          </div>

          <!-- Step 3: Review Order -->
          <div v-if="checkoutStep === 3" class="checkout-step">
            <h2>Review Your Order</h2>
            <div class="order-review">
              <div class="review-section">
                <h3>Shipping Address</h3>
                <p>{{ shippingForm.firstName }} {{ shippingForm.lastName }}</p>
                <p>{{ shippingForm.address }}</p>
                <p>{{ shippingForm.city }}, {{ shippingForm.state }} {{ shippingForm.zipCode }}</p>
                <p>{{ shippingForm.email }}</p>
              </div>

              <div class="review-section">
                <h3>Payment Method</h3>
                <p>•••• •••• •••• {{ paymentForm.cardNumber.slice(-4) }}</p>
                <p>{{ paymentForm.nameOnCard }}</p>
              </div>

              <div class="review-section">
                <h3>Order Items</h3>
                <div v-for="item in cart" :key="item.id" class="review-item">
                  <span>{{ item.name }} (x{{ item.quantity }})</span>
                  <span>${{ (item.price * item.quantity).toFixed(2) }}</span>
                </div>
                <div class="review-total">
                  <strong>Total: ${{ getTotalPrice() }}</strong>
                </div>
              </div>
            </div>
          </div>

          <!-- Step 4: Order Confirmation -->
          <div v-if="checkoutStep === 4" class="checkout-step confirmation">
            <div class="confirmation-content">
              <div class="success-icon">✅</div>
              <h2>Order Confirmed!</h2>
              <p>Thank you for your purchase. Your order has been successfully placed.</p>
              <div class="order-details">
                <p><strong>Order Number:</strong> {{ orderNumber }}</p>
                <p><strong>Total:</strong> ${{ getTotalPrice() }}</p>
                <p>You will receive a confirmation email shortly.</p>
              </div>
            </div>
          </div>
        </div>

        <div v-if="checkoutStep < 4" class="checkout-footer">
          <button v-if="checkoutStep > 1" class="btn-secondary" @click="prevStep">
            ← Back
          </button>
          <div class="spacer"></div>
          <button
            v-if="checkoutStep < 3"
            class="btn-primary"
            @click="nextStep"
            :disabled="!isStepValid"
          >
            Continue →
          </button>
          <button
            v-if="checkoutStep === 3"
            class="btn-primary"
            @click="placeOrder"
          >
            Place Order
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f0f8f0 0%, #e8f5e9 100%);
}

.hero {
  background: linear-gradient(135deg, #2c5f2d 0%, #1b3a1c 100%);
  color: white;
  padding: 4rem 2rem;
  text-align: center;
}

.hero-content h1 {
  margin: 0 0 1rem 0;
  font-size: 3rem;
  font-weight: bold;
}

.hero-content p {
  margin: 0;
  font-size: 1.3rem;
  opacity: 0.95;
}

.store-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 3rem 2rem;
}

.plants-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  justify-content: center;
}

@media (max-width: 1024px) {
  .plants-grid {
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1.5rem;
    justify-content: center;
  }
}

@media (max-width: 768px) {
  .plants-grid {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }

  .hero-content h1 {
    font-size: 2rem;
  }

  .hero-content p {
    font-size: 1rem;
  }
}

/* Cart Styles */
.cart-btn {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 2px solid rgba(255, 255, 255, 0.3);
  padding: 0.4rem 0.8rem;
  border-radius: 20px;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.3s ease;
}


.cart-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  border-color: rgba(255, 255, 255, 0.5);
}

.cart-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  display: flex;
  justify-content: flex-end;
}

.cart-sidebar {
  background: white;
  width: 100%;
  max-width: 400px;
  height: 100%;
  padding: 0;
  box-shadow: -2px 0 10px rgba(0, 0, 0, 0.1);
  overflow-y: auto;
}

.cart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid #eee;
  background: #f8f9fa;
}

.cart-header h2 {
  margin: 0;
  color: #2c5f2d;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #666;
  padding: 0.25rem;
  border-radius: 50%;
  transition: background 0.2s;
}

.close-btn:hover {
  background: #eee;
}

.empty-cart {
  text-align: center;
  padding: 3rem 1.5rem;
  color: #666;
}

.empty-cart-icon {
  font-size: 3rem;
  display: block;
  margin: 1rem 0;
  opacity: 0.5;
}

.cart-items {
  padding: 1.5rem;
}

.cart-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem 0;
  border-bottom: 1px solid #f0f0f0;
}

.cart-item:last-child {
  border-bottom: none;
}

.cart-item-image {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
}

.cart-item-info {
  flex: 1;
}

.cart-item-info h3 {
  margin: 0 0 0.25rem 0;
  font-size: 1rem;
  color: #2c5f2d;
}

.cart-item-price {
  margin: 0 0 0.5rem 0;
  font-weight: bold;
  color: #2c5f2d;
}

.quantity-controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.quantity-controls button {
  background: #f0f0f0;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.2s;
}

.quantity-controls button:hover {
  background: #e0e0e0;
}

.quantity-controls span {
  min-width: 30px;
  text-align: center;
  font-weight: bold;
}

.remove-btn {
  background: none;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 4px;
  transition: background 0.2s;
}

.remove-btn:hover {
  background: #ffeaea;
}

.cart-total {
  margin-top: 2rem;
  padding-top: 1rem;
  border-top: 2px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.checkout-btn {
  background: #2c5f2d;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
}

.checkout-btn:hover {
  background: #1b3a1c;
  transform: translateY(-1px);
}

@media (max-width: 480px) {
  .cart-sidebar {
    max-width: 100%;
  }

  .cart-item {
    flex-direction: column;
    text-align: center;
  }

  .cart-total {
    flex-direction: column;
    gap: 1rem;
  }
}

/* Header Actions */
.header-actions {
  display: flex;
  gap: 0.75rem;
  justify-content: center;
  margin-top: 0.5rem;
}

.signin-btn, .user-btn {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 2px solid rgba(255, 255, 255, 0.3);
  padding: 0.4rem 0.8rem;
  border-radius: 20px;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.signin-btn:hover, .user-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  border-color: rgba(255, 255, 255, 0.5);
}

/* Sign In Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
}

.signin-modal {
  background: white;
  border-radius: 12px;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid #eee;
  background: #f8f9fa;
}

.modal-header h2 {
  margin: 0;
  color: #2c5f2d;
}

.signin-form {
  padding: 1.5rem;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #2c5f2d;
}

.form-group input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s ease;
  box-sizing: border-box;
}

.form-group input:focus {
  outline: none;
  border-color: #2c5f2d;
}

.signin-submit-btn {
  width: 100%;
  background: #2c5f2d;
  color: white;
  border: none;
  padding: 0.75rem;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.signin-submit-btn:hover {
  background: #1b3a1c;
  transform: translateY(-1px);
}

.modal-footer {
  padding: 1.5rem;
  text-align: center;
  border-top: 1px solid #eee;
  background: #f8f9fa;
}

.link-btn {
  background: none;
  border: none;
  color: #2c5f2d;
  text-decoration: underline;
  cursor: pointer;
  font-size: inherit;
}

.link-btn:hover {
  color: #1b3a1c;
}

@media (max-width: 480px) {
  .signin-modal {
    margin: 1rem;
    width: calc(100% - 2rem);
  }

  .header-actions {
    flex-direction: column;
    gap: 0.5rem;
  }
}

/* Checkout Modal Styles */
.checkout-modal {
  background: white;
  border-radius: 12px;
  width: 90%;
  max-width: 800px;
  max-height: 90vh;
  overflow: hidden;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
}

.checkout-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid #eee;
  background: #f8f9fa;
}

.checkout-steps {
  display: flex;
  gap: 2rem;
}

.step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
}

.step-number {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  transition: all 0.3s ease;
}

.step:not(.active):not(.completed) .step-number {
  background: #e0e0e0;
  color: #666;
}

.step.active .step-number {
  background: #2c5f2d;
  color: white;
}

.step.completed .step-number {
  background: #4caf50;
  color: white;
}

.step-label {
  font-size: 0.8rem;
  color: #666;
}

.step.active .step-label {
  color: #2c5f2d;
  font-weight: 600;
}

.checkout-content {
  padding: 2rem;
  flex: 1;
  overflow-y: auto;
}

.checkout-step h2 {
  margin: 0 0 1.5rem 0;
  color: #2c5f2d;
  font-size: 1.5rem;
}

.checkout-form {
  max-width: 500px;
}

.form-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}

.form-row .form-group {
  flex: 1;
}

.security-notice {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 1rem;
  padding: 0.75rem;
  background: #f0f8f0;
  border-radius: 8px;
  color: #2c5f2d;
  font-size: 0.9rem;
}

.order-review {
  max-width: 600px;
}

.review-section {
  margin-bottom: 2rem;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 8px;
}

.review-section h3 {
  margin: 0 0 1rem 0;
  color: #2c5f2d;
  font-size: 1.1rem;
}

.review-item {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem 0;
  border-bottom: 1px solid #eee;
}

.review-item:last-child {
  border-bottom: none;
}

.review-total {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 2px solid #2c5f2d;
  text-align: right;
  font-size: 1.2rem;
}

.confirmation-content {
  text-align: center;
  padding: 2rem 0;
}

.success-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.confirmation-content h2 {
  color: #4caf50;
  margin-bottom: 1rem;
}

.order-details {
  background: #f0f8f0;
  padding: 1.5rem;
  border-radius: 8px;
  margin-top: 2rem;
}

.checkout-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-top: 1px solid #eee;
  background: #f8f9fa;
}

.spacer {
  flex: 1;
}

.btn-primary, .btn-secondary {
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: none;
}

.btn-primary {
  background: #2c5f2d;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #1b3a1c;
  transform: translateY(-1px);
}

.btn-primary:disabled {
  background: #ccc;
  cursor: not-allowed;
  transform: none;
}

.btn-secondary {
  background: white;
  color: #2c5f2d;
  border: 2px solid #2c5f2d;
}

.btn-secondary:hover {
  background: #f0f8f0;
}

@media (max-width: 768px) {
  .checkout-modal {
    width: 95%;
    max-height: 95vh;
  }

  .checkout-steps {
    gap: 1rem;
  }

  .step-label {
    font-size: 0.7rem;
  }

  .form-row {
    flex-direction: column;
    gap: 0;
  }

  .checkout-footer {
    flex-direction: column;
    gap: 1rem;
  }

  .checkout-footer .spacer {
    display: none;
  }
}

@media (max-width: 768px) {
  .header-actions {
    flex-direction: column;
    gap: 0.5rem;
    margin-top: 0.5rem;
  }

  .signin-btn, .user-btn, .cart-btn {
    padding: 0.5rem 1rem;
    font-size: 0.9rem;
  }
}
</style>
