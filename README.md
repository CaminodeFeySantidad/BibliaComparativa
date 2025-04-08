# BibliaComparativa 2
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Biblia Comparativa</title>
    <style>
        /* Estilo para aumentar el tamaño de las listas desplegables */
        select {
            font-size: 18px;  /* Aumenta el tamaño de la fuente */
            padding: 10px;    /* Aumenta el espacio dentro del select */
            margin: 5px;      /* Añade un margen alrededor del select */
            border-radius: 5px;  /* Añade bordes redondeados */
            border: 2px solid #ccc; /* Borde suave */
        }

        /* Estilo para los botones */
        button {
            font-size: 20px;  /* Aumenta el tamaño de la fuente */
            padding: 10px 20px; /* Aumenta el espacio alrededor del texto */
            margin: 5px;      /* Añade un margen alrededor del botón */
            cursor: pointer; /* Cambia el cursor al pasar por encima */
            border-radius: 5px;  /* Bordes redondeados */
            border: 2px solid #ccc; /* Borde suave */
            background-color: #f0f0f0; /* Fondo claro */
        }

        button:hover {
            background-color: #dcdcdc; /* Color de fondo cuando el botón es hovered */
        }

        /* Estilo para los títulos de cada versión de la Biblia (más pequeños que el texto bíblico) */
        h2 {
            font-size: 16px;  /* Tamaño de fuente más pequeño para los títulos de las versiones de la Biblia */
            margin-top: 20px;
        }

        /* Estilo para los textos bíblicos (cambiado a 18px) */
        p {
            font-size: 18px;  /* Tamaño de fuente para los versículos (ahora 18px) */
            line-height: 1.6; /* Espaciado entre líneas */
        }

        /* Estilo para los labels y selects para mantener todo alineado */
        label {
            font-size: 16px; /* Tamaño adecuado para las etiquetas */
            margin-right: 10px; /* Espacio entre la etiqueta y el select */
        }

        /* Espaciado entre las listas */
        br {
            clear: both; /* Asegura que las listas se alineen correctamente */
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
                { id: "https://github.com/caminodefeysantidad/BibliaComparativa/raw/main/Reina-Valera%2060.xml", elemento: "biblia1", nombre: "Reina-Valera 1960" },
                { id: "https://github.com/caminodefeysantidad/BibliaComparativa/raw/main/NTV.xml", elemento: "biblia2", nombre: "NTV" },
                { id: "https://github.com/caminodefeysantidad/BibliaComparativa/raw/main/Biblia%20Latinoamericana%2095.xml", elemento: "biblia3", nombre: "Biblia Latinoamericana 95" },
                { id: "https://github.com/caminodefeysantidad/BibliaComparativa/raw/main/NVI.xml", elemento: "biblia4", nombre: "NVI" },
                { id: "https://github.com/caminodefeysantidad/BibliaComparativa/raw/main/LBLA.xml", elemento: "biblia5", nombre: "LBLA" },
                { id: "https://github.com/caminodefeysantidad/BibliaComparativa/raw/main/DHH.xml", elemento: "biblia6", nombre: "DHH" }
            ];
            try {
                for (const version of versiones) {
                    const response = await fetch(version.id);
                    const text = await response.text();
                    const xml = new DOMParser().parseFromString(text, "application/xml");
                    biblias[version.elemento] = procesarXML(xml);
                }
                actualizarSelectLibros();
            } catch (error) {
                versiones.forEach(v => document.getElementById(v.elemento).innerText = "Error al cargar la Biblia.");
            }
        }

        function procesarXML(xml) {
            let data = {};
            const biblia = xml.getElementsByTagName("b");
            for (let b of biblia) {
                const libro = b.getAttribute("n");
                if (!libros.includes(libro)) libros.push(libro);
                data[libro] = {};
                for (let c of b.getElementsByTagName("c")) {
                    const numCapitulo = c.getAttribute("n");
                    data[libro][numCapitulo] = Array.from(c.getElementsByTagName("v"), v => v.textContent);
                }
            }
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
