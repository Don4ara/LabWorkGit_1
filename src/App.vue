<template>
  <div id="app">
    <nav class="navbar navbar-expand-md navbar-dark bg-dark mb-4 d-flex">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Lab Coffee</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarCollapse"
                aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav me-auto mb-2 mb-md-0">
            <li class="nav-item">
              <a class="nav-link" aria-current="page" href="#">Главное</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Блог</a>
            </li>
            <li class="nav-item">
              <a class="nav-link active" href="#" data-bs-toggle="offcanvas" data-bs-target="#offcanvasCart"
                 aria-controls="offcanvasCart">
                Корзина <span class="badge bg-danger" v-if="cartItems.length">{{ cartItems.length }}</span>
              </a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Каталог</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">О нас</a>
            </li>
          </ul>
        </div>
      </div>
      <div class="container-fluid">
        <i class="bi bi-bag" style="color: black"></i>
      </div>
    </nav>

    <div class="container">
      <h1>Каталог</h1>
      <div class="row mb-4">
        <div class="col-md-3">
          <h5>Фильтр</h5>
          <div class="mb-3">
            <label for="typeFilter" class="form-label">Тип кофе</label>
            <select class="form-select" id="typeFilter" v-model="filters.type">
              <option value="">Все</option>
              <option value="Зерновой">Зерновой</option>
              <option value="Молотый">Молотый</option>
              <option value="Растворимый">Растворимый</option>
            </select>
          </div>
          <div class="mb-3">
            <label for="volumeFilter" class="form-label">Объем (г)</label>
            <input type="range" class="form-range" id="volumeFilter" v-model="filters.volume" min="0" max="1000"
                   step="50">
            <div class="d-flex justify-content-between">
              <span>0</span>
              <span>1000</span>
            </div>
          </div>
          <div class="mb-3">
            <label for="priceFilter" class="form-label">Цена (₽)</label>
            <input type="range" class="form-range" id="priceFilter" v-model="filters.price" min="0" max="2000"
                   step="100">
            <div class="d-flex justify-content-between">
              <span>0</span>
              <span>2000</span>
            </div>
          </div>
          <button class="btn btn-secondary w-100" @click="resetFilters">Сбросить фильтры</button>
        </div>
        <div class="col-md-9">
          <div class="row row-cols-1 row-cols-md-3 g-4">
            <div class="col" v-for="product in filteredProducts" :key="product.id">
              <div class="card h-100">
                <img :src="product.image" class="card-img-top" alt="Product Image">
                <div class="card-body">
                  <h5 class="card-title">{{ product.name }}</h5>
                  <p class="card-text">{{ product.volume }} г</p>
                  <p class="card-text">{{ product.price }} ₽</p>
                  <button class="btn btn-primary" @click="addToCart(product)">Добавить в корзину</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Offcanvas for Cart -->
    <div class="offcanvas offcanvas-start" tabindex="-1" id="offcanvasCart" aria-labelledby="offcanvasCartLabel">
      <div class="offcanvas-header">
        <h5 class="offcanvas-title" id="offcanvasCartLabel">Корзина</h5>
        <button type="button" class="btn-close text-reset" data-bs-dismiss="offcanvas" aria-label="Close"></button>
      </div>
      <div class="offcanvas-body d-flex flex-column">
        <!-- Bootstrap Alerts -->
        <div v-for="(message, index) in alertMessages" :key="index" class="alert alert-danger alert-dismissible fade show" role="alert">
          {{ message }}
          <button type="button" class="btn-close" @click="closeAlert(index)" aria-label="Close"></button>
        </div>
        <ul class="list-group mb-4">
          <li class="list-group-item d-flex justify-content-between align-items-center" v-for="item in cartItems" :key="item.id">
            {{ item.name }} - {{ item.quantity }} x {{ item.price }}₽
            <div>
              <button class="btn btn-sm btn-outline-secondary" @click="decreaseQuantity(item)">-</button>
              <button class="btn btn-sm btn-outline-secondary" @click="increaseQuantity(item)">+</button>
            </div>
          </li>
        </ul>
        <p v-if="cartItems.length === 0">Ваша корзина пуста.</p>
        <div class="mt-auto" v-if="cartItems.length > 0">
          <p class="fw-bold">Итого: {{ totalPrice }} ₽</p>
          <button type="button" class="btn btn-danger w-100" data-bs-toggle="modal" data-bs-target="#orderModal" @click="openOrderModal">Оформить заказ</button>
        </div>
      </div>
    </div>


    <!-- Order Modal -->
    <div class="modal fade" id="orderModal" tabindex="-1" aria-labelledby="orderModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="orderModalLabel">Оформление заказа</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <h6>Ваши товары</h6>
            <ul id="cartItems" class="list-group mb-4">
              <li class="list-group-item" v-for="item in cartItems" :key="item.id">
                {{ item.name }} - {{ item.quantity }} x {{ item.price }}₽
              </li>
            </ul>

            <h6>Данные клиента</h6>
            <form @submit.prevent="placeOrder">
              <div class="mb-3">
                <label for="name" class="form-label">Имя</label>
                <input type="text" class="form-control" id="name" v-model="orderForm.name" required>
              </div>
              <div class="mb-3">
                <label for="email" class="form-label">Email</label>
                <input type="email" class="form-control" id="email" v-model="orderForm.email" required>
              </div>
              <div class="mb-3">
                <label for="address" class="form-label">Адрес</label>
                <input type="text" class="form-control" id="address" v-model="orderForm.address" required>
              </div>
              <div class="mb-3">
                <label for="paymentMethod" class="form-label">Способ оплаты</label>
                <select class="form-select" id="paymentMethod" v-model="orderForm.paymentMethod" required>
                  <option selected disabled value="">Выберите...</option>
                  <option value="card">Карта</option>
                  <option value="paypal">PayPal</option>
                  <option value="sberbank">Сбербанк</option>
                  <option value="SPB">CПБ</option>
                </select>
              </div>
              <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Закрыть</button>
                <button type="submit" class="btn btn-primary">Подтвердить заказ</button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>

    <!-- Order Animation -->
    <div v-if="showOrderAnimation" class="order-animation">
      <svg class="truck-svg" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 512"><path d="M624 368h-16V275.8c0-16.4-6.5-32.1-18.1-43.9L462.1 83.9C450.3 72.3 434.6 65.8 418.2 65.8H384V56c0-30.9-25.1-56-56-56H56C25.1 0 0 25.1 0 56V344c0 30.9 25.1 56 56 56 0 35.3 28.7 64 64 64s64-28.7 64-64h128c0 35.3 28.7 64 64 64s64-28.7 64-64h32c35.3 0 64-28.7 64-64v-8h16c8.8 0 16-7.2 16-16v-16c0-8.8-7.2-16-16-16zM144 432c-17.7 0-32-14.3-32-32s14.3-32 32-32 32 14.3 32 32-14.3 32-32 32zm256 0c-17.7 0-32-14.3-32-32s14.3-32 32-32 32 14.3 32 32-14.3 32-32 32zm96-128h-64c-8.8 0-16 7.2-16 16v8c0 30.9-25.1 56-56 56s-56-25.1-56-56v-8H192v8c0 30.9-25.1 56-56 56s-56-25.1-56-56v-8H56c-8.8 0-16-7.2-16-16V56c0-8.8 7.2-16 16-16h272c8.8 0 16 7.2 16 16v96h56.2c5.6 0 10.9 2.2 14.8 6.1l128 128c3.9 3.9 6.1 9.2 6.1 14.8V368zM384 312c-13.3 0-24-10.7-24-24s10.7-24 24-24 24 10.7 24 24-10.7 24-24 24z"/></svg>
    </div>


  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      products: [
        {
          id: 1,
          name: 'Premium Зерновой кофе 250г',
          volume: 250,
          price: 350,
          stock: 5,
          type: 'Зерновой',
          image: 'https://via.placeholder.com/150',
        },
        {
          id: 2,
          name: 'Premium Молотый кофе 500г',
          volume: 500,
          price: 500,
          stock: 5,
          type: 'Молотый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 3,
          name: 'Premium Растворимый кофе 300г',
          volume: 300,
          price: 400,
          stock: 5,
          type: 'Растворимый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 4,
          name: 'Premium Молотый кофе 500г',
          volume: 500,
          price: 550,
          type: 'Молотый',
          stock: 5,
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 5,
          name: 'Premium Зерновой кофе 1000г',
          volume: 1000,
          price: 800,
          type: 'Зерновой',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 6,
          name: 'Premium Растворимый кофе 250г',
          volume: 250,
          price: 350,
          type: 'Растворимый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 7,
          name: 'Premium Зерновой кофе 500г',
          volume: 500,
          price: 450,
          type: 'Зерновой',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 8,
          name: 'Premium Молотый кофе 750г',
          volume: 750,
          price: 600,
          type: 'Молотый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 9,
          name: 'Premium Зерновой кофе 1000г',
          volume: 1000,
          price: 950,
          type: 'Зерновой',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 10,
          name: 'Premium Растворимый кофе 250г',
          volume: 250,
          price: 300,
          type: 'Растворимый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 11,
          name: 'Premium Молотый кофе 500г',
          volume: 500,
          price: 450,
          type: 'Молотый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 12,
          name: 'Premium Растворимый кофе 750г',
          volume: 750,
          price: 500,
          type: 'Растворимый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 13,
          name: 'Premium Зерновой кофе 1000г',
          volume: 1000,
          price: 850,
          stock: 5,
          type: 'Зерновой',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 14,
          name: 'Premium Молотый кофе 250г',
          volume: 250,
          price: 350,
          stock: 5,
          type: 'Молотый',
          image: 'https://via.placeholder.com/150'
        },
        {
          id: 15,
          name: 'Premium Растворимый кофе 500г',
          volume: 500,
          price: 400,
          stock: 5,
          type: 'Растворимый',
          image: 'https://via.placeholder.com/150'
        }

// Add more products here
      ],
      cartItems: [],
      filters: {
        type: '',
        volume: 1000,
        price: 2000,
      },
      orderForm: {
        name: '',
        email: '',
        address: '',
        paymentMethod: '',
      },
      showOrderAnimation: false,
      alertMessages: [],
    };
  },
  computed: {
    filteredProducts() {
      return this.products.filter(product => {
        return (
            (!this.filters.type || product.type === this.filters.type) &&
            product.volume <= this.filters.volume &&
            product.price <= this.filters.price
        );
      });
    },
    totalPrice() {
      return this.cartItems.reduce((total, item) => total + item.price * item.quantity, 0);
    }
  },
  methods: {
    addToCart(product) {
      const cartItem = this.cartItems.find(item => item.id === product.id)
      if (cartItem) {
        if (cartItem.quantity + 1 > product.stock) {
          this.alertMessages.push(`Извините, доступно только ${product.stock} единиц(ы) товара "${product.name}".`);
          return;
        }
        cartItem.quantity++;
      } else {
        this.cartItems.push({ ...product, quantity: 1 });
      }
    },

    closeAlert(index) {
      this.alertMessages.splice(index, 1);
    },
    decreaseQuantity(item) {
      if (item.quantity > 1) {
        item.quantity--;
      } else {
        this.cartItems = this.cartItems.filter(i => i.id !== item.id);
      }
    },
    increaseQuantity(item) {
      const product = this.products.find(product => product.id === item.id)
      if (item.quantity + 1 > product.stock) {
        this.alertMessages.push(`Извините, доступно только ${product.stock} единиц(ы) товара "${product.name}".`);
        return;
      }
      item.quantity++;
    },
    resetFilters() {
      this.filters = {
        type: '',
        volume: 1000,
        price: 2000,
      };
    },
    placeOrder() {
      this.showOrderAnimation = true;
      setTimeout(() => {
        this.showOrderAnimation = false;
      }, 3000); // Duration of the animation

      this.cartItems = [];
      this.orderForm = {
        name: '',
        email: '',
        address: '',
        paymentMethod: '',
      };

      const orderModal = bootstrap.Modal.getInstance(document.getElementById('orderModal'));
      const cartOffcanvas = bootstrap.Offcanvas.getInstance(document.getElementById('offcanvasCart'));

      orderModal.hide();
      cartOffcanvas.hide();
    }
  }
};
</script>

