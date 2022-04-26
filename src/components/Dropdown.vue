<template>
  <div 
    class="select"
    :class="{select__active: open }"
    @click="openSelect">
    <span >{{value}}</span>
    <div
      v-if="!valarray.length" 
      class="select__arrow"
      :class="{select__arrow__active: open }"
      >^</div>
    <div
      v-else 
      class="select__cross"
      @click ="clear"
    >X</div>
    <div class="dropdown" v-if="open">
      <input v-if="search" class="dropdown__search" v-model="searchLine" type="text" placeholder="Search">
      <ul class="dropdown__list">
        <li 
          class="dropdown__item"
          :class="{dropdown__selected: this.valarray.includes(item)}"
          v-for="item of localItems" 
          :key="item"
          @click="mark(item)" 
         >
          <input v-if="multiple" class="dropdown__checkbox" type="checkbox" :value="item" :id="item" v-model="valarray">
          <label :for="item">{{ item }}</label>
        </li>
      </ul>
    </div>
  </div>
  {{valarray}}
</template>

<script lang="ts">
import {  Vue } from 'vue-class-component';
import { Prop, Watch } from 'vue-property-decorator'

export default class Dropdown extends Vue {
  @Prop() items!: any[]|((val: string) => Promise<any>)
  @Prop(Boolean) multiple: boolean | undefined
  @Prop(Boolean) search: boolean | undefined

  @Watch('searchLine')
  async onsearchLineChange(val: string) {
    if (typeof(this.items) !== 'function') this.localItems = this.items.filter(item => String(item).includes(val))
    else this.localItems = await this.items(val)
  }

  open = false 
  searchLine = ''
  value!: any | any[]
  valarray: any[] = []
  localItems: any[] = []

  mounted() {
    if(this.items.length > 20 && !this.search && typeof(this.items) !== 'function') { 
      this.items.length = 20 
      this.localItems = this.items
    }
    if (typeof(this.items) !== 'function') this.localItems = this.items
  }
  
  mark(item: any) {
    if (!this.multiple) {
      if(this.value !== item) this.value = item
      else this.value = null
    }
    else {
      console.log(this.valarray)
    }
  }

  openSelect(e: any) {
    if (!this.multiple) {
      if(e.target.className !== 'dropdown__search') this.open = !this.open
    }
    else if(e.target.className == 'select' || e.target.className == 'select select__active') this.open = !this.open
  }

  clear() {
    this.value = null
    this.valarray = []
  }
}
</script>

<style scoped lang="scss">
  .select {
    width: 300px;
    height: 34px;
    border: 1px solid #C4C4C4;
    border-radius: 5px;
    box-sizing: border-box;
    margin: auto;
    display: flex;
    align-items: center;
    position: relative;

    &__active {
      border: 1px solid #86D4E8;
    }
    span {
      margin-left: 7px;
    }
    &__arrow,
    &__cross {
      position: absolute;
      width: 12px;
      height: 5px;
      right: 10px;
      top: calc(50% - 7px/2 + 5.5px);
      color: #898989;
      transform: matrix(1, 0, 0, -1, 0, 0);
      cursor: pointer;
    }
      &__cross {
        top: calc(50% - 7px/2 + 8px);
      }

      &__active {
        transform: none;
        top: calc(50% - 5px);
      }
    }
  

  .dropdown {
    position: absolute;
    width: 300px;
    top: 40px;
    border: 1px solid #C4C4C4;
    border-radius: 5px;

    &__list {
      max-height: 200px;
      overflow-y: scroll;
      overflow-x: hidden;
    }

    &__search {
      width: 280px;
      height: 34px;
      margin: 10px;
      padding: 5px;
      border: 1px solid #C4C4C4;
      box-sizing: border-box;
      border-radius: 4px;
    }

    &__item {
      list-style: none;
      box-sizing: border-box;
      width: 294px;
      height: 40px;
      padding: 10px;
      display: flex;
      align-items: center;
    }
    &__checkbox {
      margin-right: 10px;;
    }
    &__selected {
      background: rgba(134, 212, 232, 0.5);
    }

    &__item:hover {
      background: rgba(134, 212, 232, 0.25);
    }
  }
</style>
