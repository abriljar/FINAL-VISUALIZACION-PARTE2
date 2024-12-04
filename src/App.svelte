
  <!-- Script JS -->
  <script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    let width = 400;
    let height = 400;

    let songData = {}; // Mapa con los datos de las canciones por nombre
    let leftSongData = null; // Datos de la canción seleccionada del scroll izquierdo
    let rightSongData = null; // Datos de la canción seleccionada del scroll derecho

    const colors = {
        left: "#d90263",
        right: "#F98B0B",
    };

    onMount(async () => {
        const csvData = await d3.csv("/data/Spotify.csv");

        // Crear mapa de canciones por nombre
        songData = csvData.reduce((acc, song) => {
            acc[song.Nombre] = {
                Usuario: song.Usuario,
                Artista: song.Artista,
                Genero: song.Genero,
                Acustica: +song.Acustica,
                Bailabilidad: +song.Bailabilidad,
                Energy: song.Energy.trim(),
                Instrumentalidad: +song.Instrumentalidad,
                Vivacidad: +song.Vivacidad,
                Palabras: +song.Palabras,
                Valencia: +song.Valencia,
            };
            return acc;
        }, {});

        // Inicializar con las dos primeras canciones
        leftSongData = Object.values(songData).find(song => song.Usuario === "Abril");
        rightSongData = Object.values(songData).find(song => song.Usuario === "Martin");

        // Dibujar el gráfico inicial
        createRadarChart(leftSongData, rightSongData);

        // Asignar eventos de clic dinámicamente
        assignClickEvents();
    });

    function assignClickEvents() {
        // Seleccionar todas las canciones del contenedor
        document.querySelectorAll(".Abril-container .disco-container").forEach(container => {
            const songId = container.getAttribute("data-song-id");
            container.addEventListener("click", () => handleLeftSongClick(songId));
        });

        document.querySelectorAll(".Martin-container .disco-container").forEach(container => {
            const songId = container.getAttribute("data-song-id");
            container.addEventListener("click", () => handleRightSongClick(songId));
        });

        console.log("Eventos asignados correctamente.");
    }

    function handleLeftSongClick(songId) {
        leftSongData = songData[songId];
        console.log("Canción seleccionada de Abril:", leftSongData);
        createRadarChart(leftSongData, rightSongData);
    }

    function handleRightSongClick(songId) {
        rightSongData = songData[songId];
        console.log("Canción seleccionada de Martin:", rightSongData);
        createRadarChart(leftSongData, rightSongData);
    }

    function createRadarChart(leftData, rightData) {
        d3.select("#radar-chart").selectAll("*").remove(); // Limpiar gráfico existente

        const relevantKeys = ["Valencia", "Instrumentalidad", "Vivacidad", "Acustica", "Bailabilidad", "Palabras"];
        const radius = Math.min(width, height) / 1.1 - 50;
        const angleSlice = (Math.PI * 2) / relevantKeys.length;
        const radialScale = d3.scaleLinear().domain([0, 1]).range([0, radius]);

        const svg = d3
            .select("#radar-chart")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${width / 2}, ${height / 2})`);

        // Dibujar círculos concéntricos
        for (let i = 1; i <= 5; i++) {
            svg.append("circle")
                .attr("r", (radius / 5) * i)
                .style("fill", "none")
                .style("stroke", "#ccc")
                .style("opacity", 0.5);
        }

        // Dibujar líneas radiales y etiquetas
        relevantKeys.forEach((key, i) => {
            const angle = angleSlice * i - Math.PI / 2;
            svg.append("line")
                .attr("x1", 0)
                .attr("y1", 0)
                .attr("x2", radialScale(1) * Math.cos(angle))
                .attr("y2", radialScale(1) * Math.sin(angle))
                .style("stroke", "#ccc")
                .style("opacity", 0.5);

            svg.append("text")
                .attr("x", radialScale(1.2) * Math.cos(angle))
                .attr("y", radialScale(1.1) * Math.sin(angle))
                .style("text-anchor", "middle")
                .style("font-size", "20px")
				.style("font-family", "Bebas Neue")
                .style("fill", "white")
                .text(key);
        });

        // Dibujar datos de la canción izquierda
        if (leftData) {
            drawSongData(svg, leftData, relevantKeys, radialScale, angleSlice, colors.left);
        }

        // Dibujar datos de la canción derecha
        if (rightData) {
            drawSongData(svg, rightData, relevantKeys, radialScale, angleSlice, colors.right);
        }
    }

    function drawSongData(svg, data, keys, radialScale, angleSlice, color) {
        const points = keys.map((key, i) => {
            const angle = angleSlice * i - Math.PI / 2;
            const x = radialScale(data[key]) * Math.cos(angle);
            const y = radialScale(data[key]) * Math.sin(angle);
            return [x, y];
        });

        // Dibujar el área del radar
        svg.append("polygon")
            .attr("points", points.map(d => d.join(",")).join(" "))
            .style("fill", color)
            .style("opacity", 0.4)
            .style("stroke", color)
            .style("stroke-width", 2);

        // Dibujar puntos
        points.forEach(([x, y]) => {
            svg.append("circle")
                .attr("cx", x)
                .attr("cy", y)
                .attr("r", 4)
                .style("fill", color);
        });
    }
</script>




<!-- Estructura contenido HTML -->
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family:wght@400&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://use.typekit.net/zsm5fov.css">


<main>

	<a href="https://final-visualizacion-abril.vercel.app/" class="boton2">VOLVER ATRÁS</a>

	<h1>Entre discos y datos: <br> sintonías que nos conectan</h1>
	<h4>Navegá por las canciones más escuchadas por Abril y Martín en el 2023.<br>Al hacer click en un disco, se grafican diferentes atributos musicales como acústica, bailabilidad, vivacidad, y más.</h4>
	


	<div class = "scroll-wrapper">
	<div class="image-container">
		<h3 class="scroll-title">Abril</h3>
		<div class="Abril-container">
			<div class="disco-container" data-song-id="La Bachata" onclick="handleDiscoClick(event)"> 
				<img src="/images/Abril/bachata cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Abril/bachata abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			<div class="disco-container" data-song-id="Milagrosa" onclick="handleDiscoClick(event)">
				<img src="/images/Abril/milagrosa cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Abril/milagrosa abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			<div class="disco-container"  data-song-id="Extasis" onclick="handleDiscoClick(event)">
				<img src="/images/Abril/extasis cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Abril/extasis abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			<div class="disco-container"data-song-id="Rara vez" onclick="handleDiscoClick(event)">
				<img src="/images/Abril/rara vez cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Abril/rara vez abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			<div class="disco-container" data-song-id="Baila Bach" onclick="handleDiscoClick(event)">
				<img src="/images/Abril/baila bach cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Abril/baila bach abierto.png" alt="Disco abierto" class="disco-hover">
		  	</div>
			  
		</div>

		
	</div>

	<div class="image-container">
		<h3 class="scroll-title">Martín</h3>
		<div class="Martin-container">
			<div class="disco-container" data-song-id="Bien o Mal" onclick="handleDiscoClick(event)"> 
				<img src="/images/Martin/Bien o mal cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Martin/Bien o mal abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			<div class="disco-container"  data-song-id="Inmortal" onclick="handleDiscoClick(event)">
				<img src="/images/Martin/inmortal cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Martin/inmortal abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			<div class="disco-container" data-song-id="Hood" onclick="handleDiscoClick(event)">
				<img src="/images/Martin/Hood cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Martin/Hood abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			<div class="disco-container" data-song-id="M.A" onclick="handleDiscoClick(event)">
				<img src="/images/Martin/MA cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Martin/MA abierto.png" alt="Disco abierto" class="disco-hover">
		  	</div>
			<div class="disco-container"data-song-id="Lo Tengo" onclick="handleDiscoClick(event)">
				<img src="/images/Martin/Lo tengo cerrado.png" alt="Disco cerrado" class="disco">
				<img src="/images/Martin/Lo tengo abierto.png" alt="Disco abierto" class="disco-hover">
			</div>
			
		</div>
	</div>

		

	</div>

	<br>
	<br>
	<br>
	<br>
	<br>
	<br>
	<br>
	<svg id="radar-chart"></svg>
	
	
	

	<img src="/images/tira.gif" alt="tira" class="tira">
	<h1>Nuestro perfil musical: Abril vs. Martín</h1>
	<h4>Explorá nuestros géneros favoritos, artistas más escuchados y descubrí cómo se diferencian nuestros gustos.
	<br> Una mirada general a lo que nos hace únicos a través de la música.</h4>

	<div class="legend">
		<div class="legend-item">
			<span class="legend-square" style="background-color: #d90263;"></span>
			<span class="legend-text">Abril</span>
		</div>
		<div class="legend-item">
			<span class="legend-square" style="background-color: #F98B0B;"></span>
			<span class="legend-text">Martín</span>
		</div>
	</div>


	<div class = "graficos">
		<div class = "laterales">
			<div class="grafico-individual">
				<h3>Total de géneros escuchados</h3> 
				<p>Entre Martín y Abril exploramos 63 géneros en total, aunque la lista de Abril incluye 42, mostrando un recorrido más amplio por la música.</p>
				<div class="flourish-embed flourish-pictogram" data-src="visualisation/20490378"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20490378/thumbnail" width="100%" alt="pictogram visualization" /></noscript></div>
			</div>

			<div class = "grafico-individual">
				<h3>Comparación de géneros</h3>
				<p>¿Qué géneros dominan nuestra música? Comparamos nuestras preferencias musicales, revelando cómo se distribuyen nuestros gustos entre distintos géneros y estilos.</p>
				<div class="flourish-embed flourish-chart" data-src="visualisation/20508177"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20508177/thumbnail" width="100%" alt="chart visualization" /></noscript></div>
			</div>

			<div class = "grafico-individual">
				<h3>Total de artistas escuchados</h3>
				<p>El recorrido musical de Abril incluyó 1032 artistas, más del doble de los 466 que escuchó Martín, reflejando nuestras distintas formas de explorar la música.</p>
				<div class="flourish-embed flourish-pictogram" data-src="visualisation/20491512"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20491512/thumbnail" width="100%" alt="pictogram visualization" /></noscript></div>
			</div>
		</div>
		

		<div class = "grafico-individual acoustic-graph">
			<h3>Acústica de las canciones</h3> 
			<p>En este gráfico decidimos centrarnos en la variable acústica de nuestras canciones favoritas, ya que fue la que mostró la mayor diferencia en el gráfico de red.</p>
			<div class="flourish-embed flourish-chart" data-src="visualisation/20508444"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20508444/thumbnail" width="100%" alt="chart visualization" /></noscript></div>
		</div>
	
		<div class = "burbujas">
			<h3>Top 5 artistas y sus géneros</h3>
			<p>Este gráfico de burbujas muestra nuestros Top 5 artistas y los géneros a los que pertenecen, <br> destacando las diferencias y similitudes entre las preferencias musicales de Martín y Abril.</p>

			<div class="graficos-burbujas">
				<div class="grafico-individual-2">
				<span class="legend-text2">Abril</span>
				<div class="flourish-embed flourish-bubble-chart" data-src="visualisation/20489860"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20489860/thumbnail" width="100%" alt="bubble-chart visualization" /></noscript></div>
			</div>
			<div class="grafico-individual-2">
				<span class="legend-text2">Martín</span>
				<div class="flourish-embed flourish-bubble-chart" data-src="visualisation/20508044"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20508044/thumbnail" width="100%" alt="bubble-chart visualization" /></noscript></div>
			</div>
		</div>
	</div>


	
</main>
<footer> Abril Jarque    |    Martín De Luca
	<br> <br>Trabajo Final   |   Visualización de Datos 2024
</footer>

<!-- Estilos CSS -->

<style>

footer{
		font-family: Bebas Neue;
		font-size: 20px;
		background-color: #6a2fb8;
		color: white;
		position: absolute; 
		left: 0;
		width: 100vw; 
		height: 15vh; 
		text-align: center;
		margin-top: 50px;
		display: flex;
		justify-content: center;
		align-items: center;
	}

.boton2{
	font-family: Remora Sans W5;
	font-weight: 250;
	font-size: 10px;
	background-color: #9747FF;
	color: white;
	border-radius: 20px;
	cursor: pointer;
	padding: 10px 20px;
	transition: background-color 0.3s ease;
	margin: 20px;
	position: absolute;
	top: 20px; /* Distancia desde la parte superior */
    left: 20px;
	text-decoration: none;
}

.boton2:hover {
	background-color: #6a2fb8;
}

.scroll-wrapper {
    display: flex;
    gap: 50px;
    width: 100%;
    justify-content: center; /* Centra los contenedores horizontalmente */
    align-items: flex-start; /* Asegura que todo se alinee desde la parte superior */
}

.image-container {
    display: flex;
    flex-direction: column;
    align-items: center; 
	justify-content: center;
	margin-top: 20px; 
	gap: 20px;
}



.scroll-title {
    font-family: Bebas Neue, sans-serif;
    font-size: 24px;
    color: white;
    text-align: center;
    margin: 0; /* Elimina márgenes adicionales */
}
	.legend-text2{
		display: block; 
    text-align: center;
    font-family: Bebas Neue, sans-serif;
    font-size: 20px;
    color: white; 
    margin-bottom: 10px;
	}

	.grafico-individual-2 {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    max-width: 400px;
    width: 100%;
}

	.legend {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 50px;
    gap: 20px; 
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 10px; 
	}

	.legend-square {
		width: 20px;
		height: 20px;
		border-radius: 4px;
		display: inline-block;
	}

	.legend-text {
		font-family: Bebas Neue;
		font-size: 20px;
		color: white;
	}

	.burbujas {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    margin: 30px auto;
    gap: 20px;
    width: 90%;
    max-width: 1000px;
}

.burbujas h3 {
    margin-bottom: 5px;
}

.burbujas p {
    margin-top: 0;
    margin-bottom: 15px;
}

.graficos-burbujas {
    display: flex;
    justify-content: center;
    gap: 20px;
    width: 100%;
    flex-wrap: wrap; 
}

.graficos-burbujas .flourish-embed {
    flex: 1; 
    max-width: 400px;
    min-width: 200px;
    height: auto;
}

	.laterales{
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 30px;
		width: 100%;
		justify-content: center; 
	}

	.grafico-individual {
		text-align: center;
		padding: 10px;
		max-width: 400px; 
		width: 100%;
	}

	.acoustic-graph {
		display: flex;
    	flex-direction: column;
    	justify-content: center;
    	align-items: center;
    	margin: 10px auto;
    	width: 80%;
    	max-width: 800px;
    	text-align: center;
    	gap: 10px;
	}

	.acoustic-graph h3 {
    margin-bottom: 5px;
	}

	.acoustic-graph p {
		margin-top: 0; 
		margin-bottom: 10px;
	}
		

	.flourish-embed {
		width: 100%;
		height: auto;
	}

	.tira{
	width: auto;
	margin-bottom: -230px;	
	}

	.scroll-wrapper{
		display: flex; 
    	gap: 50px; 
    	width: 100%; 
    	justify-content: center; 
	}

	main {
		margin: 60px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background: black;
		min-height: 100vh;
		color: white;
	}

	h1{
		font-family: Remora Sans W5;
		margin: 0; 
		text-align: center;
		font-size: 40px;

	}

	h3{
		margin-top: 80px;
		font-size: 20px;
	}
	
	h4 {
		font-family: Helvetica Neue LT Std;
		font-weight: lighter;
		text-align: center;
		font-size: 20px;
	}

	p{
		font-family: Helvetica Neue LT Std;
		font-weight: lighter;
		text-align: center;
		font-size: 17px;
	}

	#radar-chart {
		background: black;
		margin-top: 100px;
		overflow: visible;
	}
	

	.Abril-container {
		width: 350px;
		height: 350px; 
     	overflow-y: visible; 
    	overflow-x: hidden;
    	margin: 0 auto;
    	position: relative; 
		padding: 20px; 
	}

	.Martin-container {
		width: 350px;
		height: 350px; 
     	overflow-y: visible; 
    	overflow-x: hidden;
    	margin: 0 auto;
    	position: relative; 
		padding: 20px; 
	}

	.Abril-container::-webkit-scrollbar {
		display: none;
	}

	.Martin-container::-webkit-scrollbar {
		display: none;
	}

	.disco-container {
		position: relative;
		z-index: 1; 
		transition: transform 0.2s ease, z-index 0.2s ease;
	}

	.disco-container img {
		width: 100%; 
		cursor: pointer;
		transition: transform 0.3s ease;
	}

	.disco-container:hover {
		transform: scale(1.1);
		box-shadow: 0px 0px 15px #9747FF; 
		z-index: 10;
	}

	.disco {
		width: 193px;
		height: 200px; 
		object-fit: contain;
		transition: transform 0.3s ease, box-shadow 0.3s ease; 
	}

	.disco-hover {
		display: none;
	}

	.disco-container:hover .disco {
		display: none; 
	}

	.disco-container:hover .disco-hover {
		display: block;
	}

</style>
