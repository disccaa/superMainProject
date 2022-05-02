<template>
  <div>
    <h1>Products</h1>
    <input type="text" v-model="searchString" @input="pushQueryToUrl({ search: searchString })">
    <div v-for="c in categories.data" :key="c.id">
      <button @click="pushQueryToUrl({ categoryId: c.id })">{{ c.title }}</button>
    </div>
    <select name="sort" v-model="sort" @change="asd($event)">
      <option value="updatedAt:desc">Newest</option>
      <option value="price:asc">Price: Ascending</option>
      <option value="price:desc">Price: Descending</option>
    </select>
    <div v-for="p in products.data" :key="p.id">
      <p>{{ p.title }}</p>
      <p>{{ p.price }}</p>
    </div>
    <div v-for="i in products?.meta?.pagination?.pageCount" :key="i">
      <button @click="pushQueryToUrl({ page: i })">{{ i }}</button>
    </div>
  </div>
</template>

<script>
import { useECommerceStore } from "@/stores/e-commerce";
import { useRoute, useRouter } from "vue-router";
import { ref, onMounted, watchEffect } from "vue";
import { storeToRefs } from 'pinia'
import qs from 'qs'
export default {
  setup() {
    const eCommerceStore = useECommerceStore();
    const { products } = storeToRefs(eCommerceStore);
    const { categories } = storeToRefs(eCommerceStore);
    const route = useRoute();
    const router = useRouter();
    const sort = ref(route.query.sort || "updatedAt:desc");
    const searchString = ref('')
    let ezQuery = {};
    function asd(event) {
      pushQueryToUrl({ sort: event.target.value });
    }
    function pushQueryToUrl(queryParam) {
      Object.entries(queryParam).forEach(([key, value]) => {
        if (value) {
          ezQuery[key] = value;
        } else {
          ezQuery[key] = undefined;
        }
      });
      router.push({ query: ezQuery })
    }
    async function createProductQuery() {
      ezQuery = {
        page: route.query.page,
        gte: route.query.gte,
        lte: route.query.lte,
        sort: route.query.sort,
        categoryId: route.query.categoryId,
        search: route.query.search != '' || route.query.search ? route.query.search : undefined
      }
      const pagination = { page: route.query.page || 1, pageSize: 5 };
      const populate = ["category"];
      const sort = route.query.sort ? [route.query.sort] : ["updatedAt:desc"];
      const search = route.query.search != '' && route.query.search ? route.query.search : undefined
      const filters = {
        $and: [
          {
            price: {
              $gte: route.query.price_gte,
            },
          },
          {
            price: {
              $lte: route.query.price_lte,
            },
          },
          {
            category: {
              id: {
                $eq: route.query.categoryId || categories[0]?.id,
              }
            }
          },
          {
            
            title: {
              $startsWith: search,
            }
          }
        ],
      };
      const query = {
        populate,
        pagination,
        sort,
        filters,
        search,
      }
      await eCommerceStore.fetchCategories();
      await eCommerceStore.fetchProducts(query);
    }
    watchEffect(() => {
      createProductQuery();
    });
    onMounted(() => { });
    return {
      products,
      eCommerceStore,
      pushQueryToUrl,
      categories,
      sort,
      asd
    };
  },
  name: "Products",
};
</script>

<style scoped>
</style>