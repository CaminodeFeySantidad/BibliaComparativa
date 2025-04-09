<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Biblia Comparativa con Buscador Automático</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      background-color: #f4f4f4;
    }

    .container {
      width: 100%;
      max-width: 900px;
      margin: auto;
      padding: 20px;
      background-color: #fff;
      box-sizing: border-box;
    }

    select {
      padding: 12px;
      margin: 12px 0;
      font-size: 16px;
      border-radius: 6px;
      border: 1px solid #ccc;
      width: 100%;
    }

    .row-buttons {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 10px;
    }

    .toggle-section {
      flex: 1 1 48%;
    }

    .toggle-button {
      padding: 12px;
      font-size: 16px;
      border-radius: 6px;
      border: 1px solid #aaa;
      background-color: #ddd;
      cursor: pointer;
      width: 100%;
      box-sizing: border-box;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 10px;
      margin: 10px 0;
    }

    .grid button {
      padding: 12px;
      font-size: 16px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }

    .grid.chapter-mode button {
      background-color: #e0f7fa;
    }

    .grid.verse-mode button {
      background-color: #fce4ec;
    }

    .grid button:hover {
      background-color: #ccc;
    }

    .hidden {
      display: none;
    }

    .version-title {
      font-weight: bold;
      font-size: 18px;
      margin-top: 30px;
      margin-bottom: 10px;
      padding-top: 10px;
      border-top: 1px solid #ddd;
      color: #333;
    }

    .version-text {
      font-size: 16px;
      line-height: 1.6;
      padding: 10px 0;
      color: #444;
    }

    @media (max-width: 600px) {
      .toggle-section {
        flex: 1 1 100%;
      }

      .toggle-button {
        font-size: 15px;
        padding: 10px;
      }

      .grid {
        grid-template-columns: repeat(5, 1fr);
      }

      .grid button {
        font-size: 14px;
        padding: 10px;
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
    <label for="bookSelect">Selecciona un Libro:</label>
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

    <div id="numberGrid" class="grid hidden"></div>

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

          for (let b of books) {
            const bookName = b.getAttribute('n');
            const bookOption = document.createElement('option');
            bookOption.value = bookName;
            bookOption.textContent = bookName;
            bookSelect.appendChild(bookOption);

            for (let c of b.getElementsByTagName('c')) {
              const chapterNumber = c.getAttribute('n');
              for (let v of c.getElementsByTagName('v')) {
                bibleData.push({
                  book: bookName,
                  chapter: chapterNumber,
                  verse: v.getAttribute('n'),
                  text: v.textContent.trim()
                });
              }
            }
          }

          const firstBook = bookSelect.options[1]?.value;
          if (firstBook) {
            bookSelect.value = firstBook;
            updateChapters(true);
          }
        });
    }

    function updateChapters(autoSelectFirst = false) {
      selectedBook = document.getElementById('bookSelect').value;
      selectedChapter = '';
      selectedVerse = '';
      document.getElementById('numberGrid').classList.add('hidden');
      if (selectedBook) {
        const chapters = [...new Set(bibleData.filter(v => v.book === selectedBook).map(v => v.chapter))];
        if (autoSelectFirst && chapters.length > 0) {
          selectedChapter = chapters[0];
          document.getElementById('chapterToggle').textContent = `Capítulo ${selectedChapter}`;
          updateVerses(true);
        }
      }
    }

    function updateVerses(autoSelectFirst = false) {
      if (selectedBook && selectedChapter) {
        const verses = bibleData.filter(v => v.book === selectedBook && v.chapter === selectedChapter);
        if (autoSelectFirst && verses.length > 0) {
          selectedVerse = verses[0].verse;
          document.getElementById('verseToggle').textContent = `Versículo ${selectedVerse}`;
          showVerse(verses[0]);
        }
      }
    }

    function toggleGrid(mode) {
      const grid = document.getElementById('numberGrid');
      grid.innerHTML = '';
      currentMode = mode;
      grid.classList.remove('hidden', 'chapter-mode', 'verse-mode');
      grid.classList.add(mode === 'chapter' ? 'chapter-mode' : 'verse-mode');

      if (mode === 'chapter') {
        const chapters = [...new Set(bibleData.filter(v => v.book === selectedBook).map(v => v.chapter))];
        chapters.forEach(chapter => {
          const btn = document.createElement('button');
          btn.textContent = chapter;
          btn.onclick = () => {
            selectedChapter = chapter;
            document.getElementById('chapterToggle').textContent = `Capítulo ${chapter}`;
            updateVerses(true);
            grid.classList.add('hidden');
          };
          grid.appendChild(btn);
        });
      } else if (mode === 'verse') {
        const verses = bibleData.filter(v => v.book === selectedBook && v.chapter === selectedChapter);
        verses.forEach(verse => {
          const btn = document.createElement('button');
          btn.textContent = verse.verse;
          btn.onclick = () => {
            selectedVerse = verse.verse;
            document.getElementById('verseToggle').textContent = `Versículo ${verse.verse}`;
            showVerse(verse);
            grid.classList.add('hidden');
          };
          grid.appendChild(btn);
        });
      }
    }

    function showVerse(verseDetails) {
      fetchVersionVerses();
    }

    function fetchVersionVerses() {
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
          });
      });
    }

    window.onload = loadBible;
  </script>
</body>
</html>
