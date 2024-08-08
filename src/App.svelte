<script>
  import data from "$data/data.js";
  import { fade } from "svelte/transition";
  import Tooltip from "$components/Tooltip.svelte";

  let width = 460,
    height = 975;
  
  const margin = {
    top: 25,
    right: 0,
    bottom: 10,
    left: 10
  }

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

// Función para extraer el año de la cadena
function extractYear(event) {
  // Suponiendo que el año está al final de la cadena
  return parseInt(event.match(/\d+$/)[0]);
}

// Función para extraer el lugar de la cadena
function extractLocation(event) {
  // Suponiendo que el lugar está antes del año
  return event.replace(/\d+$/, '').trim();
}

// Función para agrupar medallas por año y lugar
function groupByYear(dataExample) {
  const groupedData = {};

  dataExample.forEach(item => {
    const year = extractYear(item.Año);
    const location = extractLocation(item.Año);

    // Crea la clave del grupo usando el lugar y el año
    const key = `${location} ${year}`;

    // Inicializa el array para la clave si no existe
    if (!groupedData[key]) {
      groupedData[key] = [];
    }

    // Agrega la medalla junto con los campos adicionales al array de la clave correspondiente
    groupedData[key].push({
      type: item.Medalla,
      deporte: item.Deporte,
      nombre: item.Nombre
    });
  });

  // Ordena las claves por año descendente y crea el array final
  return Object.keys(groupedData)
    .sort((b, a) => {
      const yearA = extractYear(a);
      const yearB = extractYear(b);
      return yearB - yearA; // Ordenar por año descendente
    })
    .map(key => ({
        year: key,
        medals: groupedData[key]
          .sort((a, b) => {
            // Ordenar por tipo de medalla: Oro > Plata > Bronce
            const order = { "Oro": 1, "Plata": 2, "Bronce": 3 };
            const typeComparison = order[a.type] - order[b.type];
            
            if (typeComparison !== 0) {
              return typeComparison;
            }
            
            // Si el tipo de medalla es el mismo, ordenar por deporte alfabéticamente
            if (a.deporte < b.deporte) return -1;
            if (a.deporte > b.deporte) return 1;
            
            return 0;
          })
      }));
}

  const dataAgrupada = groupByYear(data);

  const colorRange = {
    "Oro": "gold",
    "Plata": "silver",
    "Bronce": "peru",
  };

  // Extraer lista de deportes únicos y contar medallas por deporte
  const deportes = [...new Set(data.map(item => item.Deporte))];

  // Contar medallas por deporte
  const medallasPorDeporte = deportes.reduce((acc, deporte) => {
    acc[deporte] = data.filter(item => item.Deporte === deporte).length;
    return acc;
  }, {});

  let selectedDeporte = "Todos"; // Valor por defecto para mostrar todas las medallas
  
  let deportesOrdenados = deportes
  .map(deporte => ({ deporte, count: medallasPorDeporte[deporte] }))
  .sort((a, b) => b.count - a.count);

  let hovered;


  let tooltipX = 0;
  let tooltipY = 0;

</script>

<h1>El medallero olímpico español</h1>
<div class="filter-container">
  <label for="deporte-select">Filtra por deporte:</label>
  <select id="deporte-select" bind:value={selectedDeporte}>
    <option value="Todos">Todos</option>
    {#each deportesOrdenados as deporte}
      <option value={deporte.deporte}>
        {deporte.deporte} ({medallasPorDeporte[deporte.deporte]})
      </option>
    {/each}
  </select>
</div>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class='chart-container' bind:clientWidth={width}
on:click={() => {
  selection = null;
}}>
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <svg {width} {height} on:click|stopPropagation>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
      {#each dataAgrupada as edition, index}
        <!-- Añadir una fila para cada edición -->
        <g transform="translate(0, {index * 46})">
          <g class="ediciones">
            <text class="year" x="0" y="0">
              {edition.year}
              <tspan class="medal-count" dx="5">
                {edition.medals.length > 2 ? `(${edition.medals.length})` : ''}
              </tspan>
            </text>          
          </g>
          <!-- Añadir medallas -->
          {#each edition.medals as medal, medalIndex}
            <!-- svelte-ignore a11y-mouse-events-have-key-events -->
            <circle
              cx={width < 568 ? medalIndex * 16 + 8 : medalIndex * 20 + 8}
              cy="12"
              r={width < 568 ? 5.5 : 7}
              fill={colorRange[medal.type]}
              stroke="#333"
              stroke-width="1"
              opacity={(hovered && hovered !== medal) ? 0.3 : (selectedDeporte === "Todos" || selectedDeporte === medal.deporte ? 1 : 0.3)}
              on:mouseover={(event) => {
                hovered = medal;
                const rect = event.target.getBoundingClientRect();
                tooltipX = rect.left + window.scrollX;
                tooltipY = rect.top + window.scrollY;
              }}
              on:mouseout={() => {
                hovered = null;
              }}
              />
          {/each}
          {#if index < dataAgrupada.length - 1}
            <line
              x1="0"
              y1="25"
              x2={width - margin.left - margin.right}
              y2="25"
              stroke="#ddd"
              stroke-width="1"
            />
          {/if}
        </g>
      {/each}
    </g>
  </svg>
  {#if hovered}
  <Tooltip data={hovered} {tooltipX} {tooltipY} {width} {height}/>
  {/if}
</div>

<style>
    h1 {
    font-size: 20px;
    margin-bottom: 3px;
    font-weight: 600;
  }
  div {
    box-sizing: border-box;
  }
  .year {
    font-size: 0.95rem;
    font-style: italic;
    font-weight: bold;
    margin-bottom: 2px;
  }
  .medal-count {
  font-size: 0.8rem; /* Ajusta el tamaño según sea necesario */
  font-weight: 400;
  fill: #333; /* Opcional: define el color del texto */
}
  select {
    padding: 2px;
    font-size: 0.9rem;
    border: 1px solid #ccc;
  }
  .filter-container {
    display: flex;
    align-items: center;
  }

  .filter-container label {
    margin-right: 10px;
  }
  .ediciones {
    display: flex;
    align-items: baseline;
  }
</style>