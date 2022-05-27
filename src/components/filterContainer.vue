<template>
  <div class="filter-container">
    <div v-for="(filter, index) in filters" 
      :key="'filter-' + index" 
      :class="{filter: index % 1 === 0, 'light-filter': index % 2 === 0}"
    >
    <!-- i have 2 different classes for light-filter and a normal one. Depending on the index it should have different classes -->
      <FilterChoice 
        v-for="data in filter"
        :key="data.key"
        @click="$emit('filter-chosen', index, data.key)"
        :text="data.text"
        :light="data.active"
      />
      <!-- here i send the index of the array of buttons
      then i send the data.key to let app.vue know what button was clicked -->
    </div>
  </div>
</template>

<script>
import FilterChoice from "./FilterChoice";

export default {
  name: "FilterContainer",
  props: {
    filters: Array
  },
  components: {
    FilterChoice,
  },
};
</script>

<style scoped>
.filter-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.light-background {
  background-color: #d9dbf1;
}
</style>
