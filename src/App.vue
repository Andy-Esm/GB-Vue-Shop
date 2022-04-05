<script>
const API = 'https://raw.githubusercontent.com/GeekBrainsTutorial/online-store-api/master/responses';
export default {
  data() {
    return {
      catalogUrl: '/catalogData.json',
      cartUrl: '/getBasket.json',
      products: [],
      filtered: [],
      cartItems:[],
      imgCatalog: 'https://via.placeholder.com/200x150',
      imgCart: 'https://via.placeholder.com/50x100',
      userSearch: '',
    }
  },
  methods: {
    filter(value) {
      const regexp = new RegExp(this.userSearch, 'i');
      this.filtered = this.products.filter(el => regexp.test(el.product_name));
    },
    getJson(url) {
      return fetch(url)
      .then(result => result.json())
      .catch(error => {
        console.log(error);
      })
    },
    addProduct(item) {
      console.log(item.id_product);
      this.getJson(`${API}/addToBasket.json`).then(data =>{
        if(data.result === 1){
          let find = this.cartItems.find(el => el.id_product === item.id_product)
          if(find){
         find.quantity++
        }else {
            let prod = Object.assign({quantity: 1}, item);
            this.cartItems.push(prod)
          }
        }
      })

    },
    showCart(){
      let cartVis = document.querySelector('.cart-block')
      cartVis.classList.toggle('invisible')
    },

    /**
     * @remove = удаляет элементы из корзины
     * @param item - элементы корзины.
     * @param data.result - проверка на доступность проводить манипуляции с корзиной
     * **/
    remove(item){
      this.getJson(`${API}/addToBasket.json`).then(data => {
        if(data.result === 1 && item.quantity > 1){
          item.quantity--
        }else {
          this.cartItems.splice(this.cartItems.indexOf(item),1)
        }
      })
    }
  },
  mounted() {
    this.getJson(`${API + this.cartUrl}`).then(data => {
      for(let el of data.contents) {
        this.cartItems.push(el)
      }
    })

    this.getJson(`${API + this.catalogUrl}`)
    .then(data => {
      for(let el of data) {
        this.$data.products.push(el);
        this.$data.filtered.push(el);
      }
    });
    this.getJson(`getProducts.json`)
    .then(data => {
      for(let el of data) {
        this.products.push(el);
        this.filtered.push(el);
      }
    })
  }
}
</script>

<template>
  <header>
        <div class="logo">E-shop</div>
        <div class="cart">
            <form action="#" class="search-form" @submit.prevent="filter">
                <input type="text" class="search-field" v-model="userSearch">
                <button class="btn-search" type="submit">
                    <i class="fas fa-search"></i>
                </button>
            </form>
            <button class="btn-cart" type="button" @click="showCart">Корзина</button>
            <div class="cart-block" v-show="showCart">
              <p v-if="!cartItems.length">Cart is empty</p>
              <div class="cart-item" v-for="item of cartItems" :key="item.id_product">
                <div class="product-bio">
                  <img src="imgCart" alt="cart image">
                  <div class="product-title">{{item.product_name}}</div>
                  <div class="product-quantity">{{item.quantity}}</div>
                  <div class="product-single-price">{{item.price}} each</div>
                </div>
                <div class="right-block">
                  <div class="product-price">{{item.quantity * item.price}}</div>
                  <button class="del-btn" @click="remove(item)">&times;</button>
                </div>
              </div>
            </div>
        </div>
    </header>
    <main>
        <div class="products">
            <div class="product-item" v-for="product of filtered" :key="product.id_product">
                <img :src="imgCatalog" alt="Some img">
                <div class="desc">
                    <h3>{{ product.product_name }}</h3>
                    <p>{{ product.price }} $</p>
                    <button class="buy-btn" @click="addProduct(product)">Купить</button>
                </div>
            </div>
        </div>
    </main>
</template>

<style>
@import "assets/normalize.css";
@import "assets/style.css";

</style>
