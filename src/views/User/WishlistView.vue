<template>
  <main-header />
  <sub-header
    heading="Wishlist"
    :subHeading="`${products.length} items in your wishlist`"
  />
  <section v-if="loading" class="cart-section">
    <div class="container">
      <section v-if="products.length">
        <div class="cart-details">
          <table class="min-w-full table-auto">
            <thead>
              <tr>
                <td class="text-left">Image</td>
                <td class="text-left">Product</td>
                <td class="text-left">Action</td>
                <td class="text-left">Remove</td>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in products" :key="item.id">
                <td class="text-left">
                  <img :src="item.image" :alt="item.brand" class="img" />
                </td>
                <td class="text-left">{{ item.name }}</td>
                <td class="text-left">
                  <button
                    class="text-white bg-black px-6 py-2 rounded shadow"
                    @click="$router.push('/cart/' + item.id)"
                  >
                    View Product
                  </button>
                </td>
                <td
                  class="py-2 text-center flex justify-center items-center cursor-pointer"
                  @click="deleteItem(index)"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-x-circle delete-btn mt-5"
                    viewBox="0 0 16 16"
                  >
                    <path
                      d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z"
                    />
                    <path
                      d="M4.646 4.646a.5.5 0 0 1 .708 0L8 7.293l2.646-2.647a.5.5 0 0 1 .708.708L8.707 8l2.647 2.646a.5.5 0 0 1-.708.708L8 8.707l-2.646 2.647a.5.5 0 0 1-.708-.708L7.293 8 4.646 5.354a.5.5 0 0 1 0-.708z"
                    />
                  </svg>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>
      <div class="no-cart mb-5" v-else>
        <div class="no-cart-text flex flex-col items-center">
          <img src="@/assets/images/empty-cart.svg" alt="empty-cart" />
          <h3>Your wishlist is empty!</h3>
          <p>Browse our shop and discover our latest products.</p>
        </div>
        <router-link class="mb-5" to="/shop">
          <action-button btnvalue="Start Shopping" />
        </router-link>
      </div>
    </div>
  </section>

  <product-preloader v-else type="inline">Loading products...</product-preloader>

  <main-footer />
</template>

<script>
import SubHeader from "@/components/SubHeader.vue";
import ActionButton from "@/components/ActionButton.vue";
import MainHeader from "@/components/MainHeader.vue";
import MainFooter from "@/components/MainFooter.vue";
import ProductPreloader from "@/components/preloaders/ProductPreloader.vue";
import { mapGetters } from "vuex";
import axios from "@/axios";

export default {
  name: "WishlistView",
  components: {
    SubHeader,
    ActionButton,
    MainHeader,
    MainFooter,
    ProductPreloader,
  },
  data() {
    return {
      products: [],
      loading: true,
    };
  },
  computed: {
    ...mapGetters(["getWishlist"]),
  },
  methods: {
    async getProduct(id) {
      try {
        const response = await axios.get(`/products/${id}`);
        if (!this.products.some((p) => p.id === response.data.id)) {
          this.products.push(response.data);
        }
      } catch (error) {
        console.error("Error fetching product:", error);
      }
    },
    async loadWishlistProducts() {
      this.loading = true;
      this.products = [];
      try {
        await Promise.all(this.getWishlist.map((id) => this.getProduct(id)));
      } catch (error) {
        console.error("Error loading wishlist:", error);
      } finally {
        this.loading = false;
      }
    },
    deleteItem(index) {
      // Assume getWishlist is a getter that returns a reactive array, so update it via Vuex action/mutation
      this.$store.commit("REMOVE_FROM_WISHLIST", index);
    },
  },
  mounted() {
    this.loadWishlistProducts();
  },
  watch: {
    getWishlist: {
      handler() {
        this.loadWishlistProducts();
      },
      deep: true,
    },
  },
};
</script>

<style scoped>
/* ...your existing styles remain unchanged... */
</style>
