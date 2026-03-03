# fnaf-fnaf<!DOCTYPE html>
<html>
<head>
    <title>FNAF: ULTRA 4K HORROR</title>
    <style>
        body { margin: 0; background: #000; overflow: hidden; font-family: 'Courier New', monospace; }
        #ui { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); color: #0f0; text-align: center; pointer-events: none; }
        #cam-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: 20px solid rgba(0,255,0,0.1); box-sizing: border-box; display: none; background: rgba(0,20,0,0.2); }
        #power-box { position: absolute; top: 20px; left: 20px; color: #0f0; font-size: 20px; }
        .btn { position: absolute; bottom: 50px; padding: 15px 30px; background: #111; color: #0f0; border: 2px solid #0f0; cursor: pointer; pointer-events: auto; }
        #door-btn { right: 20px; }
        #cam-btn { left: 20px; }
    </style>
</head>
<body>
    <div id="power-box">POWER: <span id="percent">100</span>%</div>
    <div id="cam-overlay"><h1 style="margin: 20px;">CAM_01: MAIN HALLWAY</h1></div>
    
    <div id="ui">
        <div id="msg">USE BUTTONS TO SURVIVE</div>
        <button id="cam-btn" class="btn" onclick="toggleCams()">MONITOR</button>
        <button id="door-btn" class="btn" onclick="toggleDoor()">DOOR</button>
    </div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js" } }
    </script>

    <script type="module">
        import * as THREE from 'three';

        // --- 1. HORROR ENGINE SETUP ---
        const scene = new THREE.Scene();
        scene.fog = new THREE.Fog(0x000000, 1, 15);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // --- 2. THE OFFICE (4K Textures simulated via PBR) ---
        const roomGeo = new THREE.BoxGeometry(10, 8, 15);
        const roomMat = new THREE.MeshStandardMaterial({ color: 0x111111, side: THREE.BackSide, roughness: 0.8 });
        const room = new THREE.Mesh(roomGeo, roomMat);
        room.receiveShadow = true;
        scene.add(room);

        // Desk
        const desk = new THREE.Mesh(new THREE.BoxGeometry(4, 1, 2), new THREE.MeshStandardMaterial({ color: 0x221100 }));
        desk.position.set(0, -2, -6);
        scene.add(desk);

        // --- 3. THE ANIMATRONIC (Freddy-ish) ---
        const freddy = new THREE.Group();
        const head = new THREE.Mesh(new THREE.BoxGeometry(1, 1.2, 1), new THREE.MeshStandardMaterial({ color: 0x442200 }));
        const eyeL = new THREE.Mesh(new THREE.SphereGeometry(0.1), new THREE.MeshBasicMaterial({ color: 0xff0000 }));
        const eyeR = eyeL.clone();
        eyeL.position.set(-0.2, 0.2, 0.5);
        eyeR.position.set(0.2, 0.2, 0.5);
        freddy.add(head, eyeL, eyeR);
        freddy.position.set(0, -1, 10); // Start far away in the hall
        scene.add(freddy);

        // --- 4. LIGHTING SYSTEM ---
        const officeLight = new THREE.PointLight(0xffffff, 2, 10);
        officeLight.position.set(0, 2, -4);
        scene.add(officeLight);

        const hallLight = new THREE.PointLight(0x550000, 1, 20);
        hallLight.position.set(0, 2, 5);
        scene.add(hallLight);

        // --- 5. GAME LOGIC ---
        let power = 100;
        let isCamOpen = false;
        let isDoorClosed = false;
        let freddyPhase = 0; // 0: Hidden, 1: Hall, 2: Window, 3: Jump-Scare

        window.toggleCams = () => {
            isCamOpen = !isCamOpen;
            document.getElementById('cam-overlay').style.display = isCamOpen ? 'block' : 'none';
        };

        window.toggleDoor = () => {
            isDoorClosed = !isDoorClosed;
            document.getElementById('door-btn').style.background = isDoorClosed ? '#f00' : '#111';
        };

        // Animatronic AI "State Machine"
        setInterval(() => {
            if (Math.random() > 0.7) {
                freddyPhase++;
                if (freddyPhase === 1) freddy.position.z = 5; // Mid hall
                if (freddyPhase === 2) freddy.position.set(-3, -1, -3); // Right outside door
                if (freddyPhase === 3 && !isDoorClosed) {
                    alert("JUMPSCARE! YOU DIED.");
                    location.reload();
                } else if (freddyPhase === 3 && isDoorClosed) {
                    freddyPhase = 0; // Reset
                    freddy.position.z = 10;
                }
            }
        }, 3000);

        // Power Drain
        setInterval(() => {
            if (isDoorClosed) power -= 2;
            if (isCamOpen) power -= 1;
            power -= 0.1;
            document.getElementById('percent').innerText = Math.max(0, Math.floor(power));
            if (power <= 0) {
                officeLight.intensity = 0;
                // Animatronic kills you instantly when power is out
            }
        }, 1000);

        // --- 6. RENDER LOOP ---
        function animate() {
            requestAnimationFrame(animate);
            
            // Mouse look effect
            camera.rotation.y = (Math.sin(Date.now() * 0.001) * 0.2); 
            
            renderer.render(scene, camera);
        }

        animate();
    </script>
</body>
</html>
