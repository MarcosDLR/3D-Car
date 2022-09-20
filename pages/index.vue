<template>
  <div>
    <div class="session1">
      <v-container>
        <v-row class="mt-2">
          <v-col md="6">
            <h1 style="color: white">Three.js</h1>
          </v-col>
        </v-row>
      </v-container>
      <v-row class="mt-5" align="center" justify="center">
        <v-col md="7" class="carSession">
              <template v-if="loadCar">
                <div class="content">
                    <div class="circle"></div>
                </div>
              </template>
              <template v-if="!loadCar">
                <v-progress-circular
                  :width="30"
                  color="red"
                  indeterminate
                ></v-progress-circular>
              </template>
              <div id="canvas"></div>
        </v-col>
      </v-row>
    </div>
  </div>
</template>

<script>
  import * as THREE from 'three';
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
  import { DirectionalLight,HemisphereLight,} from 'three';

  // import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
  export default {
    data(){
      return{
        model: null,
        controls: null,
        scene: null,
        camera: null,
        renderer: null,
        directionalLight: null,
        mouse: {x: 0,y: 0},
        loadCar: false,
      }
    },
    watch: {
      model(){
        if(this.model){
          this.directionalLight = new DirectionalLight('red',10);
          this.directionalLight.position.set(0, 1, 0);
          this.directionalLight.target = this.model;
          this.directionalLight.castShadow = true;
          this.scene.add(this.directionalLight);

          let mouseGeometry = new THREE.SphereGeometry(0, 0, 0);
          let mouseMaterial = new THREE.MeshLambertMaterial({});
          let mouseMesh = new THREE.Mesh(mouseGeometry, mouseMaterial);

          mouseMesh.position.set(0, 0, 0);
          this.scene.add(mouseMesh);

          document.addEventListener('mousemove', this.onMouseMove, false);
        }
      },
    },
    async mounted(){
			this.scene = new THREE.Scene();
      // this.scene.background = new THREE.Color(0xffffff);
      this.scene.background = null;
      let canvas = document.getElementById('canvas');
      const { width, height } = canvas.getBoundingClientRect();
			this.camera = new THREE.PerspectiveCamera( 40, width / height, 1, 1500 );

			this.renderer = new THREE.WebGLRenderer({antialias: true, alpha: true});
      this.renderer.gammaOutput = true;
      this.renderer.setClearColor(0x010101, 0);
      this.renderer.setPixelRatio(window.devicePixelRatio);
      //this.renderer.setPixelRatio(2);
      this.renderer.setSize(width, height);
      this.renderer.physicallyCorrectLight = true;
      this.renderer.putputEncoding = THREE.sRGBEncoding;

      document.body.appendChild(this.renderer.domElement);

      window.addEventListener('resize', () => {
        let { width, height } = canvas.getBoundingClientRect();
        this.renderer.setSize(width, height);

        this.camera.aspect = width / height;

        this.camera.updateProjectionMatrix();

      });
      //this.controls = new OrbitControls(this.camera, this.renderer.domElement);
			this.renderer.setSize( width, height );
			document.querySelector('#canvas').appendChild( this.renderer.domElement );

      //lights
      const light = new THREE.AmbientLight( 0x404040, 150 ); // soft white light
      this.scene.add( light );

      const light2 = new HemisphereLight(0x404040, 0xFFFFFF, 0.5)
      this.scene.add(light2);

      const manager = new THREE.LoadingManager();
      const loader = new GLTFLoader(manager);

      manager.onLoad = () => {
        this.loadCar = true;
      }

      loader.load( 'car/scene.gltf', ( gltf ) => {
        this.model = gltf.scene;
        this.model.rotation.x += 0.30;
        this.scene.add( this.model );
        this.animate();
      },undefined, ( error ) => {

        console.log( error );

      } );

			this.camera.position.z = 6;
    },
    methods: {
      animate() {
				requestAnimationFrame( this.animate );
        if(this.model != null){
          this.model.rotation.y += -0.002;
        }
				//this.controls.update();
				this.renderer.render( this.scene, this.camera );
			},
      onMouseMove(event){
          event.preventDefault();
          this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
          this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

          var vector = new THREE.Vector3(this.mouse.x, this.mouse.y, 0.5);
          vector.unproject(this.camera);
          var dir = vector.sub(this.camera.position).normalize();
          var distance = -this.camera.position.z / dir.z;
          var pos = this.camera.position.clone().add(dir.multiplyScalar(distance));

          this.directionalLight.position.copy(new THREE.Vector3(pos.x, pos.y, pos.z + 2));
      },
    }
  }
</script>

<style lang="scss">
@font-face {
  font-family: Poppins-Regular;
  src: url('../static/fonts/Poppins-Regular.ttf');
}

#canvas {
  position: relative;
  outline: none;
  min-height: 70vh;
  min-width: 100%;
  z-index: 2;
}
.session1{
  background-color: black;
  min-height: 100vh;
  // background-attachment: fixed;
  // background-position: center;
  // background-repeat: no-repeat;
  // background-size: cover;
}
.session2{
  min-height: 650px;
}
.content{
  position: absolute;
  perspective: 700px;
  top: 0; left: 0; bottom: 0; right: 0;
  z-index: 1;
}
.circle{
  position: absolute;
  top: -30px; left: 20px; bottom: 0; right: 0;
  background-color: white;
  //background: conic-gradient(red, white);
  border-radius: 50%;
  max-width: 100%;
  transform: rotateX(70deg);
}
.leftColumn{
  position: absolute;
  width: 1px;
  height: 200px;
  background-color: white;
  top: 240px;
}
.rightColumn{
  position: absolute;
  width: 1px;
  height: 200px;
  background-color: white;
  top: 240px;
  right: -20px;
}
.carSession{
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.globesContent{
  max-width: 100%;
  min-width: 100%;
  min-height: 600px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  padding: 40px;
}
.globe{
  width: 400px;
  min-height: 50px;
  background-color: white;
  border-radius: 30px;
  border-top-left-radius: 0px;
  position: relative;
  margin-top: 20px;
  padding: 10px 0px 0px 15px;
  font-size: 1.3em;
  font-family: Poppins-Regular;

}
.globe::after{
  content: '';
  position: absolute;
  top: 0; left: -10px;
  width: 0;
  height: 0;
  border-right: 20px solid transparent;
  border-top: 20px solid white;
  border-left: 20px solid transparent;
  border-bottom: 20px solid transparent;
}
</style>