## 这朵花是去年520之前写的，但是忙过头了，没有来得及发出来博客，就被我遗忘了，今年又到了这个日子，作为单身贵族的我，有必要把这朵花拿出来送给非单身的小哥哥们~~
## 话不多说了，直接上代码，我注释写得还是很详细的哦~~
## 希望路过的小哥哥们给我点点关注哇~~
## 我可是熬夜到凌晨一点多，给大家写得这个文章哦~~
## 这里补充一下我的代码地址吧
码云：
github:
还有我做的一些案例展示：（包括但不限于  自动漫游，手动控制键盘鼠标漫游，获取数据报警，鼠标滑过显示当前物体名称，等等功能）

## 碎碎念，随着three.js的版本升级，换汤不换药，需要修改的代码还是很少的，大家可以放心学习three.js哦~~
```
<!DOCTYPE html
  PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">

<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <title>一朵小花~~</title>
  <style>
    html,
    body {
      width: 100%;
      height: 100%;
      margin: 0;
      padding: 0;
      overflow: hidden;
    }

    #container {
      width: 100%;
      height: 100%;
    }
  </style>
</head>

<body>
  <div id="container"></div>
  <button onclick="getC()" style="position: absolute; top: 50px;">getC</button>
  <script src="js/three.min.js"></script>
  <script src="js/tween.js"></script>
  <script src="js/stats.min.js"></script>
  <script src="js/OrbitControls.js"></script>
  <script>
    // 需要用官方版本的tween.js
    let container = document.getElementById('container');
    let camera, scene, renderer;
    let cubeGroup, labelGroup = [];
    let stats, controls;
    let textureXDown = []
    let textureXDown02 = []
    let renderOrder = 1
    init();
    update();
    function init() {
      // scene
      scene = new THREE.Scene();

      // camera
      let frustumSize = 150;
      let aspect = container.clientWidth / container.clientHeight;
      camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        1,
        50000
      )
      camera.position.set(306, 1126, 7976);
      camera.lookAt(new THREE.Vector3(0, 0, 0));

      // renderer
      renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(window.devicePixelRatio);
      container.appendChild(renderer.domElement);

      renderer.render(scene, camera);



      addFlowers()
      stats = new Stats();
      container.appendChild(stats.dom);

      controls = new THREE.OrbitControls(camera, renderer.domElement);
      window.addEventListener('resize', onWindowResize, false);
    }

    function update() {
      requestAnimationFrame(update);
      controls.update();
      renderer.render(scene, camera);
      stats.update();

      textureXDown.forEach((_) => {
        _.offset.x -= 0.01
      })
      textureXDown02.forEach((_) => {
        _.offset.x -= 0.02
      })
    }

    function getC() {
      console.log(camera)
    }
    function onWindowResize() {
      let frustumSize = 200;
      let aspect = container.clientWidth / container.clientHeight;
      camera.left = frustumSize * aspect / -2;
      camera.right = frustumSize * aspect / 2;
      camera.top = frustumSize / 2;
      camera.bottom = frustumSize / -2;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function addFlowers() {
      const confArr = [
        { toy: 1200, color: 0x8087e6, segment: 5, h: 1500, radius: 2 },
        { toy: 800, color: 0x60acfc, segment: 20, h: 600, radius: 4 },
        { toy: 800, color: 0x7885f1, segment: 15, h: 50, radius: 5 },
        { toy: 600, color: 0x8087e6, segment: 10, h: 10, radius: 4 },
        { toy: 400, color: 0x27a1ea, segment: 13, h: 10, radius: 3 },
        { toy: 600, color: 0xfb9a8a, segment: 6, h: 10, radius: 2 },
      ]
      for (const item of confArr) {
        const arr = makeFlyLine({
          size: 1500,
          segment: item.segment,
          fromy: 80,
          toy: item.toy,
          position: { x: 0, y: 0, z: 0 },
          color: item.color,
        }) // 生成围绕一个中心向四周发散的飞线数组,
        // 按中心向四周发散的飞线数组坐标生成飞线--自定义飞线
        const { group: myGroup, texture: myTexture } = addFlyLine(
          arr,
          {
            url: './img/Trail.png',
            h: item.h,
            radius: item.radius,
          },
          renderOrder
        )
        myGroup.renderOrder = renderOrder++
        item.segment % 2 === 1
          ? textureXDown.push(myTexture)
          : textureXDown02.push(myTexture)
        scene.add(myGroup)
      }
    }
    function getTexture(url) {
      // 创建一个纹理
      let texture = new THREE.TextureLoader().load(url)
      // 设置uv重复包裹方式
      texture.wrapS = THREE.RepeatWrapping
      texture.wrapT = THREE.RepeatWrapping
      // 设置重复次数
      texture.repeat.set(1, 1)
      texture.needsUpdate = true
      return texture
    }
    /**
     * 创建一些贝塞尔曲线
     * @param {Object} Vector3Arr
     * @param {Object} conf 配置
     * @param {String} conf.url 曲线贴图
     * @param {Number} conf.h 控制曲线中间那个点的高度
     * @param {Number} conf.radius 控制曲线的粗细
     * @param {THREE.Group} group 群组
     */
    function addFlyLine(Vector3Arr, conf, renderOrder = 100) {
      let group = new THREE.Group()
      const texture = getTexture(conf.url)
      Vector3Arr.forEach((_) => {
        let fromV3 = _.from.clone().add(_.to.clone()).divideScalar(2)
        fromV3.y = conf.h // 曲线中间的那个点高度
        let quadraticBezier3 = new THREE.QuadraticBezierCurve3(_.from, fromV3, _.to) // 三维二次贝塞尔曲线起点,中间的控制点,终点
        let pointMeshArr = quadraticBezier3.getPoints(80)
        let pointArr = []
        pointMeshArr.forEach(function (point) {
          return pointArr.push([point.x, point.y, point.z])
        })
        let myMaterial = new THREE.MeshBasicMaterial({
          map: texture,
          side: THREE.BackSide,
          transparent: true,
          color: _.color,
        })
        let animateLine = createVector3Line({
          pointList: pointArr,
          material: myMaterial,
          number: 100,
          radius: conf.radius ? conf.radius : 1, // 控制管道的宽度
        })
        animateLine.renderOrder = renderOrder++
        group.add(animateLine)
      })
      return { group, texture }
    }
    /**
     * 由两点之间连线成贝塞尔曲线
     * @param {Object} option 配置项
     * @param {Object} option.number 管道面数
     * @param {Object} option.radius 管道粗细程度--直径
     */
    function createVector3Line(option) {
      const l = []
      option.pointList.forEach(e => l.push(new THREE.Vector3(e[0], e[1], e[2])))
      const curve = new THREE.CatmullRomCurve3(l) // 曲线路径
      const tubeGeometry = new THREE.TubeGeometry(
        curve,
        option.number || 50,
        option.radius || 1,
        option.radialSegments
      )
      return new THREE.Mesh(tubeGeometry, option.material)
    }
    /**
     * 生成围绕一个中心向四周发散的飞线数组
     * @param {Object} conf 配置项
     * @param {Number} conf.size 控制范围大小（圆形大小）
     * @param {Number} conf.segment 控制生成多少条线
     * @param {Number} conf.fromy 起点的位置的y高度
     * @param {Number} conf.toy 终点的位置的y高度
     * @param {Object} conf.position 位置
     * @param {String} conf.color 颜色
     */
    function makeFlyLine(conf) {
      const arr = getWorldPosition({
        size: conf.size,
        segment: conf.segment,
        position: conf.position,
      })
      console.log(arr)
      const vector3Arr = []
      arr.forEach((_, i) => {
        if (i > 0) {
          vector3Arr.push({
            from: new THREE.Vector3(
              arr[0].x,
              conf.fromy ? conf.fromy : 0,
              arr[0].y
            ),
            to: new THREE.Vector3(
              Math.floor(arr[i].x * 100) / 100,
              conf.toy ? conf.toy : 0,
              Math.floor(arr[i].y * 100) / 100
            ),
            color: conf.color,
          })
        }
      })
      return vector3Arr
    }
    /**
    * 获取正多边形每个顶点的坐标数组
    * 可以用于飞线制作
    * @param {Object} conf 配置项
    * @param {Number} conf.size 控制范围大小（圆形大小）
    * @param {Number} conf.segment 控制生成多少个点
    * @param {Object} conf.position 位置
    * @return {Array} worldPositionArr 多边形的顶点位置数组
    */
    function getWorldPosition(conf) {
      const worldPositionArr = []
      const geometry = new THREE.CircleGeometry(conf.size, conf.segment)
      var material = new THREE.MeshBasicMaterial({ color: 0xf37b1d })
      var vertexCoordinates = new THREE.Mesh(geometry, material)
      vertexCoordinates.position.set(
        conf.position && conf.position.x ? conf.position.x : 0,
        conf.position && conf.position.y ? conf.position.y : 0,
        conf.position && conf.position.z ? conf.position.z : 0
      )
      scene.updateMatrixWorld(true)
      var worldPosition = new THREE.Vector3()
      vertexCoordinates.getWorldPosition(worldPosition)
      vertexCoordinates.geometry.vertices.forEach((el) => {
        var vector = el.clone()
        vector.applyMatrix4(vertexCoordinates.matrixWorld)
        worldPositionArr.push(vector)
      })
      return worldPositionArr
    }
  </script>
</body>

</html>
```