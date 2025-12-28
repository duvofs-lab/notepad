const notepad = document.getElementById("notepad");
const charCount = document.getElementById("charCount");
const wordCount = document.getElementById("wordCount");
const saveBtn = document.getElementById("saveBtn");
const pdfBtn = document.getElementById("pdfBtn");
const micBtn = document.getElementById("micBtn");
const fontUp = document.getElementById("fontUp");
const fontDown = document.getElementById("fontDown");
const darkBtn = document.getElementById("darkModeBtn");

let fontSize = 11;
let recognition;
let isListening = false;

const STORAGE_KEY = "duvofs_notepad_content";

/* LOAD SAVED CONTENT */
const savedText = localStorage.getItem(STORAGE_KEY);
if (savedText) {
  notepad.value = savedText;
}

/* COUNT + AUTO SAVE */
function updateCounts() {
  const text = notepad.value;
  charCount.textContent = text.length;
  wordCount.textContent = text.trim() ? text.trim().split(/\s+/).length : 0;
  localStorage.setItem(STORAGE_KEY, text);
}
notepad.addEventListener("input", updateCounts);
updateCounts();

/* SAVE AS TEXT */
saveBtn.onclick = () => {
  const blob = new Blob([notepad.value], { type: "text/plain" });
  const link = document.createElement("a");
  link.download = `duvofsnotepad${Date.now()}.txt`;
  link.href = URL.createObjectURL(blob);
  link.click();
};

/* EXPORT PDF */
pdfBtn.onclick = () => {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  const lines = doc.splitTextToSize(notepad.value || " ", 180);
  doc.text(lines, 10, 10);
  doc.save(`duvofsnotepad${Date.now()}.pdf`);
};

/* FONT SIZE */
fontUp.onclick = () => {
  if (fontSize < 18) fontSize++;
  notepad.style.fontSize = fontSize + "px";
};

fontDown.onclick = () => {
  if (fontSize > 10) fontSize--;
  notepad.style.fontSize = fontSize + "px";
};

/* DARK MODE */
if (localStorage.getItem("duvofs_darkmode") === "on") {
  document.documentElement.classList.add("dark");
}

darkBtn.onclick = () => {
  document.documentElement.classList.toggle("dark");
  localStorage.setItem(
    "duvofs_darkmode",
    document.documentElement.classList.contains("dark") ? "on" : "off"
  );
};

/* SPEECH TO TEXT */
if ("webkitSpeechRecognition" in window) {
  recognition = new webkitSpeechRecognition();
  recognition.continuous = true;

  recognition.onresult = (event) => {
    const transcript = event.results[event.results.length - 1][0].transcript;
    notepad.value += " " + transcript;
    updateCounts();
  };
}

micBtn.onclick = () => {
  if (!recognition) return alert("Speech not supported");

  if (!isListening) {
    recognition.start();
    micBtn.textContent = "🎙️";
    isListening = true;
  } else {
    recognition.stop();
    micBtn.textContent = "🎤";
    isListening = false;
  }
};

/* SERVICE WORKER */
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("sw.js");
}
