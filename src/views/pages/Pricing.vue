<template>
  <v-container
    id="pricing"
    class="fill-height justify-center"
    tag="section"
  >
    <ais-instant-search :search-client="searchClient" index-name="brand">
      <ais-search-box />
      <ais-hits>
        <div slot="item" slot-scope="{ item }">
          <h2>{{ item.name }}</h2>
        </div>
      </ais-hits>
    </ais-instant-search>
  </v-container>
</template>

<script>
  import algoliasearch from 'algoliasearch/lite';
  import { AisInstantSearch, AisSearchBox } from 'vue-instantsearch';
  import 'instantsearch.css/themes/algolia-min.css';

  export default {
    name: 'PagesPricing',

    components: {
      AisInstantSearch,
      AisSearchBox,
    },

    data() {
      return {
        searchClient: algoliasearch(
           'B1G2GM9NG0',
          'aadef574be1f9252bb48d4ea09b5cfe5'
        ),
      };
    },
  }
</script>

<style>
body {
  font-family: sans-serif;
  padding: 1em;
}
</style>