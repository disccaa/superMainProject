<template>
  <div>
    <button @click="dsa">asdas</button>
    <h1>Products</h1>
    <input type="text" v-model="searchString" @input="pushQueryToUrl({ search: searchString })" />
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
      <p>{{ p.spec }}</p>
    </div>
    <div v-for="i in products?.meta?.pagination?.pageCount" :key="i">
      <button @click="pushQueryToUrl({ page: i })">{{ i }}</button>
    </div>
    <div>
      <input type="checkbox" value='wifi:5' @change="pushQueryToUrl({ spec: $event.target.value })">
      <input type="checkbox" value='let:true' @change="pushQueryToUrl({ spec: $event.target.value })">
      <input type="checkbox" value='lol:gol' @change="pushQueryToUrl({ spec: $event.target.value })">
      <input type="checkbox" value='lol1:gol1' @change="pushQueryToUrl({ spec: $event.target.value })">
      <input type="checkbox" value='lol2:gol2' @change="pushQueryToUrl({ spec: $event.target.value })">

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
    const spec = ref()
    let ezQuery = {};
    function asd(event) {
      pushQueryToUrl({ sort: event.target.value });
    }
    function pushQueryToUrl(queryParam) {
      Object.entries(queryParam).forEach(([key, value]) => {

        if (value) {
          if (key == "spec") {
            if (ezQuery[key] == undefined) {
              ezQuery[key] = value
              console.log(1)
              console.log('ezQK',ezQuery[key])
                  console.log('eqq1', ezQuery)
            } else {  
              console.log(11123, ezQuery[key])
              // console.log('lol',ezQuery[key].map(s => s.join(':')+ ',' + value))
              // ezQuery[key] = ezQuery[key]+ ',' + value;
              ezQuery[key] = ezQuery[key].map(s => s.join(':')) + ',' + value;
              
              // ezQuery[key] = queryParam.spec
              console.log(2)
              console.log( 'value',value)
              console.log('ezQuery', ezQuery)
              console.log('ezQK2',ezQuery[key])
            }


          } else {
            ezQuery[key] = value;
          }
        }
        else {
          ezQuery[key] = undefined;
        }
      });
      console.log('qparam', queryParam)

      router.push({ query: ezQuery })
    }
    async function createProductQuery() {
      console.log('nado',ezQuery.spec)
      ezQuery = {
        page: route.query.page,
        gte: route.query.gte,
        lte: route.query.lte,
        sort: route.query.sort,
        categoryId: route.query.categoryId,
        spec: route.query.spec ? route.query.spec.split(',').map(s => s.split(':')) : undefined,
        search: route.query.search != '' || route.query.search ? route.query.search : undefined
      }
      console.log('ezQ.spec',ezQuery.spec) 
      // console.log(ezQuery.spec)
      const pagination = { page: route.query.page || 1, pageSize: 5 };
      const populate = ["category"];
      const sort = route.query.sort ? [route.query.sort] : ["updatedAt:desc"];
      const search = route.query.search != '' && route.query.search ? route.query.search : undefined
      const spec = route.query.spec
      console.log(spec)

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
          },
        ],
      };
      if (ezQuery.spec) {
        ezQuery.spec.map(s => {
          filters.$and.push({
            spec: {
              $containsi: `"${s[0]}":${s[1]}`
            }
          })
        })
      }


      const query = {
        populate,
        pagination,
        sort,
        filters,
      }
      await eCommerceStore.fetchCategories();
      await eCommerceStore.fetchProducts(query);
      console.log(filters);
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
      asd,
      searchString
    };
  },
  name: "Products",
};
</script>

<style scoped>
</style>