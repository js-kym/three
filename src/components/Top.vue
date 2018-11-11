<template>
  <div class="top">
    <div id="WebGL-output"></div>
  </div>
</template>

<script>
import * as THREE from 'three';
import TWEEN from '@tweenjs/tween.js';

export default {
  name: 'Top',
  data () {
    return {
      particleNum: 10000,
      step: 0,
      step2: 0,
      pointCloud: null,
      spacePoint: null,
      camera: null,
      scene: null,
      canvasRenderer: null,
      loadedGeometry: null,
      endPoint: null,
      random: 0,
      tween: null
    };
  },
  computed: {
    // 分裂の計算
    randomSphere: function () {
      return function (rad) {
        // 単位球内
        // 0-2piまでの一様乱数
        let ganma = Math.random() * Math.PI * 2;
        // -1 - 1の一様乱数
        let z = Math.random() * 2 - 1;
        // 0 - 1の一様乱数
        let r = Math.random();

        let posX = rad * Math.cbrt(r) * Math.sqrt(1 - z * z) * Math.cos(ganma);
        let posY = rad * Math.cbrt(r) * Math.sqrt(1 - z * z) * Math.sin(ganma);
        let posZ = rad * Math.cbrt(r) * z;

        let pos = {
          x: posX,
          y: posY,
          z: posZ
        };

        return pos;
      };
    },
    // シャッフル
    shuffle: function () {
      return function (ary) {
        let tmpAry = [].concat(ary);
        let i = tmpAry.length;
        while (i > 0) {
          let j = Math.floor(Math.random() * i);
          let t = tmpAry[--i];
          tmpAry[i] = ary[j];
          tmpAry[j] = t;
        }
        return tmpAry;
      };
    }
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
      TWEEN.update();
      // console.log('this.spacePoint:', this.spacePoint.rotation.x);

      this.step += 0.0003;
      this.step2 += 0.0002;

      this.spacePoint.rotation.x = this.step2;
      this.spacePoint.rotation.y = this.step2;
      this.spacePoint.rotation.z = this.step2;

      this.pointCloud.rotation.x = this.step;
      this.pointCloud.rotation.y = -this.step;
      this.pointCloud.rotation.z = this.step;

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
    },
    // ランダムで最終位置決定
    setGeom () {
      if (this.random > 5) {
        this.random = 0;
      }
      console.log('random:', this.random);
      switch (this.random) {
        // case 0:
        //   // logo
        //   endPointLogo().then(function(result) {
        //       endPoint = result;
        //   });
        //   break;
        // case 1:
        //   // space
        //   endPoint = endPointSphereRandom(20);
        //   break;
        // case 2:
        //   endPoint = endPointSphereRandomSplit(10);
        //   break;
        // case 3:
        //   // sphere
        //   endPoint = endPointSphere(20);
        //   break;
        // case 4:
        //   // donus
        //   endPoint = endPointTorus(5);
        //   break;
        // default:
        //   // cube
        //   endPoint = endPointCube(20);
        //   break;
      }
      this.endPoint = this.endPointTorus(20);
    },
    // スタート地点空間
    startPoints: function () {
      console.log('startPoints');
      let geom = new THREE.Geometry();
      let material = new THREE.PointsMaterial({
        transparent: true,
        map: this.generateSprite(),
        blending: THREE.AdditiveBlending,
        depthWrite: false
      });

      let range = 150;
      for (let i = 0; i < this.particleNum; i++) {
        let particle = new THREE.Vector3(
          Math.random() * range - range / 2,
          Math.random() * range - range / 2,
          Math.random() * range - range / 2
        );
        geom.vertices.push(particle);
      }

      this.loadedGeometry = geom.clone();
      console.log('loadedGeometry:', this.loadedGeometry);

      this.pointCloud = new THREE.Points(geom, material);
      this.pointCloud.name = 'particles';

      this.scene.add(this.pointCloud);
    },
    // 最終地点(一様乱数生成)
    endPointSphereRandom: function (radius) {
      let list = [];
      for (let i = 0; i < this.particleNum; i++) {
        // 単位球内
        // 0-2piまでの一様乱数
        let ganma = Math.random() * Math.PI * 2;
        // -1 - 1の一様乱数
        let z = Math.random() * 2 - 1;
        // 0 - 1の一様乱数
        let r = Math.random();

        let posX = radius * Math.cbrt(r) * Math.sqrt(1 - z * z) * Math.cos(ganma);
        let posY = radius * Math.cbrt(r) * Math.sqrt(1 - z * z) * Math.sin(ganma);
        let posZ = radius * Math.cbrt(r) * z;
        list.push({
          x: posX,
          y: posY,
          z: posZ
        });
      }
      return list;
    },
    // 最終地点(一様乱数生成分裂)
    endPointSphereRandomSplit: function (radius) {
      // particleNumを100こにわけて分裂させる
      let list = [];
      let pos = {};
      for (let i = 0; i < this.particleNum; i++) {
        if (i < 1000) {
          pos = this.randomSphere(3);
          list.push({
            x: pos.x + 20,
            y: pos.y + 20,
            z: pos.z + 20
          });
        } else if (i < 2000) {
          pos = this.randomSphere(5);
          list.push({
            x: pos.x - 20,
            y: pos.y - 20,
            z: pos.z - 20
          });
        } else if (i < 3000) {
          pos = this.randomSphere(5);
          list.push({
            x: pos.x + 20,
            y: pos.y - 20,
            z: pos.z - 20
          });
        } else if (i < 4000) {
          pos = this.randomSphere(5);
          list.push({
            x: pos.x + 5,
            y: pos.y + 20,
            z: pos.z - 20
          });
        } else if (i < 5000) {
          pos = this.randomSphere(5);
          list.push({
            x: pos.x - 20,
            y: pos.y + 20,
            z: pos.z + 10
          });
        } else if (i < 6000) {
          pos = this.randomSphere(4);
          list.push({
            x: pos.x + 10,
            y: pos.y - 20,
            z: pos.z + 10
          });
        } else if (i < 7000) {
          pos = this.randomSphere(4);
          list.push({
            x: pos.x - 10,
            y: pos.y - 10,
            z: pos.z + 10
          });
        } else if (i < 8000) {
          pos = this.randomSphere(4);
          list.push({
            x: pos.x + 10,
            y: pos.y + 5,
            z: pos.z + 10
          });
        } else if (i < 9000) {
          pos = this.randomSphere(5);
          list.push({
            x: pos.x - 10,
            y: pos.y + 5,
            z: pos.z + 0
          });
        } else {
          pos = this.randomSphere(4);
          list.push({
            x: pos.x + 5,
            y: pos.y - 5,
            z: pos.z - 5
          });
        }
      }
      return list;
    },
    // 最終地点円
    endPointSphere: function (radius) {
      let endPosGeom = new THREE.SphereGeometry(radius, 50, 50);
      console.log('endPointSphere:', endPosGeom);
      // 頂点で足りない分を増やす
      for (let i = endPosGeom.vertices.length; i < this.particleNum; i++) {
        let range = 150;
        let particle = new THREE.Vector3(
          Math.random() * range - range / 2,
          Math.random() * range - range / 2,
          Math.random() * range - range / 2
        );
        endPosGeom.vertices.push(particle);
      }
      let list = this.shuffle(endPosGeom.vertices);
      return list;
    },
    // 最終地点ドーナツ
    endPointTorus: function (radius) {
      let endPosGeom = new THREE.TorusGeometry(radius, 10, 45, 45);
      console.log('endPointTorus:', endPosGeom);
      // 頂点で足りない分を増やす
      for (let i = endPosGeom.vertices.length; i < this.particleNum; i++) {
        let range = 150;
        let particle = new THREE.Vector3(
          Math.random() * range - range / 2,
          Math.random() * range - range / 2,
          Math.random() * range - range / 2
        );
        endPosGeom.vertices.push(particle);
      }
      let list = this.shuffle(endPosGeom.vertices);
      return list;
    },
    // 画面クリックイベント
    clickFunc: function () {
      document.getElementById('WebGL-output').removeEventListener('click', this.clickFunc);
      this.tween.start();
    }
  },
  mounted () {
    console.log('mounted');
    const that = this;
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

    // クリックで形を変える
    document.getElementById('WebGL-output').addEventListener('click', this.clickFunc);
    let tweenPos = {pos: 0};

    this.tween = new TWEEN.Tween(tweenPos)
      .to({pos: 1}, 3000) // Quartic.InOut: Circular.Out
      .easing(TWEEN.Easing.Quadratic.InOut)
      .onUpdate(function () {
        let count = 0;
        let pos = tweenPos.pos;

        let sita = Math.PI / 2 * pos;
        let sita2 = (Math.PI / 2 * pos) - (Math.PI / 2);

        that.loadedGeometry.vertices.forEach(function (e, index) {
          let posX = ((that.endPoint[index].x - e.x) * Math.sin(sita)) + e.x;
          let posY = ((that.endPoint[index].y - e.y) * Math.cos(sita2)) + e.y;
          let posZ = ((that.endPoint[index].z - e.z) * Math.cos(sita2)) + e.z;
          that.pointCloud.geometry.vertices[count++].set(posX, posY, posZ);
        });

        that.pointCloud.geometry.verticesNeedUpdate = true;
      })
      .onComplete(function () {
        console.log('this:', this);
        this.pos = 0;
        let list = [];
        that.loadedGeometry.vertices = [];
        // 参照渡しになってしまうのであらためて入れなおす
        that.pointCloud.geometry.vertices.forEach(function (v) {
          list.push({
            x: v.x,
            y: v.y,
            z: v.z
          });
        });
        that.loadedGeometry.vertices = list;
        that.random++;
        that.setGeom();
        console.log('onComplete loadedGeometry:', that.loadedGeometry);
        document.getElementById('WebGL-output').addEventListener('click', that.clickFunc);
      });

    // スタート頂点位置
    this.startPoints();
    // ゴール頂点位置
    this.setGeom();
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
