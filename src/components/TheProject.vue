<template>
  <section id="usr">
    <h2>Bienvenido {{ username }}</h2>
    <input type="text" placeholder="USERNAME" v-model="username" />
  </section>

  <section id="shop">
    <section>
      <label for="srcProducts">Search Product</label>
      <input type="text" name="srcProducts" id="srcProducts" v-model="search" />
    </section>
    <section>
      <label for="ordSelect">Order By</label>
      <select
        name="ordSelect"
        id="ordSelect"
        @change="orderTable"
        v-model="order"
      >
        <option value="id">id</option>
        <option value="category">Category</option>
        <option value="name">Name</option>
        <option value="price">Price</option>
      </select>
      <select
        name="ctgselect"
        id="ctgselect"
        :style="[
          order == 'category' ? { display: 'inline' } : { display: 'none' },
        ]"
        @change="filterCategory"
        v-model="category"
      >
        <option v-for="option in options" :key="option">{{ option }}</option>
      </select>
    </section>
    <section>
      <button
        id="storage"
        :style="[local ? { display: 'inline' } : { display: 'none' }]"
        @click="eraseLocal"
      >
        Erase Local Cart
      </button>
    </section>
    <section>
      <button
        id="finish"
        :style="[
          productNumber > 0 ? { display: 'inline' } : { display: 'none' },
        ]"
        @click="orderProducts"
      >
        Order
      </button>
    </section>
  </section>

  <section>
    <nav>
      <a href="#" @click="cartTable = !cartTable">Cart</a> {{ productNumber }}
    </nav>
    <p id="del" hidden>{{ deleted }}</p>
  </section>

  <section
    id="crtTable"
    :style="[cartTable ? { display: 'block' } : { display: 'none' }]"
  >
    <table>
      <thead>
        <tr>
          <th>id</th>
          <th>Category</th>
          <th>Name</th>
          <th>Price</th>
          <th>Quantity</th>
          <th>Total Price</th>
          <th>Cart</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in cart" :key="product.id">
          <td>{{ product.id }}</td>
          <td>{{ product.category }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}&euro;</td>
          <td>{{ product.amount }}</td>
          <td>{{ product.total }}&euro;</td>
          <td>
            <button @click="addProduct">+</button>
            <button @click="removeProduct">-</button>
          </td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <th colspan="5">TOTAL</th>
          <th colspan="2">{{ total }}&euro;</th>
        </tr>
      </tfoot>
    </table>
  </section>

  <section>
    <table>
      <thead>
        <tr>
          <th>id</th>
          <th>Category</th>
          <th>Name</th>
          <th>Price</th>
          <th>Cart</th>
          <th>Remove</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in filterProducts" :key="product.id">
          <td>{{ product.id }}</td>
          <td
            :style="[
              product.category == 'Alcoholic'
                ? { 'background-color': 'rgba(204, 0, 0, 0.66)' }
                : {},
            ]"
          >
            {{ product.category }}
          </td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>
            <button @click="addProduct">Add</button>
          </td>
          <td>
            <button @click="deleteProduct">Remove</button>
          </td>
        </tr>
      </tbody>
    </table>
  </section>
</template>

<script>
export default {
  name: "TheProject",
  props: {
    URL: {
      type: String,
      required: false,
    },
  },
  data() {
    return {
      username: "",
      order: "",
      category: "",
      search: "",
      deleted: "",
      total: 0,
      amount: 0,
      price: 0,
      productNumber: 0,
      cartTable: false,
      local: false,
      products: [],
      cart: [],
      options: [],
      newProduct: {
        id: 0,
        category: "",
        name: "",
        price: 0,
      },
    };
  },
  methods: {
    appendProducts() {
      fetch(this.URL + "/products")
        .then((response) => response.json())
        .then((json) => (this.products = json));
    },
    orderTable() {
      if (this.order == "id") {
        (this.category = ""), this.products.sort((a, b) => a.id - b.id);
      }
      if (this.order == "name") {
        (this.category = ""),
          this.products.sort((a, b) => a.name.localeCompare(b.name));
      }
      if (this.order == "price") {
        (this.category = ""), this.products.sort((a, b) => a.price - b.price);
      }
      if (this.order == "category") {
        fetch(this.URL + "/categories")
          .then((response) => response.json())
          .then(
            (json) =>
              (this.options = [...json].sort((a, b) => a.localeCompare(b)))
          );
      }
    },
    addProduct(event) {
      if (
        !this.cart.find(
          (element) =>
            element.id == event.target.closest("tr").children[0].innerHTML
        )
      ) {
        this.newProduct.id = event.target.closest("tr").children[0].innerHTML;
        this.newProduct.category =
          event.target.closest("tr").children[1].innerHTML;
        this.newProduct.name = event.target.closest("tr").children[2].innerHTML;
        this.newProduct.price = +event.target
          .closest("tr")
          .children[3].innerHTML.replace(/[^\w.]/g, "");
        this.cart.push(this.newProduct);
        this.amount = this.newProduct.amount = 1;
        this.price = event.target
          .closest("tr")
          .children[3].innerHTML.replace(/[^\w.]/g, "");
        this.newProduct.total = this.productTotal;
        this.reset();
      } else {
        let position = this.cart.indexOf(
          this.cart.find(
            (element) =>
              element.id == event.target.closest("tr").children[0].innerHTML
          )
        );
        this.amount = ++this.cart[position].amount;
        this.price = this.cart[position].price;
        this.cart[position].total = this.productTotal;
        this.reset();
      }
      this.cartTotal();
      this.cartProducts();
      localStorage.setItem("cart", JSON.stringify(this.cart));
    },
    removeProduct(event) {
      let position = this.cart.indexOf(
        this.cart.find(
          (element) =>
            element.id == event.target.closest("tr").children[0].innerHTML
        )
      );

      if (--this.cart[position].amount == 0) {
        this.cart.splice(position, 1);
      } else {
        this.amount = this.cart[position].amount;
        this.price = this.cart[position].price;
        this.cart[position].total = this.productTotal;
      }
      this.cartTotal();
      this.cartProducts();
      localStorage.setItem("cart", JSON.stringify(this.cart));
    },
    cartTotal() {
      this.total = 0;
      this.cart.forEach((element) => {
        this.total += +element.total;
      });
      this.total.toFixed(2);
    },
    cartProducts() {
      this.productNumber = 0;
      this.cart.forEach((element) => (this.productNumber += +element.amount));
    },
    deleteProduct(event) {
      fetch(
        this.URL +
          "/products/" +
          event.target.closest("tr").children[0].innerHTML,
        {
          method: "DELETE",
          methods: {
            "Content-Type": "application/json",
          },
        }
      ).then((response) => {
        if (response.ok) {
          this.deleted = "The product has been deleted successfully.";
          this.message();
        } else {
          this.deleted = `Error ${response.status}`;
        }
      });
    },
    storedCart() {
      if (localStorage.getItem("cart")) {
        this.cart = JSON.parse(localStorage.getItem("cart"));
        this.local = true;
      }
    },
    eraseLocal() {
      localStorage.removeItem("cart");
    },
    message(theMessage) {
      let par = document.getElementById("del");
      par.hidden = false;
      if (theMessage) {
        this.deleted = theMessage;
      }
      setTimeout(() => (par.hidden = true), 5000);
    },
    reset() {
      this.newProduct = {
        id: 0,
        category: "",
        name: "",
        price: 0,
      };
      this.amount = 0;
      this.price = 0;
    },
    orderProducts() {
      let ordProducts = [];
      this.cart.forEach(product => {
        ordProducts.push({'id': product.id, 'amount': product.amount})
      });
      let finalOrder = {
        user: this.username,
        products: ordProducts,
      };
      fetch(this.URL + '/orders', {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(finalOrder),
      }).then((response) => {
        if (response.ok) {
          this.message("The products have been ordered successfully");
          this.eraseLocal();
        } else {
          this.message(`Error ${response.status}`);
        }
      });
    },
  },
  computed: {
    filterProducts() {
      if (this.category) {
        if (this.category == "Cars" || this.category == "Fashion") {
          this.message(`There aren't any products of ${this.category}`);
        }
        return this.products.filter((product) => {
          return product.category == this.category;
        });
      }
      return this.products.filter((product) => {
        return product.name.toLowerCase().match(this.search.toLowerCase());
      });
    },
    productTotal() {
      return (this.price * this.amount).toFixed(2);
    },
  },
  mounted() {
    this.appendProducts();
    this.storedCart();
    this.cartProducts();
    this.cartTotal();
  },
};
</script>

<style scoped>
#usr {
  margin: 0 auto;
  width: 20%;
  text-align: center;
}

#usr h2 {
  font-size: large;
  margin-bottom: 0.2em;
}

#usr input {
  text-align: center;
  opacity: 20%;
  margin-bottom: 1.8em;
  transition: opacity 1s;
}

#usr input:hover {
  opacity: 80%;
}

table {
  border-collapse: collapse;
  margin: 0 auto;
  background-color: snow;
}

th,
td {
  border: 1px solid black;
  padding: 0.5em;
  text-align: center;
}

tr:nth-child(odd) {
  background-color: whitesmoke;
}

#shop {
  box-sizing: border-box;
  text-align: center;
  margin: 0 auto;
  width: 50%;
}

#shop label {
  display: block;
  margin-bottom: 0.2em;
}

#shop input,
select,
#del,
#storage,
#finish {
  box-sizing: border-box;
  font-size: 1em;
  width: 60%;
  height: 2.5em;
  text-align: center;
  margin-bottom: 1.2em;
}

nav {
  text-align: center;
  margin: 0.5em 0;
}

#del {
  background-color: snow;
  margin: 0.5em auto;
  padding: 0.5em;
  width: 34%;
}

#crtTable {
  margin: 0.5em 0;
}
</style>