---
title: Paris vs Lisbon
toc: false
---

# Paris vs Lisbon

```js
// JSON array with neighbourhoods, cols are neighbourhood, count, mean
// mean it's the average price of the properties in that neighbourhood
// count it's the number of properties in that neighbourhood
const lisbonData = FileAttachment("data/lisbon.csv").csv({typed: true, header: true});
const parisData = FileAttachment("data/paris.csv").csv({typed: true, header: true});
```

```js
// Salary source: https://www.averagesalarysurvey.com/es/salario/paris
const parisStats = {
    city: "Paris",
    tourists: 30000000,
    touristsHighSeason: 2500000,
    touristsLowSeason: 1500000,
    meanSalary: 69526,
    medianSalary: 42654,
    meanPropertyPrice: d3.mean(parisData, (d) => d.mean),
    totalProperties: d3.sum(parisData, (d) => d.count),
    dailyMeanSalary: 69526 / 365,
    dailyMedianSalary: 42654 / 365,
    population: 2133000,
};
// Salary source: https://www.averagesalarysurvey.com/pt/salario/lisbon
const lisbonStats = {
    city: "Lisboa",
    tourists: 27000000,
    touristsHighSeason: 700000,
    touristsLowSeason: 400000,
    meanSalary: 40726,
    medianSalary: 15428,
    meanPropertyPrice: d3.mean(lisbonData, (d) => d.mean),
    totalProperties: d3.sum(lisbonData, (d) => d.count),
    dailyMeanSalary: 40726 / 365,
    dailyMedianSalary: 15428 / 365,
    population: 548700
};

const citiesStats = [parisStats, lisbonStats];
```
<div class="grid grid-cols-2">
  <div class="card">
    <h2>Cantidad de Turistas Anuales Paris 🗼</h2>
    <span class="big">${parisStats.tourists.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Cantidad de Turistas Anuales Lisboa 🚋</h2>
    <span class="big">${lisbonStats.tourists.toLocaleString("en-US")}</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Salario Promedio Paris 💼</h2>
    <span class="big">${parisStats.meanSalary.toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
  </div>
  <div class="card">
    <h2>Salario Promedio Lisboa 💼</h2>
    <span class="big">${lisbonStats.meanSalary.toLocaleString("en-US", {style: "currency", currency: "USD"})}</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Cantidad de habitantes Paris 🏙️</h2>
    <span class="big">${parisStats.population.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Cantidad de habitantes Lisboa 🏙️</h2>
    <span class="big">${lisbonStats.population.toLocaleString("en-US")}</span>
  </div>
</div>

```js
function comparePropertyPrices(parisData, lisbonData, {width} = 800) {
    const parisDataLabeled = parisData.map(d => ({...d, city: "Paris"}));
    const lisbonDataLabeled = lisbonData.map(d => ({...d, city: "Lisboa"}));

    const combinedData = [...parisDataLabeled, ...lisbonDataLabeled];

    return Plot.plot({
        title: "Comparación de precios promedio de propiedades",
        width,
        height: 400,
        y: {
            label: "Precio promedio",
            grid: true,
            domain: [20, 230],
        },
        x: {
            label: "Ciudad",
            domain: ["Paris", "Lisboa"],
        },
        color: {scheme: "blues"},
        marks: [
            Plot.boxY(combinedData, {
                x: "city",
                y: "mean",
                title: d => `Ciudad: ${d.city}\nPrecio promedio: ${d.mean}`,
            }),
            Plot.dot(citiesStats, {
                x: "city",
                y: "dailyMeanSalary",
                fill: "red",
                r: 6,
                title: d => `Ciudad: ${d.city}\nIngreso diario medio: ${d.dailyMeanSalary.toFixed(2)}€`
            }),

            Plot.dot(citiesStats, {
                x: "city",
                y: "dailyMedianSalary",
                fill: "green",
                r: 6,
                title: d => `Ciudad: ${d.city}\nIngreso diario mediano: ${d.dailyMedianSalary.toFixed(2)}€`
            }),
            Plot.dot(citiesStats, {
                x: "city",
                y: "meanPropertyPrice",
                fill: "white",
                stroke: "black",
                r: 6,
                title: d => `Ciudad: ${d.city}\nPrecio promedio de propiedad: ${d.meanPropertyPrice.toFixed(2)}€`
            }),
        ],
    });
}
```
## Podria una persona promedio vivir en un Airbnb?

<div class="grid grid-cols-1">
    <div class="card">
        ${resize((width) => comparePropertyPrices(parisData, lisbonData, {width}))}
        <div>
            Ingreso diario promedio: <span style="color: red;">Rojo</span>
        </div>
        <div>
            Ingreso diario mediano: <span style="color: green;">Verde</span>
        </div>
        <div>
            Precio promedio de propiedad: <span style="color: white">Blanco</span>
        </div>
    </div>
</div>

<div class="grid grid-cols-1">
    <div class="warning">
        Esto quiere decir que, más del 50% de la gente no podria vivir en un Airbnb
    </div>
</div>

<div>
    <h3>Alcanzan los Airbnb para la cantidad de turistas en temporada alta?</h3>
    <p>Asumimos que las personas alquilan un Airbnb entre 3 promedio.</p>
</div>

```js
const guestsPerProperty = view(Inputs.range([1, 10], {step: 1, value: 4}));
```

```js
function compareAirbnbCapacity(parisStats, lisbonStats, {width} = 800) {
    const data = [
        {
            city: parisStats.city,
            season: "Alta",
            touristsPerProperty: parisStats.totalProperties / (parisStats.touristsHighSeason / guestsPerProperty) ,
            tourists: parisStats.touristsHighSeason
        },
        {
            city: parisStats.city,
            season: "Baja",
            touristsPerProperty: parisStats.totalProperties / (parisStats.touristsLowSeason /guestsPerProperty),
            tourists: parisStats.touristsLowSeason,
        },
        { 
            city: lisbonStats.city,
            season: "Alta",
            touristsPerProperty: parisStats.totalProperties / (lisbonStats.touristsHighSeason/ guestsPerProperty),
            tourists: lisbonStats.touristsHighSeason
        },
        {
            city: lisbonStats.city,
            season: "Baja",
            touristsPerProperty: parisStats.totalProperties / (lisbonStats.touristsLowSeason/ guestsPerProperty),
            tourists: lisbonStats.touristsLowSeason
        }
    ];

    return Plot.plot({
        title: "Turistas por propiedad en Airbnb durante temporadas alta y baja",
        width,
        height: 400,
        marginLeft: 100,
        y: {
            label: "Ciudad y Temporada",
            domain: data.map(d => `${d.city} - ${d.season}`),
        },
        x: {
            label: "Turistas por propiedad",
            grid: true
        },
        color: {
            type: "linear",
            scheme: "RdYlGn",
            domain: [0, 1],
            legend: true,
        },
        marks: [
            Plot.barX(data, {
                y: d => `${d.city} - ${d.season}`,
                x: "tourists",
                fill: "touristsPerProperty",
                title: d => `${d.city} en ${d.season}\nTuristas por propiedad: ${d.touristsPerProperty.toFixed(2)}`
            })
        ]
    });
}
```
<div class="grid grid-cols-1">
    <div class="card">
        ${resize((width) => compareAirbnbCapacity(parisStats, lisbonStats, {width}))}
    </div>
<p>Paris parece tener un gran problema de capacidad, en Lisboa es menor</p>
</div>






