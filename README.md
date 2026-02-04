# Scaryyy
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Hotel Apocalypse - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: manipulation; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #000; font-family: 'Courier New', Courier, monospace; color: white; }

        /* Perspective & Running Effects */
        #game-view { position: relative; width: 100%; height: 100%; display: flex; flex-direction: column; justify-content: center; align-items: center; overflow: hidden; perspective: 1000px; }
        .running-bob { animation: bob 0.3s infinite linear; }
        @keyframes bob { 0%, 100% { transform: translateY(0) rotate(0deg); } 25% { transform: translateY(-10px) rotate(1deg); } 75% { transform: translateY(-10px) rotate(-1deg); } }

        /* 3D Hallway */
        #hallway {
            position: absolute; inset: 0; z-index: -1;
            background: linear-gradient(90deg, #0a0a0a 2%, transparent 20%, transparent 80%, #0a0a0a 98%),
                        repeating-linear-gradient(0deg, #000 0px, #000 40px, #111 40px, #111 80px);
            background-size: 100% 100%, 100% 400%;
        }

        /* UI & HUD */
        #hud { position: absolute; top: 7%; width: 100%; display: flex; justify-content: space-around; z-index: 100; font-size: 1.2rem; text-shadow: 0 0 10px red; }
        #heart-monitor { position: absolute; left: 20px; bottom: 20%; color: #ff0000; font-weight: bold; z-index: 100; }
        .heart-beat { animation: heart-pulse 0.8s infinite; }
        @keyframes heart-pulse { 0% { transform: scale(1); opacity: 1; } 50% { transform: scale(1.3); opacity: 0.7; } 100% { transform: scale(1); opacity: 1; } }

        /* Danger Vignette */
        #danger-vignette { position: fixed; inset: 0; pointer-events: none; z-index: 50; transition: box-shadow 0.3s; box-shadow: inset 0 0 100px rgba(0,0,0,1); }

        /* Doors */
        #door-container { display: flex; gap: 30px; transform: translateZ(-300px); transition: transform 0.4s ease-out; }
        .door {
            width: 90px; height: 160px; background: #2c1e14; border: 4px solid #1a110a;
            border-radius: 3px; display: flex; align-items: center; justify-content: center;
            font-size: 1.5rem; box-shadow: 0 0 30px #000; cursor: pointer;
        }

        /* Controls */
        #controls { position: absolute; bottom: 5%; width: 90%; display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 10px; z-index: 200; }
        .btn { background: rgba(20, 20, 20, 0.9); color: white; border: 1px solid #444; padding: 15px; border-radius: 8px; font-weight: bold; }
        .btn-run { grid-column: span 3; background: #600; padding: 25px; font-size: 1.2rem; box-shadow: 0 0 15px #300; }

        /* Ending Elements */
        #ending-screen { display: none; position: fixed; inset: 0; background: #000; z-index: 1000; flex-direction: column; justify-content: center; align-items: center; text-align: center; }
        .meteor { position: absolute; top: -300px; width: 200px; height: 200px; background: radial-gradient(circle, #fff, #ff0, #f00, #000); border-radius: 50%; box-shadow: 0 0 100px #ff4500; }
    </style>
</head>
<body>

<div id="danger-vignette"></div>

<audio id="bg-music" loop><source src="https://assets.mixkit.co/active_storage/sfx/123/123-preview.mp3" type="audio/mpeg"></audio>
<audio id="steps" loop><source src="https://assets.mixkit.co/active_storage/sfx/28/28-preview.mp3" type="audio/mpeg"></audio>
<audio id="heart-sound" loop><source src="https://assets.mixkit.co/active_storage/sfx/2180/2180-preview.mp3" type="audio/mpeg"></audio>

<div id="game-view">
    <div id="hallway"></div>
    
    <div id="hud">
        <div id="lives">HP: ❤️❤️❤️</div>
        <div id="stage">Floor 1/7</div>
    </div>

    <div id="heart-monitor" class="heart-beat">❤️ <span id="bpm">80</span> BPM</div>

    <div id="door-container">
        <div class="door" onclick="tapDoor(0)">I</div>
        <div class="door" onclick="tapDoor(1)">II</div>
        <div class="door" onclick="tapDoor(2)">III</div>
    </div>

    <div id="controls">
        <button class="btn" onclick="look('left')">LEFT</button>
        <button class="btn" onclick="lookBack()">LOOK BACK</button>
        <button class="btn" onclick="look('right')">RIGHT</button>
        <button class="btn btn-run" onmousedown="runStart()" onmouseup="runStop()" ontouchstart="runStart()" ontouchend="runStop()">HOLD TO RUN FORWARD</button>
    </div>
</div>

<div id="ending-screen">
    <div id="meteor" class="meteor"></div>
    <h1 id="victory-text" style="padding:20px;">SURVIVAL SECURED.</h1>
</div>

<script>
    let stage = 1, hp = 3, monsterCloseness = 0, runActive = false, hallwayOffset = 0;
    let correctDoor = Math.floor(Math.random() * 3);

    const hallway = document.getElementById('hallway');
    const view = document.getElementById('game-view');
    const bpmDisplay = document.getElementById('bpm');
    const vignette = document.getElementById('danger-vignette');
    const heartAudio = document.getElementById('heart-sound');

    function runStart() {
        runActive = true;
        view.classList.add('running-bob');
        document.getElementById('steps').play();
        document.getElementById('bg-music').play();
        heartAudio.play();
        animate();
    }

    function runStop() {
        runActive = false;
        view.classList.remove('running-bob');
        document.getElementById('steps').pause();
    }

    function animate() {
        if (!runActive) return;
        hallwayOffset += (10 + monsterCloseness * 2);
        hallway.style.backgroundPosition = `0px ${hallwayOffset}px`;
        requestAnimationFrame(animate);
    }

    function tapDoor(choice) {
        if (choice === correctDoor) {
            stage++;
            if (stage > 7) winSequence();
            else {
                document.getElementById('stage').innerText = `Floor ${stage}/7`;
                correctDoor = Math.floor(Math.random() * 3);
            }
        } else {
            hp--;
            monsterCloseness++;
            updateStatus();
            if (hp <= 0) caught();
        }
    }

    function lookBack() {
        monsterCloseness++;
        updateStatus();
        // Visual Scare
        vignette.style.background = "rgba(255,0,0,0.4)";
        setTimeout(() => vignette.style.background = "transparent", 200);
        if (monsterCloseness >= 6) caught();
    }

    function updateStatus() {
        document.getElementById('lives').innerText = "HP: " + "❤️".repeat(hp);
        const bpm = 80 + (monsterCloseness * 25);
        bpmDisplay.innerText = bpm;
        vignette.style.boxShadow = `inset 0 0 ${monsterCloseness * 50}px rgba(150,0,0,0.9)`;
        // Speed up heart animation
        document.querySelector('.heart-beat').style.animationDuration = `${1 - (monsterCloseness * 0.12)}s`;
    }

    function winSequence() {
        runStop();
        document.getElementById('game-view').innerHTML = `
            <div style="text-align:center">
                <h1>YOU FOUND THE CHEST</h1>
                <div style="font-size:80px; margin:20px; cursor:pointer;" onclick="defeatMonster()">🧳</div>
            </div>`;
    }

    function defeatMonster() {
        document.getElementById('game-view').innerHTML = `
            <div style="text-align:center; color:red;">
                <h1>MONSTER IN SIGHT</h1>
                <div style="font-size:100px;">👹</div>
                <button class="btn-run" onclick="finalBoom()">SHOOT</button>
            </div>`;
    }

    function finalBoom() {
        document.getElementById('game-view').style.display = 'none';
        const end = document.getElementById('ending-screen');
        end.style.display = 'flex';
        
        setTimeout(() => {
            const met = document.getElementById('meteor');
            met.style.transition = "top 1.2s cubic-bezier(0.6, -0.28, 0.735, 0.045)";
            met.style.top = "110%";
            setTimeout(() => {
                document.body.style.backgroundColor = "white";
                end.style.backgroundColor = "white";
                document.getElementById('victory-text').style.color = "black";
                document.getElementById('victory-text').innerHTML = "TO BE CONTINUED...";
            }, 1000);
        }, 3000);
    }

    function caught() {
        alert("HE CAUGHT YOU.");
        location.reload();
    }

    function look(dir) {
        const container = document.getElementById('door-container');
        if(dir === 'left') container.style.transform = "translateX(60px) translateZ(-300px)";
        else container.style.transform = "translateX(-60px) translateZ(-300px)";
        setTimeout(() => container.style.transform = "translateZ(-300px)", 300);
    }
</script>
</body>
</html>
