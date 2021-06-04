<template>
  <div 
    class="list-view" 
    ref="root"
    @scroll="handleScroll">
    <div
      class="list-view-phantom"       
      :style="{
        height: contentHeight + 'px'
      }">
    </div>
    <div
      ref="content"
      class="list-view-content">
      <div
        class="list-view-item"
        :style="{
          height: itemSizeGetter(item) + 'px'
        }"
        v-for="item in visibleData"
        :key="item">
        {{ item.value }}
      </div>
    </div>
  </div>
</template>
<script>
import { computed, nextTick, onMounted,reactive,ref } from 'vue'
export default {
  name:'ListView',
  props:{
    data:{
      type:Array,
      required:true
    },
    itemSizeGetter: {
      type: Function
    },
    estimatedItemSize: {
      type: Number,
      default: 30
    }

  },
  setup(props) {
    const root = ref(null) 
    const content = ref(null)
    const visibleData  = ref([])
    const startIndex = ref(0)
    let lastMeasuredIndex = -1
    const sizeAndOffsetCahce = {}  //缓存

    const contentHeight = computed(() => {
      const data = props.data
      const estimatedItemSize  = props.estimatedItemSize 
      let itemCount = data.length
      
      if(lastMeasuredIndex >= 0){
        const lastMeasuredSizeAndOffset  = getLastMeasuredSizeAndOffset()
        return lastMeasuredSizeAndOffset.offset + lastMeasuredSizeAndOffset.size + (itemCount - 1 - lastMeasuredIndex) * estimatedItemSize
      }else{
        return itemCount * estimatedItemSize
      }
    })

    onMounted(()=> {
      updateVisibleData()
    })


    async function updateVisibleData(scrollTop = 0){
      await nextTick()
      const start = findNearestItemIndex(scrollTop)
      const end = findNearestItemIndex(scrollTop + root.value.clientHeight )
      visibleData.value = props.data.slice(start,Math.min(end+1,props.data.length))
      content.value.style.webkitTransform  = `translate3d(0, ${ getItemSizeAndOffset(start).offset }px, 0)`

    }

    function handleScroll(){
      const scrollTop = root.value.scrollTop
      updateVisibleData(scrollTop)
    }



    function findNearestItemIndex(scrollTop){
      const lastMeasuredOffset  = getLastMeasuredSizeAndOffset().offset

      if(lastMeasuredOffset > scrollTop){
        return binarySearch(0, lastMeasuredIndex, scrollTop)
      }else{
        return exponentialSearch(scrollTop)
      }
    }
  
    function getItemSizeAndOffset(index){
      const data = props.data
      const itemSizeGetter  = props.itemSizeGetter

      if(lastMeasuredIndex >= index){
        return sizeAndOffsetCahce[index] || { offset: 0, size: 0 };
      }
      let offset = 0

      if(lastMeasuredIndex >= 0){
        const lastMeasured = sizeAndOffsetCahce[lastMeasuredIndex]
        if(lastMeasured){
          offset = lastMeasured.offset + lastMeasured.size
        }
      }
      for (let i = lastMeasuredIndex + 1; i <= index;i++){
        const item = data[i]
        const size = itemSizeGetter.call(null,item,i)
        sizeAndOffsetCahce[i] = {
          size,
          offset
        }
        offset += size
      }
      lastMeasuredIndex = Math.min(index, data.length - 1)
      return sizeAndOffsetCahce[index]
      
    }

    function getLastMeasuredSizeAndOffset(){
      return lastMeasuredIndex >= 0 ? sizeAndOffsetCahce[lastMeasuredIndex] : {offset:0 , size: 0}
    }

    function binarySearch(low, high, offset){
      let index;

      while (low <= high) {
        const middle = Math.floor((low + high) / 2)
        const middleOffset = getItemSizeAndOffset(middle).offset

        if(middleOffset === offset){
          index = middle
          break
        }else if(middleOffset > offset){
          high = middle - 1
        }else{
          low = middle + 1
        }
      }

      if(low > 0){
        index = low -1
      }

      if(typeof index === 'undefined'){
        index = 0
      }

      return index
    }

    function exponentialSearch(scrollTop){
      let bound = 1
      const data = props.data
      const start = lastMeasuredIndex >=0 ? lastMeasuredIndex : 0
      while (start + bound < data.length && getItemSizeAndOffset(start + bound).offset < scrollTop) {
        bound = bound * 2;
      }
      return binarySearch(start + Math.floor(bound / 2), Math.min(start + bound, data.length), scrollTop)
    }
    return{
      root,
      content,
      visibleData,
      contentHeight,
      handleScroll,
      itemSizeGetter:props.itemSizeGetter
    }
    
    
  },
}
</script>
<style lang="scss" scoped>
.list-view {
  height: 400px;
  overflow: auto;
  position: relative;
  border: 1px solid #aaa;
}

.list-view-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.list-view-content {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
}

.list-view-item {
  padding: 5px;
  color: #666;
  line-height: 30px;
  box-sizing: border-box;
}
</style>