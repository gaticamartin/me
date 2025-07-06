# Poemario digital

el dia de ayer tuve una de las angustias mas grande que he tenido el ultimo tiempo.
volviendo de comprar parafina me atacó algo muy raro y hasta ahora no se que fue.

es muy basico pero este tipo de emociones -lamentablemente quizas- son las que potencian de manera mas expotencial mis impulsos negativos.
me desperté hoy a las 12:00 con mucha hambre de hacer algo.

he aqui el resultado:

## idea inicial:

transcribir al codigo y a un sistema visual una serie de cartas y pensamientos acumulados en mi block de notas.

### Primera aproximacion

https://github.com/mortie/pixpeep

### Links

https://editor.p5js.org/gaticamartin
https://github.com/gaticamartin/me
https://toolsaday.com/text-tools/text-to-binary

## resultado

https://editor.p5js.org/gaticamartin/sketches/PBH6EdImj
fullscreen: https://editor.p5js.org/gaticamartin/full/PBH6EdImj

## proceso

![image](https://github.com/user-attachments/assets/d4c90765-fb76-4a44-80bc-007fa543150f)
https://toolsaday.com/text-tools/text-to-binary

guardar como .txt
![image](https://github.com/user-attachments/assets/7b9c246d-ed3d-449e-a3c8-c61733b64773)

### primer prueba de codigo asistido por chatGPT
![image](https://github.com/user-attachments/assets/3462f771-ba2b-4fc8-bd1d-1a3c2d77ed2e)
se interpreta binario en imagen animada por pixeles + audio
codigo: 

```
let binData = "";
let i = 0;
let audioOsc;
let playing = false;

function preload() {
  // Cargar un archivo de texto binario (0s y 1s)
  binData = loadStrings('archivo_binario.txt'); // Arrastra tu archivo aquí
}

function setup() {
  createCanvas(800, 800);
  frameRate(60);
  background(0);

  // Crear un oscilador para audio
  audioOsc = new p5.Oscillator('sine');
  audioOsc.amp(0.1);
}

function draw() {
  if (binData.length === 0) return;

  let line = binData[i % binData.length];
  for (let j = 0; j < line.length; j++) {
    let bit = line[j];
    let x = (i * 10 + j * 5) % width;
    let y = floor((i * 10 + j * 5) / width) % height;

    stroke(bit === '1' ? color(255, 0, 150) : color(0, 255, 100));
    point(x, y);
  }

  // Audio: cambiar frecuencia basado en bits
  if (playing && i < binData.length) {
    let val = parseInt(binData[i].replace(/[^01]/g, ""), 2);
    let freq = map(val % 256, 0, 255, 100, 2000);
    audioOsc.freq(freq);
  }

  i++;

  // Para reiniciar
  if (i >= binData.length) {
    noLoop();
    audioOsc.stop();
  }
}

function mousePressed() {
  if (!playing) {
    audioOsc.start();
    playing = true;
  } else {
    audioOsc.stop();
    playing = false;
  }
}

```

### Segundo intento de codigo asistido
sonido y imagen mas interesante
![image](https://github.com/user-attachments/assets/5083ceec-b715-42a6-b60e-09e69aac6161)
codigo:

```
    point(x, y);
  }

  // Audio dinámico
  if (playing) {
    let bitsSolo = line.replace(/[^01]/g, '');
    let val = parseInt(bitsSolo.slice(0, 8), 2); // toma solo los primeros 8 bits
    let freq = map(val, 0, 255, 100, 1500);
    audioOsc.freq(freq);
  }

  i++;
  if (i >= binData.length) {
    noLoop();
    audioOsc.stop();
  }
}

function mousePressed() {
  if (!playing) {
    audioOsc.start();
    playing = true;
  } else {
    audioOsc.stop();
    playing = false;
  }
}
```

### tercer codigo
modificaciones en lo visual y sonoro. se modifica apariencia de cada pixel interpretado y tambien se hace una busqueda en lo sonoro, mas similar al ruido digital tipo estatica
![image](https://github.com/user-attachments/assets/21f638c3-889a-4ea6-8540-97b5230cc4ec)
codigo:
```
let binData = [];
let i = 0;
let playing = false;
let noiseGen;

function preload() {
  binData = loadStrings('archivo_binario.txt'); // ← Asegúrate de subirlo al editor
}

function setup() {
  createCanvas(800, 800);
  frameRate(24);
  background(0);

  noiseGen = new p5.Noise('white');
  noiseGen.amp(0);
  noiseGen.start();
}

function draw() {
  background(0, 20); // efecto de rastro

  if (binData.length === 0) return;

  let line = binData[i % binData.length];

  let blockSize = 16; // tamaño más grande para que se noten los "pixeles"

  for (let j = 0; j < line.length; j++) {
    let bit = line[j];
    let x = (j * blockSize + frameCount * 3) % width;
    let y = (i * blockSize + sin(j * 0.5 + frameCount * 0.1) * 10) % height;

    let col;
    if (bit === '1') {
      col = color(
        200 + random(-55, 55),
        random(50, 150),
        random(150, 255),
        220
      );
    } else {
      col = color(
        random(50, 150),
        200 + random(-55, 55),
        random(50, 100),
        180
      );
    }

    noStroke();
    fill(col);
    rect(x, y, blockSize, blockSize);
  }

  // Sonido más agitado
  if (playing && frameCount % 2 === 0) {
    let energy = line.split('').filter(c => c === '1').length / line.length;
    noiseGen.amp(energy * 0.5, 0.05);
  }

  i++;
  if (i >= binData.length) {
    noLoop();
    noiseGen.amp(0, 0.5);
  }
}

function mousePressed() {
  playing = !playing;
  if (!playing) {
    noiseGen.amp(0, 0.2);
  }
}

```

### cuarto intento
correccion de lo visual nuevamente, el anterior se veía muy infantil o amigable.
se busca un lenguaje mas crudo y puramente computacional
![image](https://github.com/user-attachments/assets/c52a8571-8ab4-4fcb-b27e-7f6df3896b6b)
codigo:
```
let binData = [];
let i = 0;
let playing = false;
let noiseGen;
let blockSize = 12;

function preload() {
  binData = loadStrings('archivo_binario.txt');
}

function setup() {
  createCanvas(800, 800);
  frameRate(30);
  background(0);

  // Ruido digital
  noiseGen = new p5.Noise('white');
  noiseGen.amp(0);
  noiseGen.start();

  noSmooth(); // pixeles duros
}

function draw() {
  background(0); // limpio y duro, sin rastro

  if (binData.length === 0) return;

  let linesToDraw = 40;
  for (let l = 0; l < linesToDraw; l++) {
    let index = (i + l) % binData.length;
    let line = binData[index];

    for (let j = 0; j < line.length; j++) {
      let bit = line[j];
      let x = j * blockSize + floor(random(-1, 2)); // glitch sutil horizontal
      let y = l * blockSize;

      // Colores duros tipo terminal o glitch
      let col;
      if (bit === '1') {
        col = random() < 0.1
          ? color(255)             // glitch: blanco brillante
          : random() < 0.2
          ? color(255, 0, 0)       // glitch rojo
          : color(0, 255, 0);      // verde tipo terminal
      } else {
        col = color(0); // fondo negro
      }

      noStroke();
      fill(col);
      rect(x, y, blockSize, blockSize);
    }
  }

  // Sonido glitch digital (intensidad binaria)
  if (playing) {
    let line = binData[i % binData.length];
    let energy = line.split('').filter(c => c === '1').length / line.length;
    noiseGen.amp(energy * 0.4, 0.05);
  }

  i++;
  if (i >= binData.length) {
    noLoop();
    noiseGen.amp(0, 0.5);
  }
}

function mousePressed() {
  playing = !playing;
  if (!playing) {
    noiseGen.amp(0, 0.2);
  }
}
```

### quinto codigo
intento de llevar esto a algo mas similar a un crash con pantalla azul de error fatal
![image](https://github.com/user-attachments/assets/fda8b1c2-5036-49b2-8e64-39de33782d74)
codigo:
```
let binData = [];
let i = 0;
let playing = false;
let noiseGen;
let blockSize = 12;
let noiseTypes = ['white', 'pink', 'brown'];

function preload() {
  binData = loadStrings('archivo_binario.txt');
}

function setup() {
  createCanvas(800, 800);
  frameRate(24);
  background(0, 0, 170); // BSOD blue
  textFont('monospace');
  textSize(blockSize);
  noiseGen = new p5.Noise('white');
  noiseGen.amp(0);
  noiseGen.start();
}

function draw() {
  background(0, 0, 170); // BSOD blue cada frame

  if (binData.length === 0) return;

  let linesToDraw = 40;
  for (let l = 0; l < linesToDraw; l++) {
    let index = (i + l) % binData.length;
    let line = binData[index];

    // Introducir glitch visual (omite línea completa con probabilidad)
    if (random() < 0.05) continue;

    for (let j = 0; j < line.length; j++) {
      let bit = line[j];
      let x = j * blockSize;
      let y = l * blockSize + blockSize;

      // Glitch aleatorio: mostrar basura en vez del bit
      let char = bit;
      if (random() < 0.03) {
        char = random(['#', '%', '@', '&', '|', 'x']);
      }

      // Colores estilo BSOD glitch
      let col;
      if (bit === '1') {
        col = random() < 0.05 ? color(255, 0, 0) : color(200, 255, 255); // glitch rojo o cian pálido
      } else {
        col = color(240);
      }

      fill(col);
      noStroke();
      text(char, x, y);
    }
  }

  // Audio reactividad
  if (playing) {
    let line = binData[i % binData.length];
    let energy = line.split('').filter(c => c === '1').length / line.length;

    // Cambiar tipo de ruido aleatoriamente según energía
    if (random() < energy * 0.1) {
      let newType = random(noiseTypes);
      noiseGen.setType(newType);
    }

    // Volumen más reactivo y con jitter
    let amp = energy * 0.4 + random(-0.1, 0.1);
    noiseGen.amp(constrain(amp, 0, 0.5), 0.05);
  }

  i++;
  if (i >= binData.length) {
    noLoop();
    noiseGen.amp(0, 0.5);
  }
}

function mousePressed() {
  playing = !playing;
  if (!playing) {
    noiseGen.amp(0, 0.2);
  }
}
```
### sexto codigo
apariencia mas erratica mezclando entre ambos lenguajes visuales (binary y pixel)
![image](https://github.com/user-attachments/assets/6e3a6d6e-e8a0-49e2-97e9-2adc60b5d44a)

codigo:

```
 let binData = [];
let i = 0;
let playing = false;
let noiseGen;
let blockSize = 14;
let noiseTypes = ['white', 'pink', 'brown'];
let synth;

function preload() {
  binData = loadStrings('archivo_binario.txt');
}

function setup() {
  createCanvas(800, 800);
  frameRate(24);
  background(0, 0, 170); // BSOD blue

  textFont('monospace');
  textSize(blockSize);

  // Ruido
  noiseGen = new p5.Noise('white');
  noiseGen.amp(0);
  noiseGen.start();

  // Sintetizador para melodías glitch
  synth = new p5.MonoSynth();
}

function draw() {
  background(0, 0, 170); // Reset BSOD fondo

  if (binData.length === 0) return;

  let linesToDraw = 40;
  for (let l = 0; l < linesToDraw; l++) {
    let index = (i + l) % binData.length;
    let line = binData[index];

    // Modo visual aleatorio: texto o bloques
    let mode = random() < 0.75 ? 'text' : 'block';

    // Glitch visual: omitir líneas a veces
    if (random() < 0.04) continue;

    for (let j = 0; j < line.length; j++) {
      let bit = line[j];
      let x = j * blockSize;
      let y = l * blockSize + blockSize;

      // Colores
      let col;
      if (bit === '1') {
        col = random() < 0.05
          ? color(255, 0, 0)
          : random() < 0.1
          ? color(0, 255, 0) // verde consola
          : color(240);      // blanco/cian
      } else {
        col = color(0);
      }

      // Visualización
      if (mode === 'text') {
        let char = random() < 0.03 ? random(['#', '|', '%', '@']) : bit;
        fill(col);
        text(char, x, y);
      } else if (mode === 'block') {
        noStroke();
        fill(col);
        rect(x, y, blockSize, blockSize);
      }
    }
  }

  // Audio: ruido + notas digitales
  if (playing) {
    let line = binData[i % binData.length];
    let energy = line.split('').filter(c => c === '1').length / line.length;

    // Ruido base reactivo
    noiseGen.amp(constrain(energy * 0.5 + random(-0.1, 0.1), 0, 0.5), 0.05);

    // Aleatoriamente cambiar tipo de ruido
    if (random() < energy * 0.08) {
      noiseGen.setType(random(noiseTypes));
    }

    // MELODÍA: si la energía binaria es alta, dispara una nota aleatoria
    if (energy > 0.6 && random() < 0.1) {
      let note = random(['C4', 'D#4', 'F4', 'G#4', 'A3', 'B4']);
      let dur = 0.15;
      synth.play(note, 0.3, 0, dur);
    }
  }

  i++;
  if (i >= binData.length) {
    noLoop();
    noiseGen.amp(0, 0.5);
  }
}

function mousePressed() {
  playing = !playing;
  if (!playing) {
    noiseGen.amp(0, 0.2);
  }
}
```

### septimo codigo
- se cambia el archivo de texto -
se establece de que el codigo corre a la velocidad que se leeria el texto original (250 a 300 palabras por minuto)
asi mismo se añaden pausas entre cada uno de los 8 textos, tanto en el codigo como en el sonido.
se agrega un sistema que interpreta determinados patrones en el codigo binario como notas musicales aleatorias en escala mayor y menor
![image](https://github.com/user-attachments/assets/db558915-4c8d-477d-ab84-1c80d0e846b4)

```
let binData = [];
let i = 0;
let playing = false;
let noiseGen;
let blockSize = 16;
let noiseTypes = ['white', 'pink', 'brown'];
let synth;
let currentNote = '';
let showNoteTimer = 0;
let canvasWidth = 1890;
let canvasHeight = 1080;

let lastLineTime = 0;
let lineInterval = 350; // tiempo normal entre líneas (ms)
let pauseUntil = 0;

const majorScale = ['C4', 'D4', 'E4', 'F4', 'G4', 'A4', 'B4', 'C5'];
const minorScale = ['A3', 'B3', 'C4', 'D4', 'E4', 'F4', 'G4', 'A4'];

function preload() {
  binData = loadStrings('archivo_binario_con_pausas.txt');
}

function setup() {
  createCanvas(canvasWidth, canvasHeight);
  frameRate(30);
  background(0, 0, 170);
  textFont('monospace');
  textSize(blockSize);

  noiseGen = new p5.Noise('white');
  noiseGen.amp(0);
  noiseGen.start();

  synth = new p5.MonoSynth();
}

function draw() {
  background(0, 0, 170); // fondo BSOD

  if (binData.length === 0 || i >= binData.length) {
    noLoop();
    noiseGen.amp(0, 0.5);
    return;
  }

  // Si estamos en pausa, esperar
  if (millis() < pauseUntil) return;

  let linesToDraw = Math.floor(height / blockSize);
  for (let l = 0; l < linesToDraw; l++) {
    let index = (i + l) % binData.length;
    let line = binData[index];

    if (line === "PAUSE") continue; // no mostrar líneas de pausa

    if (random() < 0.04) continue;

    let mode = random() < 0.7 ? 'text' : 'block';

    for (let j = 0; j < line.length; j++) {
      let bit = line[j];
      let x = j * blockSize;
      let y = l * blockSize + blockSize;

      let col;
      if (bit === '1') {
        col = random() < 0.05
          ? color(255, 0, 0)
          : random() < 0.1
          ? color(0, 255, 0)
          : color(255);
      } else {
        col = color(0);
      }

      let codeChars = ['#', '/', '_', '%', '!', '&', '{', '}', '@', '|', '~'];
      let displayChar = bit;
      if (random() < 0.06) {
        displayChar = random(codeChars);
      }

      if (mode === 'text') {
        fill(col);
        text(displayChar, x, y);
      } else {
        noStroke();
        fill(col);
        rect(x, y, blockSize, blockSize);
      }
    }
  }

  // Scanlines
  stroke(0, 0, 0, 20);
  for (let y = 0; y < height; y += 4) {
    line(0, y, width, y);
  }

  // Audio
  if (playing && binData[i] !== "PAUSE") {
    let line = binData[i];
    let energy = line.split('').filter(c => c === '1').length / line.length;

    noiseGen.amp(constrain(energy * 0.5 + random(-0.1, 0.1), 0, 0.5), 0.05);

    if (random() < energy * 0.05) {
      noiseGen.setType(random(noiseTypes));
    }

    if (energy > 0.4 && random() < 0.2) {
      let scale = random() < 0.5 ? majorScale : minorScale;
      let note1 = random(scale);
      currentNote = note1;
      showNoteTimer = 15;
      synth.play(note1, 0.4, 0, 0.25);

      if (energy > 0.7 && random() < 0.5) {
        let idx = scale.indexOf(note1);
        let idx2 = (idx + 2) % scale.length;
        let note2 = scale[idx2];
        synth.play(note2, 0.25, 0.05, 0.2);
      }
    }
  }

  // Mostrar nota en pantalla
  if (showNoteTimer > 0) {
    fill(255, 255, 255, 180);
    textSize(48);
    textAlign(RIGHT, BOTTOM);
    text(currentNote, width - 20, height - 20);
    textSize(blockSize);
    textAlign(LEFT, TOP);
    showNoteTimer--;
  }

  // Avanzar línea con control de pausa
  if (millis() - lastLineTime > lineInterval) {
    if (binData[i] === "PAUSE") {
      pauseUntil = millis() + 1000; // pausa de 1 segundo
    }
    i++;
    lastLineTime = millis();
  }
}

function mousePressed() {
  playing = !playing;
  if (!playing) {
    noiseGen.amp(0, 0.2);
  }
}
```

## octavo codigo (FINAL)
ajustes de formato:1890 x 1080
se añade una pantalla y pausa inicial de 3 segundos.
se añade el texto otiginal censurado en un 90% en la esquina superior derecha
se agrega un indicador de la nota musical que se esta interpretando en la esquina inferior derecha
se agrega una fuente con este fin especifico
![image](https://github.com/user-attachments/assets/5ef30333-6095-42c6-8457-288236f500e6)

codigo:
```
let binData = [];
let censoredText = [];
let i = 0;
let textIndex = 0;

let playing = false;
let noiseGen;
let blockSize = 16;
let noiseTypes = ['white', 'pink', 'brown'];
let synth;
let currentNote = '';
let showNoteTimer = 0;

let customFont;
let monoFont;

let lastLineTime = 0;
let lineInterval = 350;
let pauseUntil = 0;

let intro = true;
let introStartTime;

const majorScale = ['C4', 'D4', 'E4', 'F4', 'G4', 'A4', 'B4', 'C5'];
const minorScale = ['A3', 'B3', 'C4', 'D4', 'E4', 'F4', 'G4', 'A4'];

function preload() {
  binData = loadStrings('archivo_binario_con_pausas.txt');
  censoredText = loadStrings('texto_censurado.txt');
  customFont = loadFont('PARCHM.TTF');
  monoFont = 'monospace';
}

function setup() {
  createCanvas(1890, 1080);
  frameRate(30);
  textFont(monoFont);
  textSize(blockSize);

  noiseGen = new p5.Noise('white');
  noiseGen.amp(0);
  noiseGen.start();

  synth = new p5.MonoSynth();
  introStartTime = millis();
}

function draw() {
  if (intro) {
    background(0, 0, 170);
    textFont(monoFont);
    textAlign(LEFT, TOP);
    fill(255);
    textSize(48);
    text("poemario", 32, 24);

    if (millis() - introStartTime > 3000) {
      intro = false;
      lastLineTime = millis();
    }
    return;
  }

  background(0, 0, 170);

  if (binData.length === 0 || i >= binData.length) {
    noLoop();
    noiseGen.amp(0, 0.5);
    return;
  }

  if (millis() < pauseUntil) return;

  // BINARIO GLITCH VISUAL
  textFont(monoFont);
  textSize(blockSize);
  let linesToDraw = Math.floor(height / blockSize);
  for (let l = 0; l < linesToDraw; l++) {
    let index = (i + l) % binData.length;
    let line = binData[index];

    if (line === "PAUSE") continue;
    if (random() < 0.04) continue;

    let mode = random() < 0.7 ? 'text' : 'block';

    for (let j = 0; j < line.length; j++) {
      let bit = line[j];
      let x = j * blockSize;
      let y = l * blockSize + blockSize;

      let col;
      if (bit === '1') {
        col = random() < 0.05 ? color(255, 0, 0)
             : random() < 0.1 ? color(0, 255, 0)
             : color(255);
      } else {
        col = color(0);
      }

      let codeChars = ['#', '/', '_', '%', '!', '&', '{', '}', '@', '|', '~'];
      let displayChar = bit;
      if (random() < 0.06) {
        displayChar = random(codeChars);
      }

      if (mode === 'text') {
        fill(col);
        text(displayChar, x, y);
      } else {
        noStroke();
        fill(col);
        rect(x, y, blockSize, blockSize);
      }
    }
  }

  // CENSORED TEXT BLOCK (TOP RIGHT)
  push();
  textFont(monoFont);
  textSize(16);
  fill(255);
  let xOffset = width - 500;
  let yOffset = 32;
  let visibleLines = 30;

  for (let j = 0; j < visibleLines; j++) {
    let idx = textIndex + j;
    if (idx < censoredText.length) {
      text(censoredText[idx], xOffset, yOffset + j * 20);
    }
  }
  pop();

  // Scanlines
  stroke(0, 0, 0, 20);
  for (let y = 0; y < height; y += 4) {
    line(0, y, width, y);
  }

  // Audio
  if (playing && binData[i] !== "PAUSE") {
    let line = binData[i];
    let energy = line.split('').filter(c => c === '1').length / line.length;

    noiseGen.amp(constrain(energy * 0.5 + random(-0.1, 0.1), 0, 0.5), 0.05);

    if (random() < energy * 0.05) {
      noiseGen.setType(random(noiseTypes));
    }

    if (energy > 0.4 && random() < 0.2) {
      let scale = random() < 0.5 ? majorScale : minorScale;
      let note1 = random(scale);
      currentNote = note1;
      showNoteTimer = 15;
      synth.play(note1, 0.4, 0, 0.25);

      if (energy > 0.7 && random() < 0.5) {
        let idx = scale.indexOf(note1);
        let idx2 = (idx + 2) % scale.length;
        let note2 = scale[idx2];
        synth.play(note2, 0.25, 0.05, 0.2);
      }
    }
  }

  // NOTA MUSICAL EN FUENTE PERSONALIZADA
  if (showNoteTimer > 0) {
    push();
    textFont(customFont);
    textSize(320);
    textAlign(RIGHT, BOTTOM);
    // Sombra pixelada blanca
    fill(255, 255, 255, 60);
    for (let dx = -2; dx <= 2; dx += 2) {
      for (let dy = -2; dy <= 2; dy += 2) {
        text(currentNote, width - 20 + dx, height - 20 + dy);
      }
    }
    // Texto principal
    fill(255);
    text(currentNote, width - 20, height - 20);
    pop();
    showNoteTimer--;
  }

  // Avanzar línea
  if (millis() - lastLineTime > lineInterval) {
    if (binData[i] === "PAUSE") {
      pauseUntil = millis() + 1000;
    }
    i++;
    textIndex++;
    lastLineTime = millis();
  }
}

function mousePressed() {
  playing = !playing;
  if (!playing) {
    noiseGen.amp(0, 0.2);
  }
}
```
[Poemario_2025_07_06_22_33_58.zip](https://github.com/user-attachments/files/21093383/Poemario_2025_07_06_22_33_58.zip)

