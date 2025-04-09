<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Freestyle Ski Game</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
  </style>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/examples/js/loaders/GLTFLoader.js"></script>
<script>
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x87ceeb);

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 2000);
  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(50, 100, 50);
  scene.add(light);
  scene.add(new THREE.AmbientLight(0x404040));

  const slope = new THREE.Mesh(
    new THREE.PlaneGeometry(200, 1000),
    new THREE.MeshLambertMaterial({ color: 0xffffff })
  );
  slope.rotation.x = -Math.PI / 2.8;
  slope.position.y = -20;
  scene.add(slope);

  const rail = new THREE.Mesh(
    new THREE.BoxGeometry(2, 0.5, 20),
    new THREE.MeshLambertMaterial({ color: 0x444444 })
  );
  rail.position.set(0, 2, 50);
  scene.add(rail);

  const rail2 = new THREE.Mesh(
    new THREE.BoxGeometry(2, 0.5, 20),
    new THREE.MeshLambertMaterial({ color: 0x555555 })
  );
  rail2.position.set(5, 2, 80);
  scene.add(rail2);

  const bigKicker = new THREE.Mesh(
    new THREE.BoxGeometry(15, 4, 10),
    new THREE.MeshLambertMaterial({ color: 0x999999 })
  );
  bigKicker.position.set(-10, 2, 100);
  bigKicker.rotation.x = -Math.PI / 14;
  scene.add(bigKicker);

  const box = new THREE.Mesh(
    new THREE.BoxGeometry(10, 0.5, 10),
    new THREE.MeshLambertMaterial({ color: 0x888888 })
  );
  box.position.set(20, 1.25, 130);
  scene.add(box);

  const trunk = new THREE.Mesh(
    new THREE.CylinderGeometry(0.5, 0.5, 4),
    new THREE.MeshLambertMaterial({ color: 0x8B4513 })
  );
  trunk.position.set(-15, 2, 150);
  scene.add(trunk);

  const leaves = new THREE.Mesh(
    new THREE.ConeGeometry(2.5, 6),
    new THREE.MeshLambertMaterial({ color: 0x228B22 })
  );
  leaves.position.set(-15, 6, 150);
  scene.add(leaves);

  let skier;
  const loader = new THREE.GLTFLoader();
  loader.load('https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/CesiumMan/glTF/CesiumMan.gltf', function(gltf) {
    skier = gltf.scene;
    skier.scale.set(2, 2, 2);
    skier.position.set(0, 1, 120);
    scene.add(skier);
  });

  const keys = {};
  window.addEventListener("keydown", e => keys[e.key] = true);
  window.addEventListener("keyup", e => keys[e.key] = false);

  let velocityY = 0;
  let isJumping = false;
  let isGrinding = false;
  const gravity = -0.01;
  let speedZ = -0.4;

  function animate() {
    requestAnimationFrame(animate);
    if (!skier) return;

    // Bodensteuerung
    if (keys["ArrowLeft"]) skier.position.x -= 0.5;
    if (keys["ArrowRight"]) skier.position.x += 0.5;
    if (keys["ArrowDown"]) speedZ = -0.2; else speedZ = -0.4; // bremsen
    if (keys["ArrowUp"]) skier.rotation.y += 0.02; // angeben (drehen)
    if (keys[" "] && !isJumping) {
      velocityY = 0.3;
      isJumping = true;
    }

    // Nosepress (Shift + Pfeil oben)
    if (keys["Shift"] && keys["ArrowUp"]) skier.rotation.x = -0.3;
    // Tailpress (Shift + Pfeil unten)
    else if (keys["Shift"] && keys["ArrowDown"]) skier.rotation.x = 0.3;
    else skier.rotation.x = 0;

    // In der Luft: Tricks
    if (isJumping) {
      if (keys["a"]) skier.rotation.y += 0.05; // Drehung links
      if (keys["d"]) skier.rotation.y -= 0.05; // Drehung rechts
      if (keys["ArrowUp"]) skier.rotation.x -= 0.05; // Frontflip
      if (keys["ArrowDown"]) skier.rotation.x += 0.05; // Backflip
      if (keys["ArrowLeft"]) skier.rotation.z += 0.05; // Lincon left
      if (keys["ArrowRight"]) skier.rotation.z -= 0.05; // Lincon right
      if (keys["Shift"]) skier.rotation.y += Math.sin(Date.now() * 0.01) * 0.05; // Shifty
    }

    velocityY += gravity;
    skier.position.y += velocityY;

    if (skier.position.y <= 1) {
      skier.position.y = 1;
      velocityY = 0;
      isJumping = false;
      isGrinding = false;
      skier.rotation.x = 0;
      skier.rotation.z = 0;
    }

    skier.position.z += speedZ;

    const onRail = (Math.abs(skier.position.x - rail.position.x) < 1.5 &&
                    Math.abs(skier.position.z - rail.position.z) < 10 &&
                    skier.position.y <= rail.position.y + 1) ||
                   (Math.abs(skier.position.x - rail2.position.x) < 1.5 &&
                    Math.abs(skier.position.z - rail2.position.z) < 10 &&
                    skier.position.y <= rail2.position.y + 1);

    if (onRail) {
      skier.position.y = 2.5;
      velocityY = 0;
      isJumping = false;
      isGrinding = true;
    } else {
      isGrinding = false;
    }

    if (isGrinding) {
      skier.rotation.z = Math.sin(Date.now() * 0.01) * 0.2;
    } else if (!isJumping) {
      skier.rotation.z = 0;
    }

    camera.position.set(skier.position.x, skier.position.y + 25, skier.position.z + 50);
    camera.lookAt(skier.position.x, skier.position.y, skier.position.z);

    renderer.render(scene, camera);
  }

  animate();
</script>
</body>
</html>
