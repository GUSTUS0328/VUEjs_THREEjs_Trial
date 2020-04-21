<template>
  <div id="solarBg">
    <ul id="starry-lite">
      <li>
        <p>太阳：</p>
        <img src="image/sun_min_bg.png" />
      </li>
      <li>
        <p>水星：</p>
        <img src="image/mercury_min_bg.png" />
      </li>
      <li>
        <p>金星：</p>
        <img src="image/venus_min_bg.png" />
      </li>
      <li>
        <p>地球：</p>
        <img src="image/earth_min_bg.png" />
      </li>
      <li>
        <p>火星：</p>
        <img src="image/spark_min_bg.png" />
      </li>
      <li>
        <p>木星：</p>
        <img src="image/jupiter_min_bg.png" />
      </li>
      <li>
        <p>土星：</p>
        <img src="image/saturn_min_bg.png" />
      </li>
      <li>
        <p>天王星：</p>
        <img src="image/uranus_min_bg.png" />
      </li>
      <li>
        <p>海王星：</p>
        <img src="image/neptune_min_bg.png" />
      </li>
    </ul>
    <div id="starry-frame" ref="starryFrame"></div>
  </div>
</template>

<script>
let THREE = require("three");
let OrbitControls = require("three-orbit-controls")(THREE);

export default {
  name: "SolarSystem",
  data() {
    return {
      // 渲染器
      renderer: null,
      // 相机
      camera: null,
      // 场景
      scene: null,
      // 中心球体（太阳）
      centerBall: null,
      // 中心球体轨道组合体（行星）
      starLites: [],
      // 包裹画布dom
      //dom: document.getElementById("starry-frame"),
      dom: null,
      // orbitcontrols对象参数
      orbitcontrols: null
    };
  },
  methods: {
    /**
     * 返回行星轨道的组合体
     * @param starLiteSize 行星的大小
     * @param starLiteRadius 行星的旋转半径
     * @param rotation 行星组合体的x,y,z三个方向的旋转角度
     * @param speed 行星运动速度
     * @param imgUrl 行星的贴图
     * @param scene 场景
     * @returns {{satellite: THREE.Mesh, speed: *}} 卫星组合对象;速度
     */
    initSatellite: function(
      starLiteSize,
      starLiteRadius,
      rotation,
      speed,
      imgUrl,
      scene
    ) {
      let track = new THREE.Mesh(
        new THREE.RingGeometry(starLiteRadius, starLiteRadius + 0.05, 50, 1),
        new THREE.MeshBasicMaterial()
      );
      let centerMesh = new THREE.Mesh(
        new THREE.SphereGeometry(1, 1, 1),
        new THREE.MeshLambertMaterial()
      ); //材质设定
      let starLite = new THREE.Sprite(
        new THREE.SpriteMaterial({
          //map: THREE.ImageUtils.loadTexture(imgUrl)
          map: new THREE.TextureLoader().load(imgUrl)
        })
      );
      starLite.scale.x = starLite.scale.y = starLite.scale.z = starLiteSize;
      starLite.position.set(starLiteRadius, 0, 0);

      let pivotPoint = new THREE.Object3D();
      pivotPoint.add(starLite);
      pivotPoint.add(track);
      centerMesh.add(pivotPoint);
      centerMesh.rotation.set(rotation.x, rotation.y, rotation.z);
      scene.add(centerMesh);
      return { starLite: centerMesh, speed: speed };
    },
    /**
     * 实现球体发光
     * @param color 颜色的r,g和b值,比如：“123,123,123”;
     * @returns {Element} 返回canvas对象
     */
    generateSprite: function(color) {
      let canvas = document.createElement("canvas");
      canvas.width = 16;
      canvas.height = 16;
      let context = canvas.getContext("2d");
      let gradient = context.createRadialGradient(
        canvas.width / 2,
        canvas.height / 2,
        0,
        canvas.width / 2,
        canvas.height / 2,
        canvas.width / 2
      );
      gradient.addColorStop(0, "rgba(" + color + ",1)");
      gradient.addColorStop(0.2, "rgba(" + color + ",1)");
      gradient.addColorStop(0.4, "rgba(" + color + ",.6)");
      gradient.addColorStop(1, "rgba(0,0,0,0)");
      context.fillStyle = gradient;
      context.fillRect(0, 0, canvas.width, canvas.height);
      return canvas;
    },
    // 执行函数
    render: function() {
      this.renderer.render(this.scene, this.camera);
      this.centerBall.rotation.y -= 0.01;
      for (let i = 0; i < this.starLites.length; i++) {
        this.starLites[i].starLite.rotation.z -= this.starLites[i].speed;
      }
      this.orbitcontrols.update();
      requestAnimationFrame(this.render);
    },
    // 初始化函数
    initThree: function() {
      this.dom = this.$refs.starryFrame;

      // 初始化场景
      this.scene = new THREE.Scene();
      // 初始化相机
      this.camera = new THREE.PerspectiveCamera(
        20,
        this.dom.clientWidth / window.innerHeight,
        1,
        1000
      );
      // 设置相机位置
      this.camera.position.set(0, 0, 500);
      this.renderer = new THREE.WebGLRenderer({
        alpha: true,
        antialias: true
      });
      // 设置窗口尺寸
      this.renderer.setSize(this.dom.clientWidth, window.innerHeight);
      // 初始化控制器
      // this.orbitcontrols = new THREE.OrbitControls(
      //   this.camera,
      //   this.renderer.domElement
      // );
      this.orbitcontrols = new OrbitControls(
        this.camera,
        this.renderer.domElement
      );
      this.dom.appendChild(this.renderer.domElement);
      // 定义太阳材质
      let sunTexture = new THREE.TextureLoader().load(
        "image/sun_bg.jpg",
        () => {
          this.renderer.render(this.scene, this.camera);
        },
        undefined
      );
      // let sunTexture = THREE.ImageUtils.loadTexture(
      //   "image/sun_bg.jpg",
      //   {},
      //   function() {
      //     this.renderer.render(this.scene, this.camera);
      //   }
      // );
      //console.log(sunTexture);
      // 太阳以及太阳材质设定
      this.centerBall = new THREE.Mesh(
        new THREE.SphereGeometry(30, 30, 30),
        new THREE.MeshBasicMaterial({
          map: sunTexture
        })
      );
      // 添加太阳发光效果
      let centerBallLite = new THREE.Sprite(
        new THREE.SpriteMaterial({
          map: new THREE.CanvasTexture(this.generateSprite("253,111,7")),
          blending: THREE.AdditiveBlending
        })
      );
      centerBallLite.scale.x = centerBallLite.scale.y = centerBallLite.scale.z = 90;
      this.scene.add(centerBallLite);
      this.scene.add(this.centerBall);
      //添加水星
      this.starLites.push(
        this.initSatellite(
          2,
          34,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.02,
          "image/mercury_bg.png",
          this.scene
        )
      );
      //添加金星
      this.starLites.push(
        this.initSatellite(
          3,
          38,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.018,
          "image/venus_bg.png",
          this.scene
        )
      );
      //添加地球
      this.starLites.push(
        this.initSatellite(
          3.2,
          42.2,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.016,
          "image/earth_bg.png",
          this.scene
        )
      );
      //添加火星
      this.starLites.push(
        this.initSatellite(
          2.5,
          47.1,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.014,
          "image/spark_bg.png",
          this.scene
        )
      );
      //添加木星
      this.starLites.push(
        this.initSatellite(
          35,
          71,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.012,
          "image/jupiter_bg.png",
          this.scene
        )
      );
      //添加土星
      this.starLites.push(
        this.initSatellite(
          45,
          110,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.01,
          "image/saturn_bg.png",
          this.scene
        )
      );
      //添加天王星
      this.starLites.push(
        this.initSatellite(
          17,
          158,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.008,
          "image/uranus_bg.png",
          this.scene
        )
      );
      //添加海王星
      this.starLites.push(
        this.initSatellite(
          15,
          188,
          { x: -Math.PI * 0.42, y: Math.PI * 0.02, z: 0 },
          0.006,
          "image/neptune_bg.png",
          this.scene
        )
      );
      this.render();
    },
    resizeHandler: function() {
      // 窗口resize事件
      // 重新初始化尺寸
      this.camera.aspect = this.dom.clientWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(this.dom.clientWidth, window.innerHeight);
    }
  },
  created() {
    window.addEventListener("resize", this.resizeHandler);
  },
  mounted() {
    this.initThree();
    //aceInitFun();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
html {
  margin: 0;
  padding: 0;
}

#solarBg {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  background: black;
  width: 100%;
  height: 100%;
  background: #000 url(../../example/solar-system/image/starry_sky_bg.jpg)
    no-repeat center center;
  overflow: hidden;
}

#starry-lite {
  list-style: none;
  position: fixed;
  left: 0;
  top: 0;
  width: 110px;
}

#starry-lite li {
  height: 35px;
  line-height: 35px;
}

#starry-lite li p {
  display: inline-block;
  font-size: 14px;
  width: 70px;
  text-align: right;
  color: #ffffff;
}

#starry-lite li img {
  width: 30px;
  height: 30px;
  vertical-align: middle;
}

.code-frame {
  position: fixed;
  right: -800px;
  top: 0;
  width: 800px;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  transition: all 0.3s;
}

.code-frame.on {
  right: 0;
  transition: all 0.3s;
}

.code-operate {
  position: absolute;
  width: 32px;
  height: 64px;
  line-height: 64px;
  text-align: center;
  font-weight: bold;
  font-size: 24px;
  color: #fff;
  left: -32px;
  top: 50%;
  margin-top: -32px;
  background: rgba(0, 0, 0, 0.5);
  cursor: pointer;
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
}

.code-content {
  width: 100%;
  height: 100%;
  overflow: auto;
}
#code-content {
  width: 100%;
  height: 100%;
}
.code-operate:hover {
  background: rgba(0, 0, 0, 0.8);
}
.ace-monokai {
  background-color: rgba(0, 0, 0, 0.5) !important;
}
.ace-monokai .ace_gutter {
  background: rgba(0, 0, 0, 0.7) !important;
}

.ace_fold-widget,
.ace_info,
.ace_error,
.ace_text-input,
.ace_warning {
  display: none !important;
}

.download-frame {
  position: absolute;
  top: 0;
  left: 50%;
  margin-left: -80px;
  width: 160px;
  height: 50px;
  background: rgba(0, 0, 0, 0.5);
  cursor: pointer;
  color: #fff;
  font-size: 16px;
  line-height: 50px;
  text-align: center;
  border-bottom-left-radius: 5px;
  border-bottom-right-radius: 5px;
}

.download-frame:hover {
  background: rgba(0, 0, 0, 0.8);
  color: #0268ca;
}

#starry-frame {
  width: 100%;
  height: 100%;
}
</style>
