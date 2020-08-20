<template>
  <div class="container">
    <div class="row">
      <div class="col-md-8 offset-md-2">
        <div v-for="(order, id) in quantity" :key="id">
          <div v-if="order" class="order-box">
            <div class="row">
                <img :src="products[id].image" @error="products[id].image = '/beers/default.png'" :alt="products[id].name" class="img-fluid col-md-3"/>
                <h2 class="title col-md-6" v-html="products[id].name"></h2>
                <p class="small-text text-muted col-md-3 text-right">
                  unit price: € {{ products[id].price }}
                </p>
            </div>
            
            <br />
            <hr />
            <div class="row">
              <p class="small-text text-muted col-md-3">
                Available Units: {{ products[id].quantity }}
              </p>

              <div class="row col-md-6"
                ><span class="col-md-4">Quantity: </span
                ><input
                  type="number"
                  name="units"
                  min="0"
                  :max="products[id].quantity"
                  class="col-md-8 float-left"
                  v-model="quantity[id]"
                  @change="checkUnits"
              /></div>

              <p class="small-text text-right col-md-3">
                Price: € {{ itemPice(id) }}
              </p>

            </div>
            
            
            
          </div>
        </div>
        <br />
        <div>
          <div v-if="!isLoggedIn">
            <h2>You need to login to continue</h2>
            <button class="col-md-4 btn btn-primary float-left" @click="login">
              Login
            </button>
            <button
              class="col-md-4 btn btn-danger float-right"
              @click="register"
            >
              Create an account
            </button>
          </div>
          <div v-if="isLoggedIn">
            <div class="row">
              <label for="address" class="col-md-3 col-form-label"
                >Delivery Address</label
              >
              <div class="col-md-9">
                <input
                  id="address"
                  type="text"
                  class="form-control"
                  v-model="address"
                  required
                />
              </div>
            </div>
            <br />
            <button
              class="col-md-4 btn btn-sm btn-success float-right"
              v-if="isLoggedIn"
              @click="placeOrder"
            >
              Continue
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { axiosGet, axiosPostPrivate } from "@/api/api";
import { uuid } from "vue-uuid";

export default {
  // props : ['pid'],
  data() {
    return {
      address: "",
      quantity: [],
      isLoggedIn: null,
      products: [],
      cart: [],
      orders: [],
      price:[]
    };
  },
  mounted() {
    this.isLoggedIn = localStorage.getItem("brewery.jwt") != null;
    this.cart = JSON.parse(localStorage.getItem("brewery.cart"));

    this.cart.forEach(order => {
      //    this.orders[order.beerId] = parseInt(order.units)

      //console.log(this.orders)

      if (this.quantity[order.beerId] !== undefined) {
        this.quantity[order.beerId] =
          parseInt(this.quantity[order.beerId]) + parseInt(order.units);
      } else {
        this.quantity[order.beerId] = parseInt(order.units);
      }
    });
    console.log(this.orders);
    console.log(this.quantity);
  },
  beforeMount() {
    axiosGet("/beers").then(data => {
      this.products = data;
    });

    if (localStorage.getItem("brewery.jwt") != null) {
      this.user = JSON.parse(localStorage.getItem("brewery.user"));
      this.token = localStorage.getItem("brewery.jwt");
    }
  },
  methods: {
    login() {
      this.$router.push({
        name: "login",
        params: { nextUrl: this.$route.fullPath }
      });
    },
    itemPice(id){
      this.price[id] = (this.quantity[id] * this.products[id].price*1).toFixed(2);
      return this.price[id];
    },
    register() {
      this.$router.push({
        name: "register",
        params: { nextUrl: this.$route.fullPath }
      });
    },

    placeOrder(e) {
      e.preventDefault();

      console.log(e);
      let billRef = uuid.v4();
      let orders = [];
      var id =0;
      for (id = 0; id < this.quantity.length; id++) {
        if (this.quantity[id] !== undefined &&
          this.quantity[id] != 0)

          orders.push({
              bill: billRef,
              beer_id: this.products[id].id,
              user_id: this.user.id,
              brewery_id: this.products[id].brewery_id,
              quantity: this.quantity[id],
              price: this.price[id],
              delivered: false
            });

      }
      let body = {
        orders: orders
      };
      console.log(body);
      //console.log(body);

      axiosPostPrivate("/orders", body, this.token).then(response => {
        console.log(response);
        this.$router.push("/confirmation");
      })
      .catch(e => console.log(e));
    },
    checkUnits() {
      if (this.quantity > this.product.units) {
        this.quantity = this.product.units;
      }
    }
  }
};
</script>

<style scoped>
.small-text {
  font-size: 18px;
}
.order-box {
  border: 1px solid #cccccc;
  padding: 10px 15px;
  height: 300px;
}

.order-box img{
  height: 100%;
}
.title {
  font-size: 36px;
}
</style>
