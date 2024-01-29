<template>
  <div class="flex justify-center h-screen w-full">
    <!-- <header class="text-center text-2xl font-sans pt-3 font-bold"> Single's 个人网站</header> -->
    <div class=" h-4/5 w-4/5 bg-white  mt-10 rounded-xl overflow-hidden shadow-2xl">
      <aside class="flex flex-col items-center  pt-2  w-36 bg-pink-300 h-full">
        <img src="../assets/images/avatar.jpg" class="w-24 h-24 bg-pink-500 rounded-full"/>
        <div class="w-full">
          <div class="h-10 first:mt-6 hover:bg-slate-200 cursor-pointer w-full text-sm text-center leading-10 tracking-widest " v-for="(item,index) in setItemList" :key="index">{{ item.text }}</div>
        </div>
      </aside>
      <div class="h-full w-full bg-slate-300">123</div>
    </div>
    
    
    <footer class="flex p-3 pb-5 flex-col justify-center absolute bottom-0 w-full h-16 bg-black opacity-70 text-gray-200 text-center text-xs " >
    <div>Copyright © 2024 - All Singleeeee. All Rights Reserved.</div>
     <a class="hover:text-blue-400" href="http://beian.miit.gov.cn/" target="_blank" rel="noopener noreferrer">互联网ICP备案:粤ICP备2024172166</a>
    <div>作者：Single | 邮箱：1197826934@qq.com </div>
    </footer>
  </div>


</template>
<script lang="ts" setup >
import { ref } from 'vue'
const setItemList = ref([
  {
    text:'个人信息',
  },
  {
    text:'个人博客'
  },
  {
    text:'项目经历'
  },
  {
    text:'其他网页'
  },
  {
    text:'留言板'
  },
  {
    text:'关于'
  },
])

</script>