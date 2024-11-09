---
theme: dashboard
title: Airbnb Dashboard
toc: false
---

# ${selected} Airbnb Dashboard

<!-- Load and transform the data -->

```js
const options = ["Lisbon", "Paris"];
const selected = view(Inputs.select(options, {label: "Selecciona una ciudad"}));
```

```js
const lisbon = FileAttachment("data/lisbon.csv").csv({typed: true, header: true});
const paris = FileAttachment("data/paris.csv").csv({typed: true, header: true});

const data = selected === "Lisbon" ? lisbon : paris;
```



<!-- Cards with big numbers -->

<div class="grid grid-cols-3">
  <div class="card">
    <h2>Anuncios 🏠</h2>
    <span class="big">${data.reduce((acc, d) => acc + d.count, 0).toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Barrios 📌</h2>
    <span class="big">${d3.group(data, (d) => d.neighbourhood).size}</span>
  </div>
  <div class="card">
    <h2>Precio Promedio 💰</h2>
    <span class="big">${d3.mean(data, (d) => d.mean).toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Barrio más caro <span class="muted">/ ${data.find((d) => d.mean === d3.max(data, (d) => d.mean)).neighbourhood}</span></h2>
    <span class="big">${d3.max(data, (d) => d.mean).toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
  </div>
    <div class="card">
        <h2>Barrio más barato <span class="muted">/ ${data.find((d) => d.mean === d3.min(data, (d) => d.mean)).neighbourhood}</span></h2>
        <span class="big">${d3.min(data, (d) => d.mean).toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
    </div>  
</div>

<!-- Plot of price per neighbourhood bar chart -->

```js
function pricePerNeighbourhood(data, {width} = {}) {
    const sortedData = [...data].sort((a, b) => b.mean - a.mean);

    return Plot.plot({
        title: "Precio promedio por barrio",
        width,
        height: 2000,
        y: {
            label: "",
            grid: true,
            domain: sortedData.map(d => d.neighbourhood),
            axis: null
        },
        x: {
            label: "Precio promedio"
        },
        color: { legend: true, type: "linear", scheme: "inferno" },
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
                fill: "white", // Cambia el color si es necesario para mejorar la visibilidad
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
        ${resize((width) => pricePerNeighbourhood(data, {width}))}
    </div>
</div>