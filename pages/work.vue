<template>
  <div class="bg-black h-screen px-6 py-12">
    <h2 class="text-white font-exo text-5xl uppercase opacity-0" ref="title" style="transform:translateY(30px);">my work</h2>
    <div class="flex space-x-2 mt-8">
      <div ref="projects" v-for="(project,index) in projects" :key="index" class="w-full opacity-0" style="transform:translateX(30px);">
        <nuxt-link to="/">
          <img :src="project.image.url" alt="" style="height:70vh" class="object-cover">
          <p class="text-white font-space-mono uppercase text-sm">{{project.title}}</p>
        </nuxt-link>
      </div>
    </div>
  </div>
</template>

<script>
  import gsap from 'gsap'
  import workImg from '~/assets/work1.jpg'
  import workImg2 from '~/assets/work2.jpg'
  import workImg3 from '~/assets/work3.jpg'
  export default{
    data(){
      return{
        projects:[
          {
            id:1,
            image: {
              url:workImg
            },
            title:'Shiny Stuff'
          },
          {
            id:2,
            image: {
              url:workImg2
            },
            title:'VR Stuff'
          },
          {
            id:3,
            image: {
              url:workImg3
            },
            title:'Computer'
          },
          {
            id:4,
            image: {
              url:workImg
            },
            title:'Shiny Stuff'
          },
          {
            id:5,
            image: {
              url:workImg2
            },
            title:'VR Stuff'
          },
          {
            id:6,
            image: {
              url:workImg3
            },
            title:'Computer'
          }
        ]
      }
    },
    mounted(){
      gsap.to(this.$refs.title,{
        opacity:1,
        duration:1.5,
        y:0,
        ease:'expo'
      })

      gsap.to(this.$refs.projects,{
        opacity:1,
        duration:2,
        stagger:0.1,
        x:0,
        ease:'expo',
        delay:0.3
      })
    }
  }
</script>
