---
title: Airbnb Dashboard
toc: false
---

# ${selected} Airbnb Dashboard

<!-- Load and transform the data -->

```js
const options = ["Paris", "Lisbon"];
const selected = view(Inputs.select(options, {label: "Selecciona una ciudad"}));
```

```js
const lisbon = FileAttachment("data/lisbon.csv").csv({typed: true, header: true});
const paris = FileAttachment("data/paris.csv").csv({typed: true, header: true});
const lisbonNeighbourhoods = FileAttachment("data/lisbon_neighbourhoods.json").json();
const parisNeighbourhoods = FileAttachment("data/paris_neighbourhoods.json").json();
```

```js
// This is a FeatureCollection with an array of features each one representing a neighbourhood as a MultiPolygon
const selectedNeighbourhoods = selected === "Lisbon" ? lisbonNeighbourhoods : parisNeighbourhoods;
const data = selected === "Lisbon" ? lisbon : paris;
```

```js
const minPropertiesCount = d3.min(data, (d) => d.count);
const maxPropertiesCount = d3.max(data, (d) => d.count);
const minCount = view(Inputs.range([minPropertiesCount, maxPropertiesCount], {label: "Número mínimo de propiedades", value: minPropertiesCount}));
const maxCount = view(Inputs.range([minPropertiesCount, maxPropertiesCount], {label: "Número máximo de propiedades", value: maxPropertiesCount}));
```

```js
const filteredData = data.filter((d) => d.count >= minCount && d.count <= maxCount);
```


<!-- Cards with big numbers -->

<div class="grid grid-cols-3">
  <div class="card">
    <h2>Anuncios 🏠</h2>
    <span class="big">${filteredData.reduce((acc, d) => acc + d.count, 0).toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Barrios 📌</h2>
    <span class="big">${d3.group(filteredData, (d) => d.neighbourhood).size}</span>
  </div>
  <div class="card">
    <h2>Precio Promedio 💰</h2>
    <span class="big">${d3.mean(filteredData, (d) => d.mean).toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Barrio más caro <span class="muted">/ ${filteredData.find((d) => d.mean === d3.max(filteredData, (d) => d.mean)).neighbourhood}</span></h2>
    <span class="big">${d3.max(filteredData, (d) => d.mean).toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
  </div>
    <div class="card">
        <h2>Barrio más barato <span class="muted">/ ${filteredData.find((d) => d.mean === d3.min(filteredData, (d) => d.mean)).neighbourhood}</span></h2>
        <span class="big">${d3.min(filteredData, (d) => d.mean).toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
    </div>  
</div>

```js
function plotMap(neighbourhoods, data) {
    const height = 610;

    const projection = d3.geoMercator();
    projection.fitExtent([[0, 0], [width, height]], neighbourhoods);

    const dataMap = new Map(data.map(d => [d.neighbourhood, {
        mean: d.mean,
        count: d.count
    }]));

    return Plot.plot({
        title: "Mapa de precios promedio por barrio",
        projection,
        width,
        height,
        color: {
            legend: true,
            scheme: "blues",
            domain: [Math.min(...data.map(d => d.mean)) - 50, Math.max(...data.map(d => d.mean))]
        },
        marks: [
            Plot.graticule(),
            Plot.geo(neighbourhoods, {
                stroke: "white",
                strokeWidth: 1,
                fill: d => {
                    const mean = dataMap.get(d.properties.neighbourhood)?.mean;
                    return mean ? mean : "black";
                },
                title: d => {
                    const mean = dataMap.get(d.properties.neighbourhood)?.mean || 0;
                    const count = dataMap.get(d.properties.neighbourhood)?.count || 0;
                    return `Barrio: ${d.properties.neighbourhood}\nPrecio promedio: ${mean}\nNúmero de propiedades: ${count}`;
                },
                tip: true
            }),
        ]
    });
}
```

<div class="grid grid-cols-1">
    <div class="card">
        ${plotMap(selectedNeighbourhoods, filteredData)}
    </div>
</div>

```js
function scatterPlotCountMean(data, {width} = {}) {
    // Ordena los datos por el eje x (count) de menor a mayor
    const sortedData = [...data].sort((a, b) => a.count - b.count);

    // Define la altura del gráfico (opcional)
    const height = 400;

    return Plot.plot({
        title: "Número de propiedades vs. Precios promedio",
        width,
        height,
        x: {
            label: "Número de propiedades",
            grid: true
        },
        y: {
            label: "Promedio",
            grid: true
        },
        marks: [
            Plot.dot(sortedData, {
                x: "count",
                y: "mean",
                title: d => `Barrio: ${d.neighbourhood}\nNúmero de propiedades: ${d.count}\nPromedio: ${d.mean}`,
                stroke: "black", // Contorno negro para destacar los puntos
                fill: "steelblue", // Color de los puntos
                tip: true // Habilita el tip para cada punto
            })
        ],
    });
}
```
```js
function histogramCount(data, {width} = {}) {
    const height = 400;

    return Plot.plot({
        title: "Distribución del número de propiedades",
        width,
        height,
        x: {
            label: "Número de propiedades",
            grid: true
        },
        y: {
            label: "Cantidad de barrios",
            grid: true
        },
        marks: [
            Plot.rectY(data, Plot.binX({y: "count", thresholds: 8}, {
                x: "count",
                fill: "steelblue",
                title: d => `Número de propiedades: ${d.bin0} - ${d.bin1}\nCantidad de barrios: ${d.count}`,
                tip: true
            }))
        ],
    });
}
```

<div class="grid grid-cols-2">
    <div class="card">
        ${resize((width) => histogramCount(filteredData, {width}))}
    </div>
    <div class="card">
        ${resize((width) => scatterPlotCountMean(filteredData, {width}))}
    </div>
</div>

<!-- Plot of price per neighbourhood bar chart -->

```js
function pricePerNeighbourhood(data, {width} = {}) {
    const sortedData = [...data].sort((a, b) => b.mean - a.mean);
    
    const height = sortedData.length * 20;

    return Plot.plot({
        title: "Precio promedio por barrio",
        width,
        height,
        y: {
            label: "",
            grid: true,
            domain: sortedData.map(d => d.neighbourhood),
            axis: null
        },
        x: {
            label: "Precio promedio"
        },
        color: { legend: true, type: "linear", scheme: "blues" },
        marks: [
            Plot.barX(sortedData, {
                y: "neighbourhood",
                x: "mean",
                fill: "count",
                title: d => `Precio: ${d.mean}`,
                tip: true
            }),
            Plot.text(sortedData, {
                y: "neighbourhood",
                x: "mean",
                text: d => d.neighbourhood,
                fill: "black", // Cambia el color si es necesario para mejorar la visibilidad
                dx: -10, // Ajusta la posición horizontal del texto
                dy: 0, // Ajusta la posición vertical del texto
                textAnchor: "end" // Alinea el texto a la derecha
            }),
            Plot.ruleY([0])
        ],
    });
}
```

<div class="grid grid-cols-1">
    <div class="card">
        ${resize((width) => pricePerNeighbourhood(filteredData, {width}))}
    </div>
</div>