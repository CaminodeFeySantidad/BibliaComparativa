<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Biblia Comparativa Online</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            padding: 0;
            background-color: #f4f4f4;
        }
        h1 {
            text-align: center;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #fff;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .verse {
            margin-bottom: 15px;
        }
        .verse span {
            font-weight: bold;
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
        select, button {
            padding: 10px;
            margin: 10px;
            font-size: 16px;
            border-radius: 5px;
            border: 1px solid #ddd;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Biblia Comparativa Online</h1>
        
        <div>
            <label for="bookSelect">Libro:</label>
            <select id="bookSelect" onchange="updateChapters()">
                <option value="">Libro</option>
            </select>
        </div>

        <div>
            <label for="chapterSelect">Capítulo:</label>
            <select id="chapterSelect" onchange="updateVerses()">
                <option value="">Capítulo</option>
            </select>
        </div>

        <div>
            <label for="verseSelect">Versículo:</label>
            <select id="verseSelect">
                <option value="">Versículo</option>
            </select>
        </div>

        <button onclick="searchVerse()">Buscar</button>

        <div id="output"></div>
    </div>

    <script>
        let allVerses = [];  // Array para almacenar todos los versículos

        function loadBible() {
            const xmlUrl = 'https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/Reina-Valera%2060.xml';

            fetch(xmlUrl)
                .then(response => {
                    if (!response.ok) {
                        throw new Error('Error al cargar el archivo XML');
                    }
                    return response.text();
                })
                .then(xmlText => {
                    const parser = new DOMParser();
                    const xmlDoc = parser.parseFromString(xmlText, "application/xml");

                    allVerses = [];
                    const books = xmlDoc.getElementsByTagName('b');
                    const bookSelect = document.getElementById('bookSelect');
                    bookSelect.innerHTML = '<option value="">Libro</option>'; // Limpiar opciones

                    for (let b = 0; b < books.length; b++) {
                        const book = books[b];
                        const bookName = book.getAttribute('n');
                        const option = document.createElement('option');
                        option.value = bookName;
                        option.textContent = bookName;
                        bookSelect.appendChild(option);

                        const chapters = book.getElementsByTagName('c');
                        const chapterSelect = document.getElementById('chapterSelect');
                        chapterSelect.innerHTML = '<option value="">Capítulo</option>'; // Limpiar capítulos

                        for (let c = 0; c < chapters.length; c++) {
                            const chapter = chapters[c];
                            const chapterNumber = chapter.getAttribute('n');
                            const verseSelect = document.getElementById('verseSelect');
                            verseSelect.innerHTML = '<option value="">Versículo</option>'; // Limpiar versículos

                            const verses = chapter.getElementsByTagName('v');
                            for (let v = 0; v < verses.length; v++) {
                                const verse = verses[v];
                                const verseNumber = verse.getAttribute('n');
                                const verseText = verse.textContent.trim();

                                allVerses.push({
                                    book: bookName,
                                    chapter: chapterNumber,
                                    verse: verseNumber,
                                    text: verseText
                                });

                                const verseOption = document.createElement('option');
                                verseOption.value = `${bookName}-${chapterNumber}-${verseNumber}`;
                                verseOption.textContent = `${verseNumber} - ${verseText}`;
                                verseSelect.appendChild(verseOption);
                            }
                        }
                    }
                })
                .catch(error => {
                    document.getElementById('output').textContent = 'Ocurrió un error: ' + error.message;
                });
        }

        function updateChapters() {
            const selectedBook = document.getElementById('bookSelect').value;
            const chapterSelect = document.getElementById('chapterSelect');
            chapterSelect.innerHTML = '<option value="">Capítulo</option>'; // Limpiar capítulos

            if (selectedBook) {
                const book = allVerses.filter(verse => verse.book === selectedBook);
                const chapters = [...new Set(book.map(verse => verse.chapter))];

                chapters.forEach(chapter => {
                    const option = document.createElement('option');
                    option.value = chapter;
                    option.textContent = chapter;
                    chapterSelect.appendChild(option);
                });
            }
        }

        function updateVerses() {
            const selectedBook = document.getElementById('bookSelect').value;
            const selectedChapter = document.getElementById('chapterSelect').value;
            const verseSelect = document.getElementById('verseSelect');
            verseSelect.innerHTML = '<option value="">Versículo</option>'; // Limpiar versículos

            if (selectedBook && selectedChapter) {
                const verses = allVerses.filter(verse => verse.book === selectedBook && verse.chapter === selectedChapter);
                verses.forEach(verse => {
                    const option = document.createElement('option');
                    option.value = `${verse.book}-${verse.chapter}-${verse.verse}`;
                    option.textContent = verse.verse; // Solo mostrar el número del versículo
                    verseSelect.appendChild(option);
                });
            }
        }

        function searchVerse() {
            const selectedVerse = document.getElementById('verseSelect').value;
            if (selectedVerse) {
                const verseDetails = allVerses.find(verse => `${verse.book}-${verse.chapter}-${verse.verse}` === selectedVerse);
                document.getElementById('output').innerHTML = `<strong>${verseDetails.book} ${verseDetails.chapter}:${verseDetails.verse}</strong> - ${verseDetails.text}`;
            } else {
                document.getElementById('output').innerHTML = 'Por favor selecciona un versículo.';
            }
        }

        // Cargar los datos de la Biblia al cargar la página
        window.onload = loadBible;
    </script>

</body>
</html>
