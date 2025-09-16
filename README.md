<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>3D Car Game - Khronos Car</title>
  <style>
    body { margin: 0; overflow: hidden; background: #000; }
    canvas { display: block; }
    .controls {
      position: absolute;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      justify-content: center;
      z-index: 10;
    }
    .btn {
      width: 60px;
      height: 60px;
      border-radius: 50%;
      background: rgba(0,0,0,0.6);
      color: white;
      font-size: 22px;
      border: none;
      outline: none;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="controls">
    <button class="btn" id="left">⬅️</button>
    <button class="btn" id="up">⬆️</button>
    <button class="btn" id="down">⬇️</button>
    <button class="btn" id="right">➡️</button>
    <button class="btn" id="boost">🔥</button>
    <button class="btn" id="brake">🛑</button>
  </div>

  <!-- Three.js -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.0/examples/js/loaders/GLTFLoader.js"></script>

  <script>
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x87ceeb);

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    window.addEventListener("resize", () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // Ground
    const ground = new THREE.Mesh(
      new THREE.PlaneGeometry(400, 400),
      new THREE.MeshStandardMaterial({ color: 0x228B22 })
    );
    ground.rotation.x = -Math.PI / 2;
    scene.add(ground);

    // Road
    const road = new THREE.Mesh(
      new THREE.PlaneGeometry(20, 400),
      new THREE.MeshStandardMaterial({ color: 0x111111 })
    );
    road.rotation.x = -Math.PI / 2;
    road.position.y = 0.01;
    scene.add(road);

    // Lane markings
    const lineGeo = new THREE.PlaneGeometry(0.5, 5);
    const lineMat = new THREE.MeshStandardMaterial({ color: 0xffffff });
    for (let z = -200; z < 200; z += 10) {
      const line = new THREE.Mesh(lineGeo, lineMat);
      line.rotation.x = -Math.PI / 2;
      line.position.set(0, 0.02, z);
      scene.add(line);
    }

    // ✅ Load Khronos Car GLB model
    let car;
    const loader = new THREE.GLTFLoader();
    loader.load(
      "https://rawcdn.githack.com/KhronosGroup/glTF-Sample-Models/master/2.0/Car/glTF-Binary/Car.glb",
      (gltf) => {
        car = gltf.scene;
        car.scale.set(0.5, 0.5, 0.5);
        car.position.set(0, 0.5, 0);
        scene.add(car);
      },
      undefined,
      (err) => console.error("Car model load error:", err)
    );

    // AI Traffic Cars
    const trafficCars = [];
    function createTrafficCar(x, z, color) {
      const trafficGeo = new THREE.BoxGeometry(2, 1, 4);
      const trafficMat = new THREE.MeshStandardMaterial({ color });
      const traffic = new THREE.Mesh(trafficGeo, trafficMat);
      traffic.position.set(x, 0.5, z);
      scene.add(traffic);
      trafficCars.push(traffic);
    }
    for (let z = -50; z > -200; z -= 40) {
      createTrafficCar(4, z, 0x0000ff);
      createTrafficCar(-4, z - 20, 0xffff00);
    }

    // Lights
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0x404040));

    // Camera
    camera.position.set(0, 5, 10);

    // Controls
    let keys = {};
    let speed = 0;
    let maxSpeed = 0.5;
    let acceleration = 0.02;

    document.addEventListener("keydown", (e) => keys[e.key] = true);
    document.addEventListener("keyup", (e) => keys[e.key] = false);

    function bindButton(id, key) {
      const btn = document.getElementById(id);
      btn.addEventListener("touchstart", () => keys[key] = true);
      btn.addEventListener("touchend", () => keys[key] = false);
    }
    bindButton("up", "ArrowUp");
    bindButton("down", "ArrowDown");
    bindButton("left", "ArrowLeft");
    bindButton("right", "ArrowRight");
    bindButton("boost", "Boost");
    bindButton("brake", "Brake");

    // Game Loop
    function animate() {
      requestAnimationFrame(animate);

      if (car) {
        // Player Car Movement
        if (keys["ArrowUp"]) {
          speed += acceleration;
          if (speed > maxSpeed) speed = maxSpeed;
        } else if (keys["ArrowDown"]) {
          speed -= acceleration;
          if (speed < -maxSpeed / 2) speed = -maxSpeed / 2;
        } else {
          speed *= 0.95;
        }
        if (keys["Boost"]) speed = Math.min(speed + 0.1, maxSpeed * 2);
        if (keys["Brake"]) speed *= 0.8;

        if (keys["ArrowLeft"]) car.rotation.y += 0.05;
        if (keys["ArrowRight"]) car.rotation.y -= 0.05;

        car.position.x -= Math.sin(car.rotation.y) * speed;
        car.position.z -= Math.cos(car.rotation.y) * speed;

        // Camera follow
        camera.position.lerp(
          new THREE.Vector3(car.position.x, car.position.y + 5, car.position.z + 10),
          0.05
        );
        camera.lookAt(car.position);
      }

      // AI Traffic Cars move
      for (let traffic of trafficCars) {
        traffic.position.z += 0.2;
        if (traffic.position.z > 50) traffic.position.z = -200;
      }

      renderer.render(scene, camera);
    }
    animate();
  </script>
</body>
</html>
