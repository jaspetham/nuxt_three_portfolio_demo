<template>
  <div>
    <canvas ref="canvas"></canvas>
    <div id="introContainer" class="absolute absolute-center text-white text-center uppercase max-w-2x w-full px-6">
      <h1 id="introTitle" class="font-space-mono tracking-wide text-xl opacity-0" style="transform:translateY(30px)">Jasper Tham</h1>
      <p id="introText" class="text-4xl opacity-0 font-exo" style="transform:translateY(30px)">One with an everlasting desire <br> for the unknown & untold</p>
      <a id="goView" class="font-space-mono border cursor-pointer px-4 py-2 rounded-lg text-sm mt-8 hover:bg-white hover:text-blue-600 inline-block opacity-0" style="transform:translateY(30px)">View Work</a>
    </div>
  </div>
</template>

<script>
  import { PlaneBufferGeometry, BufferAttribute, Scene, Raycaster, PerspectiveCamera, WebGLRenderer, MeshPhongMaterial, DoubleSide, FlatShading, Mesh, BufferGeometry, PointsMaterial, Points, DirectionalLight } from 'three'
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
  import gsap from 'gsap'
  export default {
    mounted(){
      // GUI
      const dat = require('dat.gui')
      const gui = new dat.GUI()
      gui.destroy()
      const world = {
        plane:{
          width: 400,
          height: 400,
          widthSegments: 50,
          heightSegments:50
        }
      }
      gui.add(world.plane,'width').min(1).max(500).step(1).name('PlaneWidth').onChange(generatePlane)
      gui.add(world.plane,'height').min(1).max(500).step(1).name('PlaneHeight').onChange(generatePlane)
      gui.add(world.plane,'widthSegments').min(1).max(100).step(1).name('PlaneWidthSegments').onChange(generatePlane)
      gui.add(world.plane,'heightSegments').min(1).max(100).step(1).name('PlaneHeightSegments').onChange(generatePlane)

      function generatePlane(){
        planeMesh.geometry.dispose()
        planeMesh.geometry = new PlaneBufferGeometry(
          world.plane.width,
          world.plane.height,
          world.plane.widthSegments,
          world.plane.heightSegments
        )

        const {array} = planeMesh.geometry.attributes.position
        const randomValues = []
        for(let i = 0; i < array.length;i++){
          if( i % 3 === 0){
            const x = array[i]
            const y = array[i + 1]
            const z = array[i + 2]

            array[i] = x + (Math.random() - 0.5) * 3
            array[i + 1] = y + (Math.random() - 0.5) * 3
            array[i + 2] = z + (Math.random() - 0.5) * 3
          }
          randomValues.push(Math.random() * Math.PI * 2)
        }
        planeMesh.geometry.attributes.position.originalPosition = planeMesh.geometry.attributes.position.array
        planeMesh.geometry.attributes.position.randomValues = randomValues

        const colors = []
        for(let i = 0;i < planeMesh.geometry.attributes.position.count; i++){
          colors.push(0, 0.19, 0.4)
        }
        planeMesh.geometry.setAttribute('color', new BufferAttribute(new Float32Array(colors),3))
      }

      // resize
      addEventListener('resize', () => {
        camera.aspect = innerWidth/innerHeight
        camera.updateProjectionMatrix()
        renderer.setSize(innerWidth,innerHeight)
      })

      // Scene
      const scene = new Scene()

      // Raycaster
      const raycaster = new Raycaster()

      // Camera
      const cameraPosition = {
        fov: 75,
        size: innerWidth/innerHeight,
        near:0.1,
        far:1000
      }
      const camera = new PerspectiveCamera(cameraPosition.fov,cameraPosition.size,cameraPosition.near,cameraPosition.far)
      camera.position.z = 50
      
      // Renderer
      const renderer = new WebGLRenderer({
        canvas:this.$refs.canvas
      })
      renderer.setSize(innerWidth,innerHeight)

      // Controls
      const controls = new OrbitControls(camera,renderer.domElement)

      // Plane
      const planeGeometry = new PlaneBufferGeometry(world.plane.width,world.plane.height,world.plane.widthSegments,world.plane.heightSegments)
      const planeMaterial = new MeshPhongMaterial({
        side: DoubleSide,
        flatShading: FlatShading,
        vertexColors: true
      })
      const planeMesh = new Mesh(planeGeometry,planeMaterial)
      scene.add(planeMesh)
      generatePlane()

      // Stars
      const particleGeometry = new BufferGeometry()
      const count = 10000
      const positions = new Float32Array(count * 3)

      for(let i = 0;i < count * 3; i++){
        positions[i] = (Math.random() - 0.5) * 2000
      }
      particleGeometry.setAttribute('position',new BufferAttribute(positions,3))

      const particleMaterial = new PointsMaterial()
      particleMaterial.size = 0.3
      particleMaterial.sizeAttenuation = true

      const particles = new Points(particleGeometry,particleMaterial)
      // particles.position.y = 500
      scene.add(particles)

      // Light
      const light = new DirectionalLight(0xffffff,1,1)
      light.position.set(0,-1,1)
      scene.add(light)

      const backLight = new DirectionalLight(0xffffff,1,1)
      backLight.position.set(0,0,-1)
      scene.add(backLight)

      // Mouse coordinate
      const mouse = {
        x:undefined,
        y:undefined
      }

      let frame = 0
      // Animate
      function animate(){
        requestAnimationFrame(animate)
        // time
        frame += 0.01
        // Render
        renderer.render(scene, camera)
        // Raycaster
        raycaster.setFromCamera(mouse,camera)
        const intersects = raycaster.intersectObject(planeMesh)
        if(intersects.length>0){
          const intersectFace = intersects[0].face
          const intersectColor = intersects[0].object.geometry.attributes.color

          // hover color
          const initialColor = {
            r:0,
            g:0.19,
            b:0.4
          }

          const hoverColor = {
            r:0.1,
            g:0.5,
            b:1
          }
          gsap.to(hoverColor,{
            r:initialColor.r,
            g:initialColor.g,
            b:initialColor.b,
            onUpdate: () => {
              // vertice 1
              intersectColor.setX(intersectFace.a,hoverColor.r)
              intersectColor.setY(intersectFace.a,hoverColor.g)
              intersectColor.setZ(intersectFace.a,hoverColor.b)

              // vertice 2
              intersectColor.setX(intersectFace.b,hoverColor.r)
              intersectColor.setY(intersectFace.b,hoverColor.g)
              intersectColor.setZ(intersectFace.b,hoverColor.b)

              // vertice 3
              intersectColor.setX(intersectFace.c,hoverColor.r)
              intersectColor.setY(intersectFace.c,hoverColor.g)
              intersectColor.setZ(intersectFace.c,hoverColor.b)

              // update vertice
              intersectColor.needsUpdate = true
            }
          })
        }
        const { array, originalPosition, randomValues } = planeMesh.geometry.attributes.position
        // animate plane vertice
        for(let i = 0;i < array.length; i += 3){
          // x
          array[i] = originalPosition[i] + Math.cos(frame + randomValues[i]) * 0.005
          // y
          array[i + 1] = originalPosition[i + 1] + Math.sin(frame + randomValues[i + 1]) * 0.005

        }
        planeMesh.geometry.attributes.position.needsUpdate = true

        particles.rotation.x += 0.0005
        // animate camera

      }


      animate()


      addEventListener('mousemove', (event) => {
        mouse.x = (event.clientX / innerWidth) * 2 - 1
        mouse.y = - (event.clientY / innerHeight) * 2 + 1

      })

      gsap.to('#introTitle',{
        opacity:1,
        duration:1.5,
        y:0,
        ease:'expo'
      })
      gsap.to('#introText',{
        opacity:1,
        duration:1.5,
        delay:0.3,
        y:0,
        ease:'expo'
      })
      gsap.to('#goView',{
        opacity:1,
        duration:1.5,
        delay:0.6,
        y:0,
        ease:'expo'
      })

      document.querySelector('#goView').addEventListener('click', (e) =>{
        e.preventDefault()
        gsap.to('#introContainer',{
          opacity:0
        })
        gsap.to(camera.position,{
          z:25,
          ease:'power3.inOut',
          duration:1.5,
        })
        gsap.to(camera.rotation,{
          x:Math.PI * 0.5,
          ease:'power3.inOut',
          duration:1.5
        })
        gsap.to(camera.position,{
          y:1000,
          ease:'power3.in',
          duration:1,
          delay:2,
          onComplete:() => {
            this.$router.push('/work')
          }
        })
      })
    }
  }
</script>
