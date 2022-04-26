<template>
  <Dropdown 
    :items="items"
    :search="true"
    :multiple="true"
    v-model="income"/>
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component';
import Dropdown from './components/Dropdown.vue';

@Options({
  components: {
    Dropdown
  },
})
export default class App extends Vue {
  items = ['мама', 'папа', 'лёша', 'митя', 'андрей', 'петя',
  'петя','петя','петя','петя','петя','петя','петя','петя','петя','петя','петя','петя','петя','петя','петя',]

  // items = [1, 2, 3, 4, 5, 6,7,8,8,10,11,12,13,14,15,16,17,18,19,20]
}
</script>

<style lang="scss">
  * {
    padding: 0;
    margin: 0;
    font-family: sans-serif;
  }
</style>
