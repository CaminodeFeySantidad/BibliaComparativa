# BibliaComparativa
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Biblia Comparativa</title>
    <style>
        select {
            font-size: 18px;
            padding: 10px;
            margin: 5px;
            border-radius: 5px;
            border: 2px solid #ccc;
        }

        button {
            font-size: 20px;
            padding: 10px 20px;
            margin: 5px;
            cursor: pointer;
            border-radius: 5px;
            border: 2px solid #ccc;
            background-color: #f0f0f0;
        }

        button:hover {
            background-color: #dcdcdc;
        }

        h2 {
            font-size: 16px;
            margin-top: 20px;
        }

        p {
            font-size: 18px;
            line-height: 1.6;
        }

        label {
            font-size: 16px;
            margin-right: 10px;
        }

        br {
            clear: both;
        }
    </style>
    <script>
        let libros = [];
        let libroActual = "Génesis";
        let capituloActual = "1";
        let versiculoActual = 1;
        let biblias = {};

        async function cargarBiblia() {
            const versiones = [
                { id: "Reina-Valera 60.html", elemento: "biblia1", nombre: "Reina-Valera 1960" },
                { id: "Latinoamericana 95.html", elemento: "biblia2", nombre: "NTV" },
                { id: "NTV.html", elemento: "biblia3", nombre: "Biblia Latinoamericana 95" },
                { id: "NVI.html", elemento: "biblia4", nombre: "NVI" },
                { id: "LBLA.html", elemento: "biblia5", nombre: "LBLA" },
                { id: "DHH.html", elemento: "biblia6", nombre: "DHH" }
            ];

            try {
                for (const version of versiones) {
                    const response = await fetch(version.id);
                    const text = await response.text();
                    const html = new DOMParser().parseFromString(text, "text/html");
                    biblias[version.elemento] = procesarHTML(html);
                }
                actualizarSelectLibros();
            } catch (error) {
                versiones.forEach(v => document.getElementById(v.elemento).innerText = "Error al cargar la Biblia.");
            }
        }

        function procesarHTML(html) {
            let data = {};
            const librosHTML = html.querySelectorAll('b');
            librosHTML.forEach(b => {
                const libro = b.getAttribute("n");
                if (!libros.includes(libro)) libros.push(libro);
                data[libro] = {};
                const capitulos = b.querySelectorAll('c');
                capitulos.forEach(c => {
                    const numCapitulo = c.getAttribute("n");
                    const versiculos = Array.from(c.querySelectorAll('v')).map(v => v.textContent);
                    data[libro][numCapitulo] = versiculos;
                });
            });
            return data;
        }

        function actualizarSelectLibros() {
            const selectLibro = document.getElementById("selectLibro");
            selectLibro.innerHTML = libros.map(libro => `<option value="${libro}">${libro}</option>`).join("");
            selectLibro.value = libroActual;
            actualizarSelectCapitulos();
        }

        function actualizarSelectCapitulos() {
            const selectCapitulo = document.getElementById("selectCapitulo");
            selectCapitulo.innerHTML = Object.keys(biblias["biblia1"][libroActual] || {}).map(num => `<option value="${num}">${num}</option>`).join("");
            selectCapitulo.value = capituloActual;
            actualizarSelectVersiculos();
        }

        function actualizarSelectVersiculos() {
            const selectVersiculo = document.getElementById("selectVersiculo");
            selectVersiculo.innerHTML = (biblias["biblia1"][libroActual]?.[capituloActual] || []).map((_, index) => `<option value="${index + 1}">${index + 1}</option>`).join("");
            selectVersiculo.value = versiculoActual;
            actualizarTextoTodasVersiones();
        }

        function actualizarTextoTodasVersiones() {
            for (const [key, data] of Object.entries(biblias)) {
                document.getElementById(key).innerText = data[libroActual]?.[capituloActual]?.[versiculoActual - 1] || "";
            }
        }

        function cambiarLibro(event) {
            libroActual = event.target.value;
            actualizarSelectCapitulos();
        }

        function cambiarCapitulo(event) {
            capituloActual = event.target.value;
            actualizarSelectVersiculos();
        }

        function cambiarVersiculo(event) {
            versiculoActual = parseInt(event.target.value);
            actualizarTextoTodasVersiones();
        }

        function cambiarVersiculoBoton(direccion) {
            const maxVersiculos = biblias["biblia1"][libroActual][capituloActual]?.length || 1;
            versiculoActual = Math.max(1, Math.min(versiculoActual + direccion, maxVersiculos));
            document.getElementById("selectVersiculo").value = versiculoActual;
            actualizarTextoTodasVersiones();
        }

        window.onload = cargarBiblia;
    </script>
</head>
<body>
    <label for="selectLibro">Libro:</label>
    <select id="selectLibro" onchange="cambiarLibro(event)"></select>
    <br>
    <label for="selectCapitulo">Capítulo:</label>
    <select id="selectCapitulo" onchange="cambiarCapitulo(event)"></select>
    <br>
    <label>Versículo:</label>
    <button onclick="cambiarVersiculoBoton(-1)">-</button>
    <select id="selectVersiculo" onchange="cambiarVersiculo(event)"></select>
    <button onclick="cambiarVersiculoBoton(1)">+</button>
    <hr>
    <h2>Reina-Valera 1960</h2>
    <p id="biblia1">Cargando...</p>
    <hr>
    <h2>NTV</h2>
    <p id="biblia2">Cargando...</p>
    <hr>
    <h2>Biblia Latinoamericana 95</h2>
    <p id="biblia3">Cargando...</p>
    <hr>
    <h2>NVI</h2>
    <p id="biblia4">Cargando...</p>
    <hr>
    <h2>LBLA</h2>
    <p id="biblia5">Cargando...</p>
    <hr>
    <h2>DHH</h2>
    <p id="biblia6">Cargando...</p>
</body>
</html>
