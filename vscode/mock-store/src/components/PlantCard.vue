<template>
  <div class="plant-card">
    <div class="plant-image">
      <img :src="plant.image" :alt="plant.name" />
    </div>
    <div class="plant-info">
      <h2>{{ plant.name }}</h2>
      <p class="scientific-name">{{ plant.scientificName }}</p>
      
      <div class="care-section">
        <h3>Care Requirements</h3>
        <p><strong>Light:</strong> {{ plant.care.light }}</p>
        <p><strong>Water:</strong> {{ plant.care.water }}</p>
        <p><strong>Humidity:</strong> {{ plant.care.humidity }}</p>
      </div>
      
      <div class="important-info">
        <h3>Important</h3>
        <p>{{ plant.importantInfo }}</p>
      </div>
      
      <div class="price-section">
        <div class="price">
          <span class="amount">${{ plant.price }}</span>
        </div>
        <button class="add-to-cart-btn" @click="addToCart">
          🛒 Add to Cart
        </button>
      </div>
    </div>
    </div>
</template>

<script setup>
const props = defineProps({
  plant: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['add-to-cart'])

const addToCart = () => {
  emit('add-to-cart', props.plant)
}
</script>

<style scoped>
.plant-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  cursor: pointer;
  height: 100%;
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 400px;
}

.plant-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.15);
}

.plant-image {
  width: 100%;
  height: 200px;
  overflow: hidden;
  background: linear-gradient(135deg, #e8f5e9 0%, #f0f8f0 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.plant-image::before {
  content: "🌿";
  font-size: 3rem;
  opacity: 0.3;
  position: absolute;
}

.plant-image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: opacity 0.3s ease;
}

.plant-image img:not([src]) {
  opacity: 0;
}

.plant-image img[src=""] {
  opacity: 0;
}

.plant-info {
  padding: 1.5rem;
  flex: 1;
  display: flex;
  flex-direction: column;
}

h2 {
  margin: 0 0 0.25rem 0;
  color: #2c5f2d;
  font-size: 1.5rem;
}

.scientific-name {
  margin: 0 0 1rem 0;
  color: #666;
  font-style: italic;
  font-size: 0.9rem;
}

.care-section {
  margin-bottom: 1rem;
}

.care-section h3,
.important-info h3 {
  margin: 0.75rem 0 0.5rem 0;
  color: #2c5f2d;
  font-size: 1rem;
}

.care-section p,
.important-info p {
  margin: 0.25rem 0;
  font-size: 0.9rem;
  color: #555;
  line-height: 1.4;
}

.important-info {
  margin-bottom: 1rem;
  padding: 0.75rem;
  background: #f0f8f0;
  border-left: 3px solid #2c5f2d;
  border-radius: 4px;
}

.price-section {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  padding-top: 1rem;
  border-top: 1px solid #eee;
  margin-top: auto;
}

.price {
  text-align: center;
}

.amount {
  font-size: 1.8rem;
  font-weight: bold;
  color: #2c5f2d;
}

.add-to-cart-btn {
  background: #2c5f2d;
  color: white;
  border: none;
  padding: 0.75rem 1rem;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.add-to-cart-btn:hover {
  background: #1b3a1c;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(44, 95, 45, 0.3);
}

.add-to-cart-btn:active {
  transform: translateY(0);
}
</style>
