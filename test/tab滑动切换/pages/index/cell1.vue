<template>
  <view @click="pressCell()" class="" style="display: flex;flex-direction: row;align-items: center;justify-content: center; background-color: #A0CFFF;width: 100%;height: 100%;">
	{{itemData.title}}
  </view>
</template>

<script>
  export default {
    props: {
      itemData: {
        type: Object,
        default: function(e) {
          return {}
        }
      }
    },
    methods: {
      pressCell(){
		  uni.showModal({
		  	title:"点击",
		  })
	  }
	  
    }
  }
</script>

<style scoped>
  
</style>
