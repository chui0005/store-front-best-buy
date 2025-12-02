<template>
  <TopNav :cartItemCount="cartItemCount"/>
  <router-view
    :products="products"
    :cartItems="cartItems"
    @addToCart="addToCart"
    @removeFromCart="removeFromCart"
    @submitOrder="submitOrder"
  ></router-view>
</template>

<script>
import TopNav from './components/TopNav.vue'

export default {
  name: 'App',
  components: {
    TopNav
  },
  data() {
    return {
      cartItems: [],
      products: [],
    }
  },
  computed: {
    cartItemCount() {
      return this.cartItems.reduce((total, item) => {
        return total + item.quantity
      }, 0)
    }
  },
  mounted() {
    this.getProducts()
  },
  methods: {
    getProducts() {
      fetch('/products')
        .then(response => response.json())
        .then(products => {
          console.log('success getting proxy products')
          this.products = products
        })
        .catch(error => {
          console.log(error)
          alert('Error occurred while fetching products')
        })
    },
    addToCart({ productId, quantity }) {
      const existingCartItem = this.cartItems.find(
        item => item.product.id == productId
      )
      if (existingCartItem) {
        existingCartItem.quantity += quantity
      } else {
        const product = this.products.find(product => product.id == productId)
        this.cartItems.push({ product, quantity })
      }
    },
    removeFromCart(index) {
      this.cartItems.splice(index, 1)
    },
    submitOrder() {
      const order = {
        customerId: Math.floor(Math.random() * 10000000000).toString(),
        items: this.cartItems.map(item => {
          return {
            productId: item.product.id,
            quantity: item.quantity,
            price: item.product.price
          }
        })
      }

      console.log(JSON.stringify(order));

      fetch(`/order`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(order)
      })
        .then(response => {
          console.log(response)
          if (!response.ok) {
            alert('Error occurred while submitting order')
          } else {
            this.cartItems = []
            alert('Order submitted successfully')
          }
        })
        .catch(error => {
          console.log(error)
          alert('Error occurred while submitting order')
        })
    }
  },
}
</script>

<style>
/* Global layout + Best Buy–style palette */
body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue",
    Arial, "Noto Sans", sans-serif;
  background: #f4f6f9; /* light, neutral retail background */
  color: #1f2933;
}

#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  max-width: 1200px;
  margin: 100px auto 40px; /* space for a fixed/top nav + page breathing room */
  padding: 0 1.5rem 3rem;
}



/* Top navigation base (TopNav.vue will provide structure, this styles it) */
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #0046be;
  color: #ffffff;
  padding: 0.75rem 1.5rem;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

nav .brand {
  font-weight: 700;
  letter-spacing: 0.03em;
}

nav .brand span.highlight {
  color: #ffe000; /* Best Buy yellow accent */
}

ul {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
}

li {
  margin: 0 0.75rem;
}

a {
  color: #ffffff;
  text-decoration: none;
  font-size: 0.95rem;
}

a:hover,
a:focus {
  text-decoration: underline;
}

/* Primary buttons (Add to cart, etc.) */
button {
  padding: 0.6rem 1.1rem;
  background-color: #FFD100;
  color: black;
  border: none;
  border-radius: 999px;
  cursor: pointer;
  height: 42px;
  font-weight: 600;
  font-size: 0.95rem;
  letter-spacing: 0.02em;
  transition: background-color 0.15s ease, transform 0.05s ease;
}

button:hover,
button:focus {
  background-color: #00349a;
  transform: translateY(-1px);
}

button:active {
  transform: translateY(0);
}

/* Product listing grid (store-front feel) */
.product-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  grid-gap: 1.5rem;
  margin-top: 1.5rem;
}

/* Product cards styled like retail tiles */
.product-card {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: space-between;
  padding: 1rem 1.25rem 1.25rem;
  border-radius: 0.75rem;
  background-color: #ffffff;
  box-shadow: 0 4px 12px rgba(15, 23, 42, 0.08);
  border: 1px solid #e5e7eb;
}

.product-card img {
  max-width: 100%;
  height: auto;
  margin-bottom: 0.75rem;
  object-fit: contain;
}

.product-card a {
  text-decoration: none;
  color: #111827;
}

.product-card h2 {
  font-weight: 700;
  margin-bottom: 0.25rem;
  font-size: 1.05rem;
}

.product-card p {
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
  color: #4b5563;
}

.product-price {
  font-weight: 700;
  font-size: 1.15rem;
  color: #0046be;
  margin-bottom: 0.5rem;
}

/* Quantity + add-to-cart row */
.product-controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 0.75rem;
  gap: 0.75rem;
}

.product-controls p {
  margin: 0;
  font-size: 0.9rem;
  color: #374151;
}

.quantity-input {
  width: 60px;
  height: 32px;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  padding: 0 0.5rem;
  font-size: 0.9rem;
}

/* Shopping cart panel */
.shopping-cart {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  padding: 1.5rem;
  margin-top: 1.5rem;
  background-color: #ffffff;
  border-radius: 0.75rem;
  box-shadow: 0 4px 12px rgba(15, 23, 42, 0.08);
  border: 1px solid #e5e7eb;
}

.shopping-cart h2 {
  font-size: 1.3rem;
  margin: 0 0 1rem;
}

/* Cart table */
.shopping-cart-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.shopping-cart-table th,
.shopping-cart-table td {
  padding: 0.6rem 0.4rem;
  text-align: left;
  border-bottom: 1px solid #e5e7eb;
}

.shopping-cart-table th {
  font-weight: 600;
  background-color: #f3f4f6;
}

.shopping-cart-table td img {
  display: block;
  margin: 0 auto;
}

/* Checkout button as a strong CTA */
.checkout-button {
  margin-top: 1.25rem;
  padding: 0.75rem 1.5rem;
  background-color: #ffe000;
  color: #111827;
  border-radius: 999px;
  border: none;
  font-weight: 700;
  font-size: 0.95rem;
  cursor: pointer;
  align-self: flex-end;
  transition: background-color 0.15s ease, transform 0.05s ease;
}

.checkout-button:hover,
.checkout-button:focus {
  background-color: #ffd200;
  transform: translateY(-1px);
}
</style>
