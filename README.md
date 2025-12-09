<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>AI Avatar</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- Three.js -->
  <script src="https://unpkg.com/three@0.152.2/build/three.min.js"></script>
  <script src="https://unpkg.com/three@0.152.2/examples/js/loaders/GLTFLoader.js"></script>

  <style>
    html, body {
      margin: 0;
      padding: 0;
      background: black;
      overflow: hidden;
      width: 100%;
      height: 100%;
    }
    canvas {
      display: block;
    }
  </style>
</head>
<body>

<script>
  let scene, camera, renderer, avatar, mixer, clock;
  let mouthMesh = null;
  let headBone = null;

  init();

  function init() {
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x000000);

    camera = new THREE.PerspectiveCamera(35, window.innerWidth / window.innerHeight, 0.1, 100);
    camera.position.set(0, 1.6, 2.2);

    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const light = new THREE.HemisphereLight(0xffffff, 0x444444, 1.2);
    scene.add(light);

    const dirLight = new THREE.DirectionalLight(0xffffff, 1);
    dirLight.position.set(0, 2, 2);
    scene.add(dirLight);

    clock = new THREE.Clock();

    loadAvatar();
    animate();
  }

  function loadAvatar() {
    const loader = new THREE.GLTFLoader();

    // FREE human avatar (ReadyPlayerMe demo)
    loader.load(
      "https://models.readyplayer.me/64f2f8e1d8a8c0b0b4c5a7f4.glb",
      (gltf) => {
        avatar = gltf.scene;
        avatar.scale.set(1, 1, 1);
        scene.add(avatar);

        avatar.traverse((obj) => {
          if (obj.isMesh && obj.name.toLowerCase().includes("mouth")) {
            mouthMesh = obj;
          }
          if (obj.isBone && obj.name.toLowerCase().includes("head")) {
            headBone = obj;
          }
        });

        mixer = new THREE.AnimationMixer(avatar);
      }
    );
  }

  function animate() {
    requestAnimationFrame(animate);
    const delta = clock.getDelta();

    if (mixer) mixer.update(delta);

    // Idle head movement
    if (headBone) {
      headBone.rotation.y = Math.sin(Date.now() * 0.001) * 0.15;
    }

    renderer.render(scene, camera);
  }

  // Receive messages from React Native WebView
  window.addEventListener("message", (event) => {
    let data;
    try {
      data = JSON.parse(event.data);
    } catch {
      return;
    }

    if (data.type === "play_audio") {
      playAudio(data.audioUrl, data.emotion || "neutral");
    }
  });

  function playAudio(url, emotion) {
    const audio = new Audio(url);
    audio.play();

    const interval = setInterval(() => {
      if (!mouthMesh) return;
      mouthMesh.scale.y = 1 + Math.random() * 0.4;
    }, 80);

    audio.onended = () => {
      clearInterval(interval);
      if (mouthMesh) mouthMesh.scale.y = 1;
    };

    // Emotion reactions
    if (headBone) {
      if (emotion === "happy") headBone.rotation.x = -0.2;
      if (emotion === "serious") headBone.rotation.x = 0.1;
    }
  }

  window.addEventListener("resize", () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
</script>

</body>
</html>
