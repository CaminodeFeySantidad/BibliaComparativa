<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Biblia Comparativa</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 10px;
      background-color: #f4f4f4;
    }

    .container {
      max-width: 900px;
      margin: auto;
      padding: 15px;
      background-color: #fff;
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
      border: 1px solid #aaa;
      background-color: #ddd;
      cursor: pointer;
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
      background-color: #e0e0e0;
      cursor: pointer;
    }

    .grid button:hover {
      background-color: #ccc;
    }

    .grid.chapter-mode button {
      background-color: #e0e0e0;
    }

    .grid.verse-mode button {
      background-color: #f0e0ff;
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
      'Dios Habla Hoy': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/DHH.xml',
      'La Biblia de Las Américas': 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/LBLA.xml'
    };

    const versionOrder = Object.keys(bibleVersions);

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
        // If the same button is clicked, hide the grid
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
      versionOrder.forEach(version => {
        fetch(bibleVersions[version])
          .then(response => response.text())
          .then(xmlText => {
            const parser = new DOMParser();
            const xmlDoc = parser.parseFromString(xmlText, "application/xml");
            const verses = xmlDoc.querySelectorAll(`b[n="${selectedBook}"] c[n="${selectedChapter}"] v[n="${selectedVerse}"]`);
            if (verses.length > 0) {
              const verseText = verses[0].textContent.trim();
              const versionDiv = document.createElement('div');
              versionDiv.classList.add('version-title');
              versionDiv.innerHTML = `<strong>${version}</strong><div class="version-text">${verseText}</div>`;
              versionResults.appendChild(versionDiv);
            }
          })
          .catch(error => console.error('Error al cargar la versión:', version, error));
      });
    }

    // Gestor de deslizamiento
    let startTouchX = 0;

    document.addEventListener('touchstart', function (e) {
      startTouchX = e.changedTouches[0].pageX;
    }, false);

    document.addEventListener('touchend', function (e) {
      const endTouchX = e.changedTouches[0].pageX;
      const touchDifference = startTouchX - endTouchX;

      if (Math.abs(touchDifference) > 50) { // Si la diferencia de deslizamiento es significativa
        if (touchDifference > 0) {
          // Deslizar hacia la izquierda, retroceder al versículo anterior
          goToPreviousVerse();
        } else {
          // Deslizar hacia la derecha, avanzar al siguiente versículo
          goToNextVerse();
        }
      }
    }, false);

    function goToNextVerse() {
      if (selectedBook && selectedChapter && selectedVerse) {
        const verses = bibleData.filter(v => v.book === selectedBook && v.chapter === selectedChapter);
        const currentIndex = verses.findIndex(v => v.verse === selectedVerse);
        const nextVerse = verses[currentIndex + 1];

        if (nextVerse) {
          selectedVerse = nextVerse.verse;
          document.getElementById('verseToggle').textContent = `Versículo ${selectedVerse}`;
          showVerse();
        }
      }
    }

    function goToPreviousVerse() {
      if (selectedBook && selectedChapter && selectedVerse) {
        const verses = bibleData.filter(v => v.book === selectedBook && v.chapter === selectedChapter);
        const currentIndex = verses.findIndex(v => v.verse === selectedVerse);
        const previousVerse = verses[currentIndex - 1];

        if (previousVerse) {
          selectedVerse = previousVerse.verse;
          document.getElementById('verseToggle').textContent = `Versículo ${selectedVerse}`;
          showVerse();
        }
      }
    }

    window.onload = loadBible;
  </script>
</body>
</html>
