<template>
  <div 
      class="select"
      :class="{select__active: open, select__nullable: nullableMark }"
      v-click-outside="close"
      @click="openSelect">
    <span v-if="!display" class="select__result">
      {{result || resultArray[0] || (nullableMark ? "field can't be empty":'choose variants')}}
    </span>
    <span v-else class="select__result">
      {{display(result) || display(resultArray[0]) || (nullableMark ? "field can't be empty":'choose variants')}}
    </span>
    <div
      v-if="!resultArray.length || !open" 
      class="select__arrow"
      :class="{select__arrow__active: open }"
      >
        <svg width="13" height="7" viewBox="0 0 13 7" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M7.10793 6.76714C6.78181 7.07762 6.25307 7.07762 5.92696 6.76714L0.244587 1.35721C-0.0815288 1.04673 -0.0815289 0.543341 0.244586 0.232861C0.570703 -0.07762 1.09944 -0.0776205 1.42556 0.23286L6.50041 5.0644L11.5744 0.233644C11.9006 -0.0768371 12.4293 -0.0768371 12.7554 0.233644C13.0815 0.544124 13.0815 1.04751 12.7554 1.35799L7.23227 6.61633C7.1981 6.67005 7.15666 6.72075 7.10793 6.76714Z" fill="#898989"/>
        </svg>
      </div>
    <div
      v-else-if="open" 
      class="select__cross"
      @click ="clear"
    >
      <img src="@/assets/close.png" alt="close">
    </div>
    <div class="dropdown" v-if="open">
      <input v-if="search" class="dropdown__search" v-model="searchLine" type="text" placeholder="Search">
      <ul class="dropdown__list">
        <div v-if="!multiple">
          <li 
            class="dropdown__item"
            :class="{dropdown__selected: item == result}"
            v-for="item of localItems" 
            :key="item"
            @click="mark(item)" 
         >
         {{ display ? display(item) : item  }}
         </li>
        </div>
        <div v-else>
          <label v-for="item of localItems" :key="item" :for="item.id || item" class="dropdown__item"
                :class="{dropdown__selected: resultArray.includes(item)}">
            <input class="dropdown__checkbox" type="checkbox" :value="item" :id="item.id || item" v-model="resultArray">
            {{ display ? display(item) : item }}
          </label>
        </div>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component';
import { Prop, Watch } from 'vue-property-decorator'
import vClickOutside from 'click-outside-vue3'

@Options({
  directives: {
    clickOutside: vClickOutside.directive
  }
})
export default class Dropdown extends Vue {
  @Prop({ required: true }) items!: any[]|((val: string) => Promise<any>)
  @Prop({Boolean, required: true }) multiple!: boolean
  @Prop({Boolean, required: true }) search!: boolean 
  @Prop(Boolean) nullable: boolean | undefined
  @Prop() display: ((val: any) => string) | undefined

  open = false
  nullableMark = false 
  searchLine = ''
  result!: any 
  resultArray: any[] = []
  localItems: any[] = []

  @Watch('searchLine')
  async onsearchLineChange(val: string) {
    val = val?.toLowerCase()
    if (typeof(this.items) !== 'function') this.localItems = this.items.filter(item => {
      if(typeof(item) === 'object' && this.display) return this.display(item).toLowerCase().includes(val)
      return String(item).toLowerCase().includes(val)
    })
    else this.localItems = await this.items(val)
  } 
  @Watch('resultArray')
  onresultArrayChange(val:any) {
    if(typeof(val[0]) === 'object' ){
      this.$emit('update:choose', this.resultArray.map(item => item.id))
    }
    else this.$emit('update:choose', this.resultArray)
  }

  @Watch('open')  // пункт 4 - только отмеченные элементы после открытия
  async onOpenChange(val: boolean) {
    if (this.multiple) {
      this.markNull(val, this.resultArray.length)

      if (typeof(this.items) !== 'function') {
        if(val && this.resultArray.length > 0) this.localItems = this.items.filter(item => [...this.resultArray].includes(item))
        else this.localItems = [...this.items]
      }
      else {
        if(val && this.resultArray.length > 0) { 
          this.localItems = (await this.items('')).filter((item:any) => [...this.resultArray].includes(item))  }
        else this.localItems = await this.items('')
      }
    }
    else {
      this.markNull(val, this.result)
      if (typeof(this.items) !== 'function') {
        if(val && Boolean(this.result)) { 
          this.localItems.length = 0; 
          this.localItems[0] = this.items.find(item => this.result == item);
        }
        else  this.localItems = [...this.items]
      }
      else {
        if(val && Boolean(this.result)) { 
          this.localItems.length = 0; 
          this.localItems[0] = (await this.items('')).find((item:any) => this.result == item);
        }
        else  this.localItems = await this.items('')
      }
    }
    if(!val) this.searchLine = ''
  }

  mounted() {
    if(this.items.length > 20 && typeof(this.items) !== 'function') { 
      this.items.length = 20 
      this.localItems = [...this.items]
    }
    if (typeof(this.items) !== 'function' && typeof(this.items[0]) === 'object' && !this.display) {
      alert('For this type of data Display function should be provided') }
    else if(typeof(this.items) !== 'function' && typeof(this.items[0]) !== 'object' && this.display) {
      alert('For this type of data Display function should not be provided') }
  }
  
  markNull(val: boolean, result: any) {
    if(this.nullable && !val && !result) this.nullableMark = true
    else this.nullableMark = false
  }

  mark(item: any) {
    if(this.result !== item) this.result = item
    else this.result = null
    this.$emit('update:choose', this.result?.id || this.result)
  }

  openSelect(e: any) {
    if (!this.multiple) {
      if(e.target.className !== 'dropdown__search') this.open = !this.open
    }
    else if(Array.from(e.target.classList).includes('select')) this.open = !this.open
  }
  close() {
    return this.open = false
  }
  clear() {
    this.result = null
    this.resultArray = []
    if (typeof(this.items) !== 'function') this.localItems = [...this.items]
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
    display: flex;
    align-items: center;
    position: relative;
    cursor: pointer;

    &__active {
      border: 1px solid #86D4E8;
    }
    &__nullable {
      border: 1px solid red;
    }
    &__result {
      margin-left: 7px;
      color: #888888;
    }
    &__arrow,
    &__cross {
      position: absolute;
      width: 12px;
      height: 5px;
      right: 10px;
      top: calc(50% - 3.5px + 8.5px);
      color: #898989;
      transform: matrix(1, 0, 0, -1, 0, 0);
      
       &__active {
        transform: none;
        top: calc(50% - 9px);
      }
    }
      &__cross {
        top: calc(50% - 7px/2 + 8px);
        img {
          width: 13px;
        }
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
      padding: 0;
      margin: 0;
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
