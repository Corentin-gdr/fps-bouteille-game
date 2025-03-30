# fps-bouteille-game
init();
animate();

function init() {
  // Création de la scène et de la caméra
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x87ceeb); // bleu ciel

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 1000);
  
  // Contrôles en mode FPS
  controls = new THREE.PointerLockControls(camera, document.body);
  document.addEventListener('click', () => {
    controls.lock();
  }, false);
  scene.add(controls.getObject());
  
  // Géométrie du sol
  const floorGeometry = new THREE.PlaneGeometry(1000, 1000, 10, 10);
  const floorMaterial = new THREE.MeshBasicMaterial({ color: 0x228B22 });
  const floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.rotation.x = -Math.PI / 2;
  scene.add(floor);
  
  // Prototype du perso : une bouteille cartoon (représentée par un cylindre avec des bras/jambes)
  const bottleGeometry = new THREE.CylinderGeometry(1, 1, 3, 32);
  const bottleMaterial = new THREE.MeshBasicMaterial({ color: 0xffc0cb });
  const bottle = new THREE.Mesh(bottleGeometry, bottleMaterial);
  bottle.position.set(0, 1.5, 0);
  scene.add(bottle);
  
  // Bras et jambes simplifiés (cubes attachés)
  const limbGeometry = new THREE.BoxGeometry(0.5, 1, 0.5);
  const limbMaterial = new THREE.MeshBasicMaterial({ color: 0xff69b4 });
  const leftArm = new THREE.Mesh(limbGeometry, limbMaterial);
  leftArm.position.set(-1, 1.5, 0);
  scene.add(leftArm);
  const rightArm = new THREE.Mesh(limbGeometry, limbMaterial);
  rightArm.position.set(1, 1.5, 0);
  scene.add(rightArm);
  const leftLeg = new THREE.Mesh(limbGeometry, limbMaterial);
  leftLeg.position.set(-0.5, 0.5, 0);
  scene.add(leftLeg);
  const rightLeg = new THREE.Mesh(limbGeometry, limbMaterial);
  rightLeg.position.set(0.5, 0.5, 0);
  scene.add(rightLeg);
  
  // Arme cartoon réaliste tenue par le personnage
  const weaponGeometry = new THREE.BoxGeometry(0.2, 0.2, 2);
  const weaponMaterial = new THREE.MeshBasicMaterial({ color: 0x333333 });
  const weapon = new THREE.Mesh(weaponGeometry, weaponMaterial);
  // Positionner l'arme dans le champ de vision (simulée comme un objet fixe attaché à la caméra)
  weapon.position.set(0.3, -0.3, -1);
  camera.add(weapon);

  // Cibles fixes à détruire
  for (let i = 0; i < 10; i++) {
    const targetGeometry = new THREE.BoxGeometry(1, 1, 1);
    const targetMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
    const target = new THREE.Mesh(targetGeometry, targetMaterial);
    target.position.set(Math.random()*50-25, 0.5, Math.random()*50-25);
    scene.add(target);
  }

  // Configuration du renderer
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  // Gestion des événements clavier
  const onKeyDown = function (event) {
    switch (event.code) {
      case 'KeyZ':
      case 'ArrowUp':
        moveForward = true;
        break;
      case 'KeyS':
      case 'ArrowDown':
        moveBackward = true;
        break;
      case 'KeyQ':
      case 'ArrowLeft':
        moveLeft = true;
        break;
      case 'KeyD':
      case 'ArrowRight':
        moveRight = true;
        break;
    }
  };

  const onKeyUp = function (event) {
    switch (event.code) {
      case 'KeyZ':
      case 'ArrowUp':
        moveForward = false;
        break;
      case 'KeyS':
      case 'ArrowDown':
        moveBackward = false;
        break;
      case 'KeyQ':
      case 'ArrowLeft':
        moveLeft = false;
        break;
      case 'KeyD':
      case 'ArrowRight':
        moveRight = false;
        break;
    }
  };

  document.addEventListener('keydown', onKeyDown, false);
  document.addEventListener('keyup', onKeyUp, false);
  
  // Gestion du clic pour tirer
  document.addEventListener('mousedown', (event) => {
    if (event.button === 0) {  // clic gauche
      tirer();
    }
  }, false);

  window.addEventListener('resize', onWindowResize, false);
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

// Fonction de tir (ici, création d'un rayon qui détecte les cibles)
function tirer() {
  const raycaster = new THREE.Raycaster();
  const mouse = new THREE.Vector2(0, 0);
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(scene.children, false);
  if (intersects.length > 0) {
    // On retire la première cible touchée (si c'est une cible rouge)
    const intersect = intersects[0].object;
    if (intersect.material && intersect.material.color.getHex() === 0xff0000) {
      scene.remove(intersect);
      console.log('Cible touchée !');
    }
  }
}

function animate() {
  requestAnimationFrame(animate);

  const time = performance.now();
  const delta = (time - prevTime) / 1000;

  velocity.x -= velocity.x * 10.0 * delta;
  velocity.z -= velocity.z * 10.0 * delta;

  const speed = 400.0;

  if (moveForward) velocity.z -= speed * delta;
  if (moveBackward) velocity.z += speed * delta;
  if (moveLeft) velocity.x -= speed * delta;
  if (moveRight) velocity.x += speed * delta;

  controls.moveRight(-velocity.x * delta);
  controls.moveForward(-velocity.z * delta);

  prevTime = time;

  renderer.render(scene, camera);
}
