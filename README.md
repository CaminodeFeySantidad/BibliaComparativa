<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Biblia Comparativa</title>
  <style>
    body {
      margin: 0;
      padding: 10px;
    }

    .container {
      max-width: 900px;
      margin: auto;
      padding: 15px;
      background-color: rgba(255, 255, 255, 0.5);
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      border-radius: 8px;
    }

    #bookSelect {
      width: 100%;
      padding: 10px;
      font-size: 16px;
      border-radius: 5px;
      margin-bottom: 15px;
      border: 1px solid #ccc;
      background-color: #fff;
    }

    .row-buttons {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 10px;
    }

    .toggle-section {
      flex: 1;
      min-width: 150px;
    }

    .toggle-button {
      width: 100%;
      padding: 10px;
      font-size: 16px;
      border-radius: 5px;
      border: 1px solid #242440;
      background-color: #242440;
      color: white;
      cursor: pointer;
    }

    .toggle-button:hover {
      background-color: #1d1f35;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 8px;
      margin-top: 10px;
    }

    .grid button {
      padding: 10px;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      background-color: white; /* Fondo blanco */
      color: black; /* Texto negro */
      cursor: pointer;
    }

    .grid button:hover {
      background-color: #f1f1f1; /* Fondo blanco claro al pasar el ratón */
    }

    .grid.chapter-mode button {
      background-color: white; /* Fondo blanco */
    }

    .grid.verse-mode button {
      background-color: white; /* Fondo blanco */
    }

    .hidden {
      display: none;
    }

    .version-title {
      font-weight: bold;
      font-size: 18px;
      margin-top: 20px;
      text-align: center;
    }

    .version-text {
      font-weight: normal;
      font-size: 16px;
      margin-top: 5px;
      text-align: left;
    }

    @media (max-width: 600px) {
      .container {
        padding: 10px;
        max-width: 95vw;
      }

      #bookSelect {
        font-size: 14px;
        padding: 8px;
      }

      .toggle-section {
        flex: 1 1 48%;
        min-width: unset;
      }

      .toggle-button {
        font-size: 14px;
        padding: 8px;
      }

      .grid {
        gap: 6px;
      }

      .grid button {
        font-size: 13px;
        padding: 8px;
      }

      .version-title {
        font-size: 16px;
      }

      .version-text {
        font-size: 15px;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <select id="bookSelect" onchange="updateChapters(true)">
      <option value="">Seleccionar Libro</option>
    </select>

    <div class="row-buttons">
      <div class="toggle-section">
        <button id="chapterToggle" class="toggle-button" onclick="toggleGrid('chapter')">Capítulo 1</button>
      </div>
      <div class="toggle-section">
        <button id="verseToggle" class="toggle-button" onclick="toggleGrid('verse')">Versículo 1</button>
      </div>
    </div>

    <div id="sharedGrid" class="grid hidden"></div>
    <div id="versionResults"></div>
  </div>

  <script>
    let bibleData = [];
    let selectedBook = '';
    let selectedChapter = '';
    let selectedVerse = '';
    let currentMode = '';

    const bibleVersions = {
      'Reina Valera 1960': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/Reina-Valera%2060.xml',
      'Latinoamericana 1995': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/Latinoamericana%2095.xml',
      'Nueva Traducción Viviente': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/NTV.xml',
      'Nueva Versión Internacional': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/NVI.xml',
      'La Biblia de Las Américas': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/LBLA.xml',
      'Dios Habla Hoy': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/DHH.xml'
    };

    const versionOrder = [
      'Reina Valera 1960',
      'Latinoamericana 1995',
      'Nueva Traducción Viviente',
      'Nueva Versión Internacional',
      'La Biblia de Las Américas',
      'Dios Habla Hoy'
    ];

    function loadBible() {
      fetch(bibleVersions['Reina Valera 1960'])
        .then(response => response.text())
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
                bibleData.push({ book: bookName, chapter: chapterNumber, verse: verseNumber, text: verseText });
              }
            }
          }

          const firstBook = bookSelect.options[1]?.value;
          if (firstBook) {
            bookSelect.value = firstBook;
            updateChapters(true);
          }
        })
        .catch(error => console.error('Error:', error));
    }

    function updateChapters(autoSelect = false) {
      selectedBook = document.getElementById('bookSelect').value;
      selectedChapter = '';
      selectedVerse = '';
      document.getElementById('chapterToggle').textContent = 'Capítulo 1';
      document.getElementById('verseToggle').textContent = 'Versículo 1';

      if (selectedBook) {
        const chapters = [...new Set(bibleData.filter(v => v.book === selectedBook).map(v => v.chapter))];
        if (autoSelect && chapters.length > 0) {
          selectedChapter = chapters[0];
          document.getElementById('chapterToggle').textContent = `Capítulo ${selectedChapter}`;
          updateVerses(true);
        }
      }
    }

    function updateVerses(autoSelect = false) {
      if (selectedBook && selectedChapter) {
        const verses = bibleData.filter(v => v.book === selectedBook && v.chapter === selectedChapter);
        if (autoSelect && verses.length > 0) {
          selectedVerse = verses[0].verse;
          document.getElementById('verseToggle').textContent = `Versículo ${selectedVerse}`;
          showVerse();
        }
      }
    }

    function toggleGrid(mode) {
      const grid = document.getElementById('sharedGrid');
      if (currentMode === mode) {
        grid.classList.add('hidden');
        currentMode = '';
      } else {
        currentMode = mode;
        grid.classList.remove('hidden');
        grid.innerHTML = '';
        grid.classList.remove('chapter-mode', 'verse-mode');
        grid.classList.add(mode === 'chapter' ? 'chapter-mode' : 'verse-mode');

        if (mode === 'chapter') {
          const chapters = [...new Set(bibleData.filter(v => v.book === selectedBook).map(v => v.chapter))];
          chapters.forEach(ch => {
            const btn = document.createElement('button');
            btn.textContent = ch;
            btn.onclick = () => {
              selectedChapter = ch;
              document.getElementById('chapterToggle').textContent = `Capítulo ${ch}`;
              grid.classList.add('hidden');
              updateVerses(true);
            };
            grid.appendChild(btn);
          });
        } else if (mode === 'verse') {
          const verses = bibleData.filter(v => v.book === selectedBook && v.chapter === selectedChapter);
          verses.forEach(v => {
            const btn = document.createElement('button');
            btn.textContent = v.verse;
            btn.onclick = () => {
              selectedVerse = v.verse;
              document.getElementById('verseToggle').textContent = `Versículo ${v.verse}`;
              grid.classList.add('hidden');
              showVerse();
            };
            grid.appendChild(btn);
          });
        }
      }
    }

    function showVerse() {
      const versionResults = document.getElementById('versionResults');
      versionResults.innerHTML = '';

      const fetchPromises = versionOrder.map(version =>
        fetch(bibleVersions[version])
          .then(response => response.text())
          .then(xmlText => {
            const parser = new DOMParser();
            const xmlDoc = parser.parseFromString(xmlText, "application/xml");
            const verseNode = xmlDoc.querySelector(`b[n="${selectedBook}"] c[n="${selectedChapter}"] v[n="${selectedVerse}"]`);
            const verseText = verseNode ? verseNode.textContent.trim() : '(Versículo no encontrado)';
            return { version, verseText };
          })
          .catch(() => ({ version, verseText: '(Error al cargar)' }))
      );

      Promise.all(fetchPromises).then(results => {
        results.forEach(({ version, verseText }) => {
          const versionDiv = document.createElement('div');
          versionDiv.classList.add('version-title');

          const versionButton = document.createElement('button');
          versionButton.style = "all: unset; cursor: pointer; font-weight: bold;";
          versionButton.textContent = version;
          versionButton.onclick = () => copyVerseText(versionButton, verseText);

          versionDiv.appendChild(versionButton);
          const verseDiv = document.createElement('div');
          verseDiv.classList.add('version-text');
          verseDiv.textContent = verseText;
          versionDiv.appendChild(verseDiv);

          versionResults.appendChild(versionDiv);
        });
      });
    }

    function copyVerseText(el, verseText) {
      const reference = `${selectedBook} ${selectedChapter}:${selectedVerse} `;
      const fullTextToCopy = reference + verseText;

      navigator.clipboard.writeText(fullTextToCopy).then(() => {
        const originalText = el.textContent;
        el.textContent = originalText + ' Copiado';
        setTimeout(() => {
          el.textContent = originalText;
        }, 1500);
      });
    }

    window.onload = loadBible;
  </script>
</body>
</html>
