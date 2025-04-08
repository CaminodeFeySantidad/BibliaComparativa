<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Biblia Comparativa con Buscador Automático</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background-color: #f4f4f4;
    }
    .container {
      max-width: 800px;
      margin: auto;
      padding: 20px;
      background-color: #fff;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }
    #output {
      white-space: pre-wrap;
      word-wrap: break-word;
      background-color: #eee;
      padding: 10px;
      border-radius: 5px;
      margin-top: 20px;
      font-family: monospace;
      font-size: 14px;
    }
    select {
      padding: 10px;
      margin: 10px 0;
      font-size: 16px;
      border-radius: 5px;
      border: 1px solid #ddd;
      width: 100%;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 10px;
      margin: 10px 0;
    }
    .grid button {
      padding: 10px;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      background-color: #e0e0e0;
      cursor: pointer;
    }
    .grid button:hover {
      background-color: #ccc;
    }
    .toggle-button {
      padding: 10px;
      font-size: 16px;
      margin: 10px 0;
      border-radius: 5px;
      border: 1px solid #aaa;
      background-color: #ddd;
      cursor: pointer;
      width: 100%;
      text-align: left;
    }
    .hidden {
      display: none;
    }
    .version-title {
      font-weight: bold;
      font-size: 16px;
      margin-top: 20px;
      text-align: center;
    }
    .version-text {
      font-weight: normal;
      font-size: 14px;
      margin-top: 5px;
      text-align: left;
    }
  </style>
</head>
<body>
  <div class="container">
    <div>
      <label for="bookSelect">Selecciona un Libro:</label>
      <select id="bookSelect" onchange="updateChapters(true)">
        <option value="">Seleccionar Libro</option>
      </select>
    </div>

    <div>
      <button id="chapterToggle" class="toggle-button" onclick="toggleGrid('chapterGrid')">Capítulo 1</button>
      <div id="chapterGrid" class="grid hidden"></div>
    </div>

    <div>
      <button id="verseToggle" class="toggle-button" onclick="toggleGrid('verseGrid')">Versículo 1</button>
      <div id="verseGrid" class="grid hidden"></div>
    </div>

    <div id="output"></div>

    <div id="versionResults"></div>
  </div>

  <script>
    let bibleData = [];
    let selectedBook = '';
    let selectedChapter = '';
    let selectedVerse = '';

    const bibleVersions = {
      'Reina Valera 1960': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/Reina-Valera%2060.xml',
      'Latinoamericana 1995': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/Latinoamericana%2095.xml',
      'Nueva Traducción Viviente': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/NTV.xml',
      'Nueva Versión Internacional': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/NVI.xml',
      'Dios Habla Hoy': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/DHH.xml',
      'La Biblia de Las Américas': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/LBLA.xml'
    };

    const versionOrder = [
      'Reina Valera 1960',
      'Latinoamericana 1995',
      'Nueva Traducción Viviente',
      'Nueva Versión Internacional',
      'Dios Habla Hoy',
      'La Biblia de Las Américas'
    ];

    function loadBible() {
      fetch(bibleVersions['Reina Valera 1960'])
        .then(response => {
          if (!response.ok) throw new Error('Error al cargar el archivo XML');
          return response.text();
        })
        .then(xmlText => {
          const parser = new DOMParser();
          const xmlDoc = parser.parseFromString(xmlText, "application/xml");
          const books = xmlDoc.getElementsByTagName('b');

          bibleData = [];
          const bookSelect = document.getElementById('bookSelect');
          bookSelect.innerHTML = '<option value="">Seleccionar Libro</option>';

          for (let b = 0; b < books.length; b++) {
            const book = books[b];
            const bookName = book.getAttribute('n');
            const bookOption = document.createElement('option');
            bookOption.value = bookName;
            bookOption.textContent = bookName;
            bookSelect.appendChild(bookOption);

            const chapters = book.getElementsByTagName('c');
            for (let c = 0; c < chapters.length; c++) {
              const chapter = chapters[c];
              const chapterNumber = chapter.getAttribute('n');

              const verses = chapter.getElementsByTagName('v');
              for (let v = 0; v < verses.length; v++) {
                const verse = verses[v];
                const verseNumber = verse.getAttribute('n');
                const verseText = verse.textContent.trim();

                bibleData.push({
                  book: bookName,
                  chapter: chapterNumber,
                  verse: verseNumber,
                  text: verseText
                });
              }
            }
          }

          const firstBook = bookSelect.options[1]?.value;
          if (firstBook) {
            bookSelect.value = firstBook;
            updateChapters(true);
          }
        })
        .catch(error => {
          document.getElementById('output').textContent = 'Ocurrió un error: ' + error.message;
        });
    }

    function updateChapters(autoSelectFirst = false) {
      selectedBook = document.getElementById('bookSelect').value;
      selectedChapter = '';
      selectedVerse = '';
      const chapterGrid = document.getElementById('chapterGrid');
      const verseGrid = document.getElementById('verseGrid');
      chapterGrid.innerHTML = '';
      verseGrid.innerHTML = '';
      chapterGrid.classList.add('hidden');
      verseGrid.classList.add('hidden');

      if (selectedBook) {
        const chapters = [...new Set(bibleData.filter(v => v.book === selectedBook).map(v => v.chapter))];
        chapters.forEach(chapter => {
          const btn = document.createElement('button');
          btn.textContent = chapter;
          btn.onclick = () => {
            selectedChapter = chapter;
            document.getElementById('chapterToggle').textContent = `Capítulo ${chapter}`;
            updateVerses(true);
            chapterGrid.classList.add('hidden');
          };
          chapterGrid.appendChild(btn);
        });

        if (autoSelectFirst && chapters.length > 0) {
          selectedChapter = chapters[0];
          document.getElementById('chapterToggle').textContent = `Capítulo ${selectedChapter}`;
          updateVerses(true);
        }
      }
    }

    function updateVerses(autoSelectFirst = false) {
      const verseGrid = document.getElementById('verseGrid');
      verseGrid.innerHTML = '';
      verseGrid.classList.add('hidden');

      if (selectedBook && selectedChapter) {
        const verses = bibleData.filter(v => v.book === selectedBook && v.chapter === selectedChapter);
        verses.forEach(verse => {
          const btn = document.createElement('button');
          btn.textContent = verse.verse;
          btn.onclick = () => {
            selectedVerse = verse.verse;
            document.getElementById('verseToggle').textContent = `Versículo ${verse.verse}`;
            showVerse(verse);
            verseGrid.classList.add('hidden');
          };
          verseGrid.appendChild(btn);
        });

        if (autoSelectFirst && verses.length > 0) {
          selectedVerse = verses[0].verse;
          document.getElementById('verseToggle').textContent = `Versículo ${selectedVerse}`;
          showVerse(verses[0]);
        }
      }
    }

    function showVerse(verseDetails) {
      document.getElementById('output').textContent = '';  // Eliminar cualquier texto en el área de salida
      fetchVersionVerses();
    }

    function fetchVersionVerses() {
      const versionResults = document.getElementById('versionResults');
      versionResults.innerHTML = '';  // Limpiar resultados anteriores

      versionOrder.forEach(version => {
        const versionDiv = document.createElement('div');
        versionDiv.classList.add('version-title');
        versionDiv.innerHTML = `<strong>${version}</strong><div class="version-text">Cargando...</div>`;  // Añadir texto por defecto
        versionResults.appendChild(versionDiv);

        fetch(bibleVersions[version])
          .then(response => response.text())
          .then(xmlText => {
            const parser = new DOMParser();
            const xmlDoc = parser.parseFromString(xmlText, "application/xml");
            const verses = xmlDoc.querySelectorAll(`b[n="${selectedBook}"] c[n="${selectedChapter}"] v[n="${selectedVerse}"]`);

            if (verses.length > 0) {
              const verseText = verses[0].textContent.trim();
              versionDiv.querySelector('.version-text').textContent = verseText;  // Actualizar el texto con el versículo correspondiente
            } else {
              versionDiv.querySelector('.version-text').textContent = 'Versículo no encontrado';  // Si no se encuentra el versículo
            }
          })
          .catch(error => {
            versionDiv.querySelector('.version-text').textContent = 'Error al cargar';  // Mostrar mensaje de error
            console.error('Error al cargar la versión:', version, error);
          });
      });
    }

    function toggleGrid(id) {
      const grid = document.getElementById(id);
      grid.classList.toggle('hidden');
    }

    window.onload = loadBible;
  </script>
</body>
</html>
