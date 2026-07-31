# Create an updated HTML file with support for PDF and EPUB uploads, text extraction, speech synthesis, and book downloads/exports.
html_code = """<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Mi Creador Privado de Libros IA</title>

  <!-- PWA & Mobile Capabilities (Android y iOS) -->
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="Mis Libros IA">
  <meta name="theme-color" content="#375A7C">
  <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/3429/3429312.png">

  <!-- Manifest Data URI Inline para PWA -->
  <link rel="manifest" id="manifest-placeholder">

  <!-- Librerías Externas para Importar PDF (pdf.js), EPUB (epubjs & jszip) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>
  <script>
    pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.worker.min.js';
  </script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/epubjs/dist/epub.min.js"></script>

  <style>
    :root {
      --primary: #375A7C;
      --bg-light: #F9FFF3;
      --primary-light: #4A739C;
      --text-dark: #1E3143;
      --shadow: rgba(55, 90, 124, 0.15);
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      background-color: var(--bg-light);
      color: var(--text-dark);
      display: flex;
      height: 100vh;
      overflow: hidden;
    }

    #app-container {
      display: flex;
      width: 100vw;
      height: 100vh;
      position: relative;
    }

    /* Sidebar / Biblioteca */
    #sidebar {
      width: 320px;
      background-color: var(--primary);
      color: var(--bg-light);
      display: flex;
      flex-direction: column;
      transition: transform 0.3s ease;
      z-index: 10;
    }

    .sidebar-header {
      padding: 20px;
      font-size: 1.2rem;
      font-weight: bold;
      border-bottom: 1px solid rgba(249, 255, 243, 0.2);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .btn-group-sidebar {
      display: flex;
      flex-direction: column;
      gap: 10px;
      padding: 15px;
    }

    .btn-action {
      background-color: var(--bg-light);
      color: var(--primary);
      border: none;
      padding: 12px 16px;
      border-radius: 8px;
      font-weight: bold;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 8px;
      justify-content: center;
      box-shadow: 0 4px 6px var(--shadow);
      transition: opacity 0.2s;
    }

    .btn-action:active {
      opacity: 0.8;
    }

    .book-list {
      flex: 1;
      overflow-y: auto;
      padding: 10px 15px;
    }

    .book-item {
      padding: 12px 15px;
      border-radius: 8px;
      margin-bottom: 8px;
      cursor: pointer;
      background-color: rgba(249, 255, 243, 0.1);
      display: flex;
      justify-content: space-between;
      align-items: center;
      transition: background 0.2s;
    }

    .book-item:hover, .book-item.active {
      background-color: rgba(249, 255, 243, 0.25);
    }

    .book-item-title {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      max-width: 200px;
    }

    .btn-delete {
      background: none;
      border: none;
      color: #ff8b8b;
      font-size: 1.1rem;
      cursor: pointer;
      padding: 2px 6px;
    }

    /* Área Principal */
    #main-content {
      flex: 1;
      display: flex;
      flex-direction: column;
      background-color: var(--bg-light);
    }

    header {
      background-color: var(--primary);
      color: var(--bg-light);
      padding: 15px 20px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      box-shadow: 0 2px 8px var(--shadow);
    }

    .header-actions {
      display: flex;
      gap: 10px;
    }

    .menu-toggle {
      background: none;
      border: none;
      color: var(--bg-light);
      font-size: 1.4rem;
      cursor: pointer;
    }

    #chat-history {
      flex: 1;
      padding: 20px;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
      gap: 15px;
    }

    .message {
      max-width: 85%;
      padding: 14px 18px;
      border-radius: 16px;
      line-height: 1.5;
      position: relative;
      animation: fadeIn 0.3s ease;
      white-space: pre-wrap;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .message.user {
      align-self: flex-end;
      background-color: var(--primary);
      color: var(--bg-light);
      border-bottom-right-radius: 2px;
    }

    .message.ai {
      align-self: flex-start;
      background-color: white;
      color: var(--text-dark);
      border: 1px solid rgba(55, 90, 124, 0.2);
      border-bottom-left-radius: 2px;
      box-shadow: 0 2px 5px var(--shadow);
    }

    /* Controles de Audio */
    .audio-controls {
      display: flex;
      gap: 8px;
      margin-top: 10px;
      border-top: 1px solid rgba(0, 0, 0, 0.08);
      padding-top: 8px;
      flex-wrap: wrap;
    }

    .btn-audio {
      background: var(--bg-light);
      border: 1px solid var(--primary);
      color: var(--primary);
      padding: 6px 12px;
      border-radius: 12px;
      font-size: 0.8rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 5px;
      font-weight: 600;
    }

    .btn-audio:hover {
      background-color: var(--primary);
      color: var(--bg-light);
    }

    .btn-audio.stop {
      border-color: #d9534f;
      color: #d9534f;
    }
    .btn-audio.stop:hover {
      background-color: #d9534f;
      color: white;
    }

    /* Input Bar */
    .input-container {
      padding: 15px;
      background-color: white;
      border-top: 1px solid rgba(55, 90, 124, 0.15);
      display: flex;
      gap: 10px;
    }

    .input-container textarea {
      flex: 1;
      border: 1px solid var(--primary);
      border-radius: 20px;
      padding: 12px 18px;
      resize: none;
      outline: none;
      height: 48px;
      font-size: 1rem;
    }

    .btn-send {
      background-color: var(--primary);
      color: var(--bg-light);
      border: none;
      width: 48px;
      height: 48px;
      border-radius: 50%;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.2rem;
    }

    /* Modal de Ajustes */
    .modal {
      display: none;
      position: fixed;
      inset: 0;
      background-color: rgba(0, 0, 0, 0.5);
      z-index: 100;
      align-items: center;
      justify-content: center;
    }

    .modal-content {
      background-color: var(--bg-light);
      padding: 25px;
      border-radius: 16px;
      width: 90%;
      max-width: 420px;
    }

    .modal-content h3 {
      color: var(--primary);
      margin-bottom: 15px;
    }

    .form-group {
      margin-bottom: 15px;
    }

    .form-group label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
      font-size: 0.9rem;
    }

    .form-group select, .form-group input {
      width: 100%;
      padding: 10px;
      border-radius: 8px;
      border: 1px solid var(--primary);
    }

    /* Loader */
    #loader-overlay {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(55, 90, 124, 0.7);
      color: white;
      z-index: 200;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      font-size: 1.2rem;
      gap: 15px;
    }

    .spinner {
      width: 40px;
      height: 40px;
      border: 4px solid #F9FFF3;
      border-top: 4px solid transparent;
      border-radius: 50%;
      animation: spin 1s linear infinite;
    }

    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }

    /* Mobile Responsive */
    @media (max-width: 768px) {
      #sidebar {
        position: absolute;
        height: 100%;
        transform: translateX(-100%);
      }
      #sidebar.open {
        transform: translateX(0);
      }
    }
  </style>
</head>
<body>

  <div id="loader-overlay">
    <div class="spinner"></div>
    <div id="loader-text">Procesando archivo...</div>
  </div>

  <div id="app-container">
    <!-- Sidebar / Biblioteca -->
    <aside id="sidebar">
      <div class="sidebar-header">
        <span>📚 Biblioteca Privada</span>
      </div>
      
      <div class="btn-group-sidebar">
        <button class="btn-action" onclick="createNewBook()">➕ Nuevo Libro Interactivo</button>
        <button class="btn-action" onclick="triggerFileInput()">📂 Subir PDF / EPUB</button>
        <input type="file" id="fileInput" accept=".pdf, .epub" style="display: none;" onchange="handleFileUpload(event)">
      </div>

      <div class="book-list" id="bookList">
        <!-- Lista de libros -->
      </div>
    </aside>

    <!-- Área Principal -->
    <main id="main-content">
      <header>
        <button class="menu-toggle" onclick="toggleSidebar()">☰</button>
        <h2 id="bookTitle">Mi Libro Privado</h2>
        <div class="header-actions">
          <button class="menu-toggle" onclick="exportCurrentBook()" title="Descargar Libro">📥</button>
          <button class="menu-toggle" onclick="openVoiceSettings()" title="Ajustes de Voz">⚙️</button>
        </div>
      </header>

      <div id="chat-history">
        <!-- Contenido del libro -->
      </div>

      <div class="input-container">
        <textarea id="userInput" placeholder="Escribe tu idea, sugerencia o diálogo para la IA..."></textarea>
        <button class="btn-send" onclick="sendMessage()">➔</button>
      </div>
    </main>
  </div>

  <!-- Modal Configuración de Voces -->
  <div class="modal" id="voiceModal">
    <div class="modal-content">
      <h3>Configuración de Voces IA</h3>
      <div class="form-group">
        <label for="femaleVoice">Voz Femenina:</label>
        <select id="femaleVoice"></select>
      </div>
      <div class="form-group">
        <label for="maleVoice">Voz Masculina:</label>
        <select id="maleVoice"></select>
      </div>
      <div class="form-group">
        <label for="speechRate">Velocidad de Lectura:</label>
        <input type="range" id="speechRate" min="0.5" max="1.5" step="0.1" value="0.95">
      </div>
      <button class="btn-action" style="width:100%;" onclick="closeVoiceSettings()">Guardar Configuración</button>
    </div>
  </div>

  <script>
    // --- ESTADO DE LA APLICACIÓN ---
    let books = JSON.parse(localStorage.getItem('my_private_books')) || [
      {
        id: '1',
        title: 'Mi Primer Cap\u00edtulo',
        messages: [
          { sender: 'ai', text: '[Narrador]: Hab\u00eda una vez en un reino distante...', gender: 'male' },
          { sender: 'ai', text: '[Elena]: "\u00a1Hola! Bienvenido a tu espacio privado para escribir, subir tus libros en PDF/EPUB y escucharlos con voz real."', gender: 'female' }
        ]
      }
    ];
    let currentBookId = books[0]?.id || '1';
    let voices = [];

    // --- CARGA Y SELECCIÓN DE VOCES ---
    function loadVoices() {
      voices = window.speechSynthesis.getVoices();
      const femaleSelect = document.getElementById('femaleVoice');
      const maleSelect = document.getElementById('maleVoice');
      
      if (!femaleSelect || !maleSelect) return;

      femaleSelect.innerHTML = '';
      maleSelect.innerHTML = '';

      const esVoices = voices.filter(v => v.lang.includes('es') || v.lang.includes('ES'));
      const voiceList = esVoices.length > 0 ? esVoices : voices;

      voiceList.forEach((voice, index) => {
        const optionF = document.createElement('option');
        optionF.value = index;
        optionF.textContent = `${voice.name} (${voice.lang})`;
        
        const optionM = optionF.cloneNode(true);

        const vName = voice.name.toLowerCase();
        if (vName.includes('sabina') || vName.includes('monica') || vName.includes('female') || vName.includes('helena') || vName.includes('lucia') || vName.includes('maria')) {
          optionF.selected = true;
        }
        if (vName.includes('jorge') || vName.includes('pablo') || vName.includes('male') || vName.includes('raul') || vName.includes('diego') || vName.includes('enrique')) {
          optionM.selected = true;
        }

        femaleSelect.appendChild(optionF);
        maleSelect.appendChild(optionM);
      });
    }

    if (typeof speechSynthesis !== 'undefined') {
      speechSynthesis.onvoiceschanged = loadVoices;
    }

    function readText(text, gender) {
      if ('speechSynthesis' in window) {
        window.speechSynthesis.cancel(); // Detener cualquier reproducción previa
        
        // Limpiar sintaxis de etiquetas de formato si existen
        const cleanText = text.replace(/\[.*?\]:/g, '');

        const utterance = new SpeechSynthesisUtterance(cleanText);
        const rateVal = parseFloat(document.getElementById('speechRate')?.value || 0.95);
        utterance.rate = rateVal;
        utterance.pitch = gender === 'female' ? 1.1 : 0.9;

        const femaleSelect = document.getElementById('femaleVoice');
        const maleSelect = document.getElementById('maleVoice');
        
        const esVoices = voices.filter(v => v.lang.includes('es') || v.lang.includes('ES'));
        const availableVoices = esVoices.length > 0 ? esVoices : voices;

        if (gender === 'female' && availableVoices[femaleSelect?.value]) {
          utterance.voice = availableVoices[femaleSelect.value];
        } else if (gender === 'male' && availableVoices[maleSelect?.value]) {
          utterance.voice = availableVoices[maleSelect.value];
        }

        window.speechSynthesis.speak(utterance);
      }
    }

    function stopAudio() {
      if ('speechSynthesis' in window) {
        window.speechSynthesis.cancel();
      }
    }

    // --- MANEJO DE ARCHIVOS (PDF Y EPUB) ---
    function triggerFileInput() {
      document.getElementById('fileInput').click();
    }

    async function handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      showLoader(`Cargando y procesando "${file.name}"...`);

      try {
        let extractedText = '';
        const fileNameWithoutExt = file.name.replace(/\.[^/.]+$/, "");

        if (file.name.endsWith('.pdf')) {
          extractedText = await extractTextFromPDF(file);
        } else if (file.name.endsWith('.epub')) {
          extractedText = await extractTextFromEPUB(file);
        } else {
          alert('Formato no soportado. Por favor sube un archivo PDF o EPUB.');
          hideLoader();
          return;
        }

        if (!extractedText.trim()) {
          alert('No se pudo extraer texto legible del archivo.');
          hideLoader();
          return;
        }

        // Dividir el texto en párrafos/bloques para facilitar lectura
        const paragraphs = extractedText.split(/\n\s*\n/).filter(p => p.trim().length > 0);
        const messages = paragraphs.slice(0, 500).map((para, idx) => ({
          sender: 'ai',
          text: para.trim(),
          gender: idx % 2 === 0 ? 'male' : 'female'
        }));

        const newBook = {
          id: Date.now().toString(),
          title: fileNameWithoutExt,
          messages: messages
        };

        books.push(newBook);
        currentBookId = newBook.id;
        saveAndRefresh();
        alert(`¡Libro "${file.name}" importado con éxito!`);

      } catch (err) {
        console.error(err);
        alert('Ocurrió un error al procesar el libro: ' + err.message);
      } finally {
        hideLoader();
        event.target.value = '';
      }
    }

    // Extractor de PDF usando pdf.js
    async function extractTextFromPDF(file) {
      const arrayBuffer = await file.arrayBuffer();
      const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
      let fullText = '';

      for (let i = 1; i <= pdf.numPages; i++) {
        const page = await pdf.getPage(i);
        const textContent = await page.getTextContent();
        const pageText = textContent.items.map(item => item.str).join(' ');
        fullText += pageText + '\n\n';
      }
      return fullText;
    }

    // Extractor de EPUB usando ePub.js
    async function extractTextFromEPUB(file) {
      const arrayBuffer = await file.arrayBuffer();
      const book = ePub(arrayBuffer);
      await book.ready;

      let fullText = '';
      const spine = book.spine;

      for (const item of spine.spineItems) {
        try {
          const doc = await item.load(book.load.bind(book));
          const text = doc.textContent || doc.body.innerText || '';
          fullText += text + '\n\n';
        } catch (e) {
          console.warn('Error al leer capítulo del EPUB', e);
        }
      }
      return fullText;
    }

    // --- DESCARGAR / EXPORTAR LIBRO ---
    function exportCurrentBook() {
      const activeBook = books.find(b => b.id === currentBookId);
      if (!activeBook) return;

      let fileContent = `=====================================\n`;
      fileContent += `LIBRO: ${activeBook.title.toUpperCase()}\n`;
      fileContent += `Guardado en Mi Creador Privado de Libros IA\n`;
      fileContent += `=====================================\n\n`;

      activeBook.messages.forEach((msg, index) => {
        if (msg.sender === 'user') {
          fileContent += `[TÚ]: ${msg.text}\n\n`;
        } else {
          fileContent += `${msg.text}\n\n`;
        }
      });

      const blob = new Blob([fileContent], { type: 'text/plain;charset=utf-8' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${activeBook.title.replace(/[^a-z0-9]/gi, '_').toLowerCase()}.txt`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
    }

    // --- INTERFAZ Y RENDERIZADO ---
    function renderLibrary() {
      const bookListEl = document.getElementById('bookList');
      bookListEl.innerHTML = '';
      
      books.forEach(book => {
        const item = document.createElement('div');
        item.className = `book-item ${book.id === currentBookId ? 'active' : ''}`;
        
        const titleSpan = document.createElement('span');
        titleSpan.className = 'book-item-title';
        titleSpan.textContent = book.title;
        titleSpan.onclick = () => selectBook(book.id);

        const deleteBtn = document.createElement('button');
        deleteBtn.className = 'btn-delete';
        deleteBtn.innerHTML = '🗑️';
        deleteBtn.title = 'Eliminar libro';
        deleteBtn.onclick = (e) => {
          e.stopPropagation();
          deleteBook(book.id);
        };

        item.appendChild(titleSpan);
        if (books.length > 1) {
          item.appendChild(deleteBtn);
        }
        bookListEl.appendChild(item);
      });
    }

    function renderMessages() {
      const historyEl = document.getElementById('chat-history');
      historyEl.innerHTML = '';

      const activeBook = books.find(b => b.id === currentBookId);
      if (!activeBook) return;

      document.getElementById('bookTitle').textContent = activeBook.title;

      activeBook.messages.forEach((msg) => {
        const msgDiv = document.createElement('div');
        msgDiv.className = `message ${msg.sender}`;
        msgDiv.innerHTML = `<div>${msg.text}</div>`;

        if (msg.sender === 'ai') {
          const audioDiv = document.createElement('div');
          audioDiv.className = 'audio-controls';
          
          const btnFemale = document.createElement('button');
          btnFemale.className = 'btn-audio';
          btnFemale.innerHTML = '👩 Voz Mujer';
          btnFemale.onclick = () => readText(msg.text, 'female');

          const btnMale = document.createElement('button');
          btnMale.className = 'btn-audio';
          btnMale.innerHTML = '👨 Voz Hombre';
          btnMale.onclick = () => readText(msg.text, 'male');

          const btnStop = document.createElement('button');
          btnStop.className = 'btn-audio stop';
          btnStop.innerHTML = '⏹ Detener';
          btnStop.onclick = stopAudio;

          audioDiv.appendChild(btnFemale);
          audioDiv.appendChild(btnMale);
          audioDiv.appendChild(btnStop);
          msgDiv.appendChild(audioDiv);
        }

        historyEl.appendChild(msgDiv);
      });

      historyEl.scrollTop = historyEl.scrollHeight;
    }

    function sendMessage() {
      const input = document.getElementById('userInput');
      const text = input.value.trim();
      if (!text) return;

      const activeBook = books.find(b => b.id === currentBookId);
      activeBook.messages.push({ sender: 'user', text });

      setTimeout(() => {
        const isFemale = Math.random() > 0.5;
        const charName = isFemale ? 'Valeria' : 'Gabriel';
        const aiResponse = `[${charName}]: "${text.length > 25 ? 'Me parece un excelente giro para la trama. Sigamos explorando esta escena.' : 'Perfecto, anotado en el cap\u00edtulo.'}"`;
        
        activeBook.messages.push({
          sender: 'ai',
          text: aiResponse,
          gender: isFemale ? 'female' : 'male'
        });

        saveAndRefresh();
        readText(aiResponse, isFemale ? 'female' : 'male');
      }, 800);

      input.value = '';
      saveAndRefresh();
    }

    function createNewBook() {
      const title = prompt('T\u00edtulo del nuevo libro:');
      if (title && title.trim()) {
        const newBook = {
          id: Date.now().toString(),
          title: title.trim(),
          messages: [{ sender: 'ai', text: `[Narrador]: Inicio del libro "${title.trim()}".`, gender: 'male' }]
        };
        books.push(newBook);
        currentBookId = newBook.id;
        saveAndRefresh();
      }
    }

    function deleteBook(id) {
      if (confirm('¿Est\u00e1s seguro de que deseas eliminar este libro?')) {
        books = books.filter(b => b.id !== id);
        if (currentBookId === id) {
          currentBookId = books[0]?.id || '';
        }
        saveAndRefresh();
      }
    }

    function selectBook(id) {
      currentBookId = id;
      renderLibrary();
      renderMessages();
      if (window.innerWidth <= 768) {
        toggleSidebar();
      }
    }

    function saveAndRefresh() {
      localStorage.setItem('my_private_books', JSON.stringify(books));
      renderLibrary();
      renderMessages();
    }

    function toggleSidebar() {
      document.getElementById('sidebar').classList.toggle('open');
    }

    function openVoiceSettings() {
      document.getElementById('voiceModal').style.display = 'flex';
      loadVoices();
    }

    function closeVoiceSettings() {
      document.getElementById('voiceModal').style.display = 'none';
    }

    function showLoader(msg) {
      document.getElementById('loader-text').textContent = msg;
      document.getElementById('loader-overlay').style.display = 'flex';
    }

    function hideLoader() {
      document.getElementById('loader-overlay').style.display = 'none';
    }

    // --- CONFIGURACIÓN PWA ---
    function setupPWA() {
      const manifest = {
        name: "Mis Libros IA Privados",
        short_name: "Libros IA",
        start_url: ".",
        display: "standalone",
        background_color: "#F9FFF3",
        theme_color: "#375A7C",
        icons: [{
          src: "https://cdn-icons-png.flaticon.com/512/3429/3429312.png",
          sizes: "512x512",
          type: "image/png"
        }]
      };
      const stringManifest = JSON.stringify(manifest);
      const blob = new Blob([stringManifest], {type: 'application/json'});
      const manifestURL = URL.createObjectURL(blob);
      document.getElementById('manifest-placeholder').setAttribute('href', manifestURL);
    }

    window.onload = () => {
      setupPWA();
      renderLibrary();
      renderMessages();
      setTimeout(loadVoices, 500);
    };
  </script>
</body>
</html>
"""

with open("index.html", "w", encoding="utf-8") as f:
    f.write(html_code)

print("HTML actualizado creado con exito.")
