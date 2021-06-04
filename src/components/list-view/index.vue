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
    }
  },
  setup(props) {
    const root = ref(null) 
    const content = ref(null)
    const visibleData  = ref([])
    let lastMeasuredIndex = -1
    const sizeAndOffsetCahce = {}  //缓存

    const contentHeight = computed(() => {
      const data = props.data
      const itemSizeGetter  = props.itemSizeGetter
      let total = 0
      for (let i = 0; i < data.length; i++) {
        total += itemSizeGetter.call(null,data[i],i)
      }
      return total
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
      const data = props.data

      let total = 0
      for (let i = 0, j = data.length; i < j; i++) {
        const size = getItemSizeAndOffset(i).size
        total += size
        if(total >= scrollTop || i === j -1){
          return i
        }
        
      }
      return 0
    }

    function getItemSizeAndOffset(index){
      const data = props.data
      const itemSizeGetter  = props.itemSizeGetter

      if(lastMeasuredIndex >= index){
        return sizeAndOffsetCahce[index]
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
      if(index > lastMeasuredIndex){
        lastMeasuredIndex = index
      }
      return sizeAndOffsetCahce[index]
      
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