<template>
  <div class="container">
    <div class="row">
      <div class="col-md-8 offset-md-2">
        <div v-for="(order, id) in quantity" :key="id">
          <div v-if="order" class="order-box">
            <img :src="products[id].image" :alt="products[id].name" />
            <h2 class="title" v-html="products[id].name"></h2>
            <p class="small-text text-muted float-left">
              € {{ products[id].price }}
            </p>
            <p class="small-text text-muted float-right">
              Available Units: {{ products[id].units }}
            </p>
            <br />
            <hr />
            <label class="row"
              ><span class="col-md-2 float-left">Quantity: </span
              ><input
                type="number"
                name="units"
                min="1"
                :max="products[id].units"
                class="col-md-2 float-left"
                v-model="quantity[id]"
                @change="checkUnits"
            /></label>
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
      orders: []
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

    register() {
      this.$router.push({
        name: "register",
        params: { nextUrl: this.$route.fullPath }
      });
    },

    placeOrder(e) {
      e.preventDefault();
      let body = {
        bill: uuid.v4(),
        beer_id: this.product.id,
        user_id: this.user.id,
        brewery_id: this.product.brewery_id,
        quantity: this.quantity,
        unitprice: this.product.price,
        delivered: false
      };
      //console.log(body);

      axiosPostPrivate("/orders", body, this.token).then(response => {
        console.log(response);
        this.$router.push("/confirmation");
      });
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
}
.title {
  font-size: 36px;
}
</style>
