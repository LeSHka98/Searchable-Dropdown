<template>
  <div>
    <h2>{{income}}</h2>

    <Dropdown 
      class="dropdownSelect"
      :items="items"
      :search="true"
      :multiple="true"
      :nullable="true" 
      v-model:choose="income"/>
      <!-- <Dropdown 
      :items="DATA_SET_2"   //   :items="load"
      :search="true"
      :multiple="true"
      :nullable="true"
      :display="displaySET2"  
      v-model:choose="income"/> -->
  </div>
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

  income = []
  items = ['сергей', 'ира', 'лёша', 'митя', 'андрей', 'петя',
  'слава','вася','настя','алла','коля','игорь','данила','матвей','наташа','василиса','егор','валера','костя','женя','никита']
  items2 = [1, 2, 3, 4, 5, 6,7,8,8,10,11,12,13,14,15,16,17,18,19,20]
  DATA_SET_2 = [
    { id: 203, name: 'Андрей Стаселько'},
    { id: 200, name: 'Валентина Гелясина'},
    { id: 185, name: 'Виктория Апетёнок'},
    { id: 192, name: 'Владислав Романов'},
    { id: 196, name: 'Елена Рыштакова'},
    { id: 347, name: 'Захар Журавлёв'},
    { id: 186, name: 'Ирина Толкачёва'},
    { id: 296, name: 'Наталья Нестерович'},
    { id: 188, name: 'Парфенова Раиса'},
    { id: 184, name: 'Татьяна Сарокина'},
    { id: 215, name: 'Андрей Степин'},
    { id: 216, name: 'Анна Сулима'},
    { id: 213, name: 'Артём Карпук'},
    { id: 261, name: 'Валерий Матусевич'},
    { id: 207, name: 'Дина Богатко'},
    { id: 210, name: 'Елена Кицило'},
    { id: 274, name: 'Елизавета Каленкович'},
    { id: 208, name: 'Ирина Кондрасюк'},
    { id: 211, name: 'Маргарита Четверик'},
    { id: 209, name: 'Наталья Друтько'}
];


  load( val: string): Promise<any> {  
    val = val?.toLowerCase()                       // DATA_SET_2 изменить по ситуации
    return new Promise((resolve) => resolve(this.DATA_SET_2.filter((item: any) => { 
      if(typeof(item) !== 'object') return String(item).toLowerCase().includes(val)
                // displaySET2 изменить по ситуации
      return this.displaySET2(item).toLowerCase().includes(val) }   )))
  }
  // displaySET2 изменить по ситуации
  displaySET2(item: {id:number, name: string}) {
    return item?.name
  }

  
}
</script>

<style lang="scss" scoped>
  * {
    padding: 0;
    margin: 0;
    font-family: sans-serif;
  }
  #app {
    padding: 10px;
  }

  .dropdownSelect {
    margin: 20px auto;
  }
</style>
