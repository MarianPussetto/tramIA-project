<template>
  <div class="fondo">
    <div class="header">
      <h1 class="titulo">Tram<span>IA</span></h1>
      <h2 class="subtitulo">by Danka</h2>
      <p class="descripcion">
        Herramienta con IA para traducir trámites estatales y requisitos complejos a un lenguaje simple y accesible.
      </p>
    </div>

    <main class="contenedor">
      <textarea v-model="texto" placeholder="Ingresá el texto que querés entender mejor" class="textarea"></textarea>

      <label class="label">O subí un archivo PDF ↓</label>
      <input type="file" accept="application/pdf" @change="leerPDF" class="input" />

      <button @click="procesarTexto" class="boton">
        Pasar en limpio
      </button>

      <div v-if="respuesta" class="respuesta">
        <h2 class="respuesta-titulo">Resultado:</h2>
        <p>{{ respuesta }}</p>
      </div>
    </main>
  </div>
</template>

<script>
import * as pdfjsLib from 'pdfjs-dist/legacy/build/pdf';

pdfjsLib.GlobalWorkerOptions.workerSrc = new URL(
  'pdfjs-dist/legacy/build/pdf.worker.js',
  import.meta.url
).toString();

export default {
  data() {
    return {
      texto: '',
      respuesta: '',
      backgroundUrl: '/background-app.jpg', // ✅ imagen en public
    };
  },

  methods: {
    async leerPDF(event) {
      const archivo = event.target.files[0];
      if (!archivo) return;

      const lector = new FileReader();
      lector.onload = async (e) => {
        const typedarray = new Uint8Array(e.target.result);
        const pdf = await pdfjsLib.getDocument({ data: typedarray }).promise;

        let textoExtraido = '';
        for (let i = 1; i <= pdf.numPages; i++) {
          const pagina = await pdf.getPage(i);
          const contenido = await pagina.getTextContent();
          const textoPagina = contenido.items.map(item => item.str).join(' ');
          textoExtraido += textoPagina + '\n';
        }

        this.texto = textoExtraido.trim();
      };
      lector.readAsArrayBuffer(archivo);
    },

    async procesarTexto() {
      if (!this.texto.trim()) return;

      const prompt = `Explicá este texto en lenguaje claro, como si se lo contaras a una persona común. No uses palabras técnicas ni legales. Sé amable, concreta y sencilla:\n\n${this.texto}`;

      try {
        const respuestaIA = await fetch("https://openrouter.ai/api/v1/chat/completions", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Authorization": "Bearer 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", //
            "HTTP-Referer": "http://localhost:5173/",
            "X-Title": "TramIA"
          },
          body: JSON.stringify({
            model: "openai/gpt-3.5-turbo",
            messages: [
              { role: "system", content: "Sos una traductora de textos burocráticos al lenguaje ciudadano." },
              { role: "user", content: prompt }
            ]
          })
        });

        const data = await respuestaIA.json();
        this.respuesta = data.choices?.[0]?.message?.content || "Hubo un error al procesar el texto.";
      } catch (error) {
        console.error("Error al llamar a la API:", error);
        this.respuesta = "Hubo un error al conectar con la API.";
      }
    }
  }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@200..900&display=swap');

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  font-family: 'Manrope', sans-serif;
  background-image: url('/background-app.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  height: 100dvh;
  overflow-x: hidden;
  overflow-y: auto;
  display: flex;
  justify-content: center;
  align-items: center;
}

.fondo {
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  padding: 16px;
  gap: 80px;
  box-sizing: border-box;
}

.header {
  text-align: center;
}

.subtitulo {
  font-size: 1.1rem;
  color: #bbb;
  margin-top: -62px;
  margin-right: 27%;
  font-weight: 500;
  text-align: right;
  text-shadow: 0 0 3px #ccc;
}

.descripcion {
  font-family: 'Fira Code', monospace;
  font-size: 1rem;
  font-weight: 800;
  color: #e0e0e0;
  margin-top: 1.5rem;
  max-width: 100%;
  margin-left: auto;
  margin-right: auto;
}

.contenedor {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  padding: 32px;
  max-width: 45%;
  width: 90%;
  max-height: 100%;
  overflow-y: auto;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.25);
  transition: all 0.3s ease-in-out;
}

.titulo {
  color: #ffffff;
  font-size: 72px;
  font-weight: 900;
  margin-bottom: 60px;
  text-align: center;
  text-shadow: 0 0 6px rgba(0, 0, 0, 0.5);
}

.titulo span {
  color: #39FF14;
  font-weight: 900;
  text-shadow: 0 0 5px #39FF14;
}

.textarea {
  resize: none;
  display: block;
  width: 100% !important;
  max-height: 30vh;
  height: 180px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
  font-family: 'Manrope', sans-serif;
  font-size: 16px;
  line-height: 1.6;
  color: #222;
  background-color: #eee;
  transition: border-color 0.2s, box-shadow 0.2s;
  overflow: auto;
}


.textarea:focus {
  border-color: #39FF14;
  box-shadow: 0 0 0 3px rgba(57, 255, 20, 0.2);
  outline: none;
}

.textarea::-webkit-scrollbar {
  width: 10px;
}

.textarea::-webkit-scrollbar-track {
  background: #e6f3e6;
  border-radius: 6px;
}

.textarea::-webkit-scrollbar-thumb {
  background-color: #6bd97e;
  border-radius: 6px;
  border: 2px solid #e6f3e6;
}

.textarea::-webkit-scrollbar-thumb:hover {
  background-color: #4ecf6c;
}

.label {
  display: block;
  margin-top: 10px;
  margin-bottom: 8px;
  font-weight: 600;
  color: #ccc;
  text-align: center;
  font-size: 16px;
}

.input-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.input {
  margin-bottom: 20px;
}

input[type="file"]::-webkit-file-upload-button {
  background-color: #e6e6e6;
  margin-top: 20px;
  margin-bottom: 20px;
  margin-right: 10px;
  color: #000;
  padding: 10px 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  font-family: 'Manrope', sans-serif;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
}

input[type="file"]::-webkit-file-upload-button:hover {
  background-color: #dcdcdc;
  box-shadow: 0 0 0 2px #39FF14aa;
}

input[type="file"]::-webkit-file-upload-button:focus {
  outline: none;
  box-shadow: 0 0 0 2px #39FF14aa;
}

input[type="file"] {
  color: #ddd;
  font-size: 14px;
  font-family: 'Manrope', sans-serif;
}

.boton {
  background: linear-gradient(90deg, #00cc66, #00b34d);
  color: white;
  padding: 14px;
  width: 100%;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 800;
  font-size: 16px;
  transition: transform 0.2s ease, box-shadow 0.3s ease;
  box-shadow: 0 2px 6px rgba(0, 255, 128, 0.25);
  position: relative;
  overflow: hidden;
}

.boton::before {
  content: "";
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.12) 0%, transparent 80%);
  animation: pulse 3s infinite;
  pointer-events: none;
}

.boton:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 10px rgba(0, 255, 128, 0.35);
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 0.2;
  }

  50% {
    transform: scale(1.05);
    opacity: 0.1;
  }

  100% {
    transform: scale(1);
    opacity: 0.2;
  }
}


.respuesta {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 16px;
  padding: 28px;
  color: #e8fbe8;
  font-family: 'Manrope', sans-serif;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.25);
  text-align: center;
  max-height: 30vh;
  overflow-y: auto;
  margin-top: 48px;
  animation: fadeIn 0.5s ease-in-out;
  max-width: 640px;
  margin-left: auto;
  margin-right: auto;
}

.respuesta h2,
.respuesta .respuesta-titulo {
  font-size: 22px;
  font-weight: 800;
  color: #fff;
  margin-bottom: 14px;
  text-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
}

.respuesta p {
  font-size: 16px;
  line-height: 1.6;
  color: #dcfcdc;
}

.respuesta::-webkit-scrollbar {
  width: 10px;
}

.respuesta::-webkit-scrollbar-track {
  background: #e6f3e6;
  /* igual que el de arriba */
  border-radius: 6px;
}

.respuesta::-webkit-scrollbar-thumb {
  background-color: #6bd97e;
  border-radius: 6px;
  border: 2px solid #e6f3e6;
}

.respuesta::-webkit-scrollbar-thumb:hover {
  background-color: #4ecf6c;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/*ADAPTACIÓN MOBILE*/
@media (max-width: 767px) {

  html,
  body {
    margin: 0;
    padding: 0;
    width: 100vw;
    min-height: 100vh;
    height: auto;
    overflow-x: hidden;
    overflow-y: auto;
    box-sizing: border-box;
    display: block;
  }

  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  #app {
    width: 100%;
    max-width: 100%;
    padding: 0;
    margin: 0 auto;
  }

  .fondo {
    width: 100vw;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 24px;
    padding: 24px 16px;
  }

  .header {
    text-align: center;
    padding: 8px 16px;
  }

  .titulo {
    font-size: 50px;
    margin-bottom: 4px;
    text-align: center;
  }

  .subtitulo {
    font-size: 0.98rem;
    margin-top: -2px;
    margin-bottom: 30px;
    margin-right: 0;
    text-align: center;
  }

  .descripcion {
    font-size: 1.1rem;
    text-align: center;
    padding: 0 16px;
    line-height: 1.4;
    max-width: 100%;
    /* 🔧 antes 90%, para evitar margen visual raro */
    margin: 0 0 0 auto;
  }

  .contenedor {
    width: 90%;
    max-width: 100%;
    padding: 24px 16px;
    margin: 0 auto;
    overflow-x: hidden;
  }

  .textarea {
    height: 140px;
    font-size: 16px;
    margin-bottom: 16px;
    width: 100%;
  }

  .input-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    text-align: center;
  }

  input[type="file"] {
    display: block;
    margin: 0 auto;
    width: 100%;
    max-width: 280px;
    text-align: center;
    font-size: 13px;
    margin-bottom: 10px;
  }

  input[type="file"]::-webkit-file-upload-button {
    margin: 0 auto 6px;
    margin-right: 3px;
    max-width: 100%;
    font-size: 14px;
  }

  .boton {
    width: 100%;
    font-size: 16px;
    padding: 14px;
    margin-top: 8px;
  }

  .respuesta {
    padding: 20px;
    font-size: 14px;
    margin-top: 32px;
    width: 100%;
  }
}
</style>
