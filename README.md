<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Biblia Comparativa con Buscador Automático</title>
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
        select {
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
        <h1>Buscar Versículos en la Biblia</h1>
        
        <div>
            <label for="bookSelect">Selecciona un Libro:</label>
            <select id="bookSelect" onchange="updateChapters()">
                <option value="">Seleccionar Libro</option>
            </select>
        </div>

        <div>
            <label for="chapterSelect">Selecciona un Capítulo:</label>
            <select id="chapterSelect" onchange="updateVerses()">
                <option value="">Seleccionar Capítulo</option>
            </select>
        </div>

        <div>
            <label for="verseSelect">Selecciona un Versículo:</label>
            <select id="verseSelect" onchange="showVerse()">
                <option value="">Seleccionar Versículo</option>
            </select>
        </div>

        <div id="output"></div>
    </div>

    <script>
        let bibleData = []; // Array para almacenar todos los versículos

        // Cargar la Biblia desde un archivo XML
        function loadBible() {
            fetch('https://raw.githubusercontent.com/caminodefeysantidad/BibliaComparativa/main/Reina-Valera%2060.xml')
                .then(response => {
                    if (!response.ok) {
                        throw new Error('Error al cargar el archivo XML');
                    }
                    return response.text();
                })
                .then(xmlText => {
                    const parser = new DOMParser();
                    const xmlDoc = parser.parseFromString(xmlText, "application/xml");
                    const books = xmlDoc.getElementsByTagName('b');

                    bibleData = [];
                    const bookSelect = document.getElementById('bookSelect');
                    bookSelect.innerHTML = '<option value="">Seleccionar Libro</option>'; // Limpiar opciones de libro

                    // Iterar sobre los libros
                    for (let b = 0; b < books.length; b++) {
                        const book = books[b];
                        const bookName = book.getAttribute('n');
                        const bookOption = document.createElement('option');
                        bookOption.value = bookName;
                        bookOption.textContent = bookName;
                        bookSelect.appendChild(bookOption);

                        const chapters = book.getElementsByTagName('c');
                        // Iterar sobre los capítulos
                        for (let c = 0; c < chapters.length; c++) {
                            const chapter = chapters[c];
                            const chapterNumber = chapter.getAttribute('n');

                            const verses = chapter.getElementsByTagName('v');
                            // Iterar sobre los versículos
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
                })
                .catch(error => {
                    document.getElementById('output').textContent = 'Ocurrió un error: ' + error.message;
                });
        }

        // Actualizar los capítulos según el libro seleccionado
        function updateChapters() {
            const selectedBook = document.getElementById('bookSelect').value;
            const chapterSelect = document.getElementById('chapterSelect');
            chapterSelect.innerHTML = '<option value="">Seleccionar Capítulo</option>'; // Limpiar capítulos
            if (selectedBook) {
                const chapters = [...new Set(bibleData.filter(verse => verse.book === selectedBook).map(verse => verse.chapter))];
                chapters.forEach(chapter => {
                    const option = document.createElement('option');
                    option.value = chapter;
                    option.textContent = chapter;
                    chapterSelect.appendChild(option);
                });
            }
        }

        // Actualizar los versículos según el libro y capítulo seleccionados
        function updateVerses() {
            const selectedBook = document.getElementById('bookSelect').value;
            const selectedChapter = document.getElementById('chapterSelect').value;
            const verseSelect = document.getElementById('verseSelect');
            verseSelect.innerHTML = '<option value="">Seleccionar Versículo</option>'; // Limpiar versículos
            if (selectedBook && selectedChapter) {
                const verses = bibleData.filter(verse => verse.book === selectedBook && verse.chapter === selectedChapter);
                verses.forEach(verse => {
                    const option = document.createElement('option');
                    option.value = `${verse.book}-${verse.chapter}-${verse.verse}`;
                    option.textContent = `${verse.verse} - ${verse.text}`;
                    verseSelect.appendChild(option);
                });
            }
        }

        // Mostrar el versículo seleccionado
        function showVerse() {
            const selectedVerse = document.getElementById('verseSelect').value;
            if (selectedVerse) {
                const verseDetails = bibleData.find(verse => `${verse.book}-${verse.chapter}-${verse.verse}` === selectedVerse);
                document.getElementById('output').innerHTML = `<strong>${verseDetails.book} ${verseDetails.chapter}:${verseDetails.verse}</strong> - ${verseDetails.text}`;
            } else {
                document.getElementById('output').innerHTML = 'Por favor selecciona un versículo.';
            }
        }

        // Cargar la Biblia al cargar la página
        window.onload = loadBible;
    </script>

</body>
</html>
