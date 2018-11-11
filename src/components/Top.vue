<template>
  <div class="top">
    <div id="WebGL-output"></div>
  </div>
</template>

<script>
import * as THREE from 'three';

export default {
  name: 'Top',
  data () {
    return {
      step: 0,
      step2: 0,
      pointCloud: null,
      spacePoint: null,
      camera: null,
      scene: null,
      canvasRenderer: null
    };
  },
  methods: {
    // 空間にパーティクル生成
    createPointsSpace: function () {
      let geom = new THREE.Geometry();
      let material = new THREE.PointsMaterial({
        transparent: true,
        map: this.generateSprite(),
        blending: THREE.AdditiveBlending,
        depthWrite: false
      });

      let range = 150;
      for (let i = 0; i < 7500; i++) {
        let particle = new THREE.Vector3(
          Math.random() * range - range / 2,
          Math.random() * range - range / 2,
          Math.random() * range - range / 2
        );
        geom.vertices.push(particle);
      }

      this.spacePoint = new THREE.Points(geom, material);
      this.spacePoint.name = 'particlesSpace';

      this.scene.add(this.spacePoint);
    },
    render () {
      // TWEEN.update();
      // console.log('this.spacePoint:', this.spacePoint.rotation.x);

      this.step += 0;
      this.step2 += 0.0002;

      this.spacePoint.rotation.x = this.step2;
      this.spacePoint.rotation.y = this.step2;
      this.spacePoint.rotation.z = this.step2;

      requestAnimationFrame(this.render);
      this.canvasRenderer.render(this.scene, this.camera);
    },
    // オレンジ色のグラデーションの円をつくる
    generateSprite () {
      let canvas = document.createElement('canvas');
      canvas.width = 16;
      canvas.height = 16;

      let context = canvas.getContext('2d');
      let gradient = context.createRadialGradient(canvas.width / 2, canvas.height / 2, 0, canvas.width / 2, canvas.height / 2, canvas.width / 2);
      gradient.addColorStop(0, 'rgba(255,255,255,1)');
      gradient.addColorStop(0.2, 'rgba(250,237,167,1)');
      gradient.addColorStop(0.4, 'rgba(225,109,25,1)');
      gradient.addColorStop(1, 'rgba(0,0,0,1)');

      context.fillStyle = gradient;
      context.fillRect(0, 0, canvas.width, canvas.height);

      let texture = new THREE.Texture(canvas);
      texture.needsUpdate = true;
      return texture;
    }
  },
  mounted () {
    console.log('mounted');
    // let loader = new THREE.TDSLoader();

    // シーンを生成 すべての物体・高原を保持して変更を監視するコンテナオブジェクト
    this.scene = new THREE.Scene();

    // カメラオブジェクト シーンを描画するときに何が見えるか
    this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);

    // レンダラーオブジェクト カメラオブジェクトの角度に基づいてブラウザ内でsceneオブジェクトがどのように見えるかを計算
    this.canvasRenderer = new THREE.WebGLRenderer();
    this.canvasRenderer.setClearColor(new THREE.Color(0x000000));
    this.canvasRenderer.setSize(window.innerWidth, window.innerHeight);

    // カメラのポジション
    this.camera.position.x = 0;
    this.camera.position.y = 0;
    this.camera.position.z = 100;
    this.camera.lookAt(new THREE.Vector3(0, 0, 0));

    document.getElementById('WebGL-output').appendChild(this.canvasRenderer.domElement);

    // スタート頂点位置
    // this.startPoints();
    // 空間にパーティクル
    this.createPointsSpace();
    this.render();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.top {
  width: 100%;
  height: 100%;
}
#WebGL-output {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}
</style>
