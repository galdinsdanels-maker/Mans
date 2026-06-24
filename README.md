# Mans
Music notes
<!DOCTYPE html>
<html lang="lv">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Notu Atpazīšana</title>
<style>
    body {
        font-family: Arial, sans-serif;
        text-align: center;
        margin-top: 50px;
    }

    #note {
        font-size: 60px;
        font-weight: bold;
        color: #0077cc;
    }

    #history {
        margin-top: 20px;
        font-size: 24px;
        word-wrap: break-word;
    }
</style>
</head>
<body>

<h1>Notu Atpazīšana</h1>

<button id="startBtn">Sākt klausīties</button>

<div id="note">-</div>

<h3>Pierakstītās notis:</h3>
<div id="history"></div>

<script>
const noteDisplay = document.getElementById("note");
const historyDisplay = document.getElementById("history");

const noteStrings = [
  "C", "C#", "D", "D#", "E", "F",
  "F#", "G", "G#", "A", "A#", "B"
];

let notesHistory = [];

function frequencyToNote(freq) {
    const noteNum = 12 * (Math.log(freq / 440) / Math.log(2));
    const midi = Math.round(noteNum) + 69;
    return noteStrings[midi % 12];
}

function autoCorrelate(buffer, sampleRate) {
    let SIZE = buffer.length;
    let rms = 0;

    for (let i = 0; i < SIZE; i++) {
        rms += buffer[i] * buffer[i];
    }

    rms = Math.sqrt(rms / SIZE);

    if (rms < 0.01) return -1;

    let r1 = 0, r2 = SIZE - 1, threshold = 0.2;

    for (let i = 0; i < SIZE / 2; i++) {
        if (Math.abs(buffer[i]) < threshold) {
            r1 = i;
            break;
        }
    }

    for (let i = 1; i < SIZE / 2; i++) {
        if (Math.abs(buffer[SIZE - i]) < threshold) {
            r2 = SIZE - i;
            break;
        }
    }

    buffer = buffer.slice(r1, r2);
    SIZE = buffer.length;

    let c = new Array(SIZE).fill(0);

    for (let i = 0; i < SIZE; i++) {
        for (let j = 0; j < SIZE - i; j++) {
            c[i] += buffer[j] * buffer[j + i];
        }
    }

    let d = 0;
    while (c[d] > c[d + 1]) d++;

    let maxval = -1;
    let maxpos = -1;

    for (let i = d; i < SIZE; i++) {
        if (c[i] > maxval) {
            maxval = c[i];
            maxpos = i;
        }
    }

    return sampleRate / maxpos;
}

document.getElementById("startBtn").onclick = async () => {
    const stream = await navigator.mediaDevices.getUserMedia({
        audio: true
    });

    const audioContext = new AudioContext();
    const analyser = audioContext.createAnalyser();

    const source = audioContext.createMediaStreamSource(stream);
    source.connect(analyser);

    analyser.fftSize = 2048;

    const buffer = new Float32Array(analyser.fftSize);

    setInterval(() => {
        analyser.getFloatTimeDomainData(buffer);

        const freq = autoCorrelate(buffer, audioContext.sampleRate);

        if (freq > 0) {
            const note = frequencyToNote(freq);

            noteDisplay.textContent = note;

            if (
                notesHistory.length === 0 ||
                notesHistory[notesHistory.length - 1] !== note
            ) {
                notesHistory.push(note);
                historyDisplay.textContent = notesHistory.join(" ");
            }
        }
    }, 200);
};
</script>

</body>
</html>
