
<!-- This is a trick to get the input form to the left of the results -->
<style media="screen" type="text/css">
.yamlresult {
    width: 15em; 
    float: left; 
    margin-right: 3em;
    margin-top: 1em;
    padding-left: 1em;
    padding-right: 1em;
    background-color: LightSteelBlue;
    line-height: 190%;
    -moz-border-radius: 10px;
    -webkit-border-radius: 10px;
    border-radius: 10px; 
    -khtml-border-radius: 10px;
}
</style>

# Arc flash in a padmounted switch

```yaml jquery=jsonForm name=frm
schema: 
  D:
    type: number
    title: "Working distance, in"
    default: 36
  clothing:
    type: number
    title: "Clothing rating, cal/cm^2"
    default: 8.0
  I:
    type: number
    title: "Bolted current, kA"
    default: 6.0
  t:
    type: number
    title: "Duration, sec"
    default: 1.0
  k:
    type: number
    title: "Safety multiplier"
    default: 1.15
  graphextras: 
    type: string
    title: Plotting extras
    enum: 
      - Vary working distance
      - Vary clothing
      - None
form: 
  - "*"
```


```js
t = Number(t)
I = Number(I)
clothing = Number(clothing)
D = Number(D)
k = Number(k)

<!-- t^1.35 = E * d^2.1 / (k * 3547 * I^1.5) -->
<!-- k * 3547 * I^1.5 * t^1.35 = E * d^2.1 -->

pow = Math.pow

findcals = function(I, t, d, k) {
   return k * 3547 * pow(I, 1.5) * pow(t, 1.35) / pow(d, 2.1);
}
findduration = function(E, I, d, k) {
   return pow(E * pow(d, 2.1) / (k * 3547 * pow(I, 1.5)), 1/1.35);
}
console.log(1)
```

## Results

```js output=markdown
console.log(2)
cals = findcals(I, t, D, k)
console.log(3)
duration = findduration(clothing, I, D, k)
console.log(4)

println()
println("Incident energy for the given current and duration = **" + cals.toFixed(2) + " cal/cm^2**")
println()
println("Duration limit for the given current and clothing = **" + duration.toFixed(2) + " secs**")

```

```yaml name=plotinfo
chart:
    type: line
    width: 500
    height: 500
    spacingRight: 20
title:
    text: Time-current curve
                
plotOptions: 
    series:
        marker: 
            enabled: false
xAxis:
    type: 'logarithmic'
    min: 1
    max: 100
    endOnTick: true
    tickInterval: 1
    minorTickInterval: 0.1
    gridLineWidth: 1
    title:
        text: "Current, kA"

yAxis:
    type: 'logarithmic'
    min: .02
    max: 10
    tickInterval: 1
    minorTickInterval: 0.1
    title:
        text: "Time, sec"

legend: 
    align: right
    verticalAlign: middle
    layout: vertical
    borderWidth: 0
```

```js
currents = numeric.pow(10,numeric.linspace(0,2,100)) 
durations1 = _.map(currents, function(I) {return findduration(clothing, I, D, k)})
series1 = _.zip(currents,durations1)
if (graphextras == "Vary clothing") {
    durations0 = _.map(currents, function(I) {return findduration(clothing * 2, I, D, k)})
    series0 = _.zip(currents,durations0)
    durations2 = _.map(currents, function(I) {return findduration(clothing / 2, I, D, k)})
    series2 = _.zip(currents,durations2)
    plotinfo.series = [{name: clothing*2 + " cals at " + D + '"', data: series0}, 
                       {name: clothing   + " cals at " + D + '"', data: series1}, 
                       {name: clothing/2 + " cals at " + D + '"', data: series2}] 
} else if (graphextras == "Vary working distance") {
    durations0 = _.map(currents, function(I) {return findduration(clothing, I, D * 2, k)})
    series0 = _.zip(currents,durations0)
    durations2 = _.map(currents, function(I) {return findduration(clothing, I, D / 2, k)})
    series2 = _.zip(currents,durations2)
    plotinfo.series = [{name: clothing + " cals at " + D*2 + '"', data: series0}, 
                       {name: clothing + " cals at " + D + '"', data: series1}, 
                       {name: clothing + " cals at " + D/2 + '"', data: series2}] 
} else {
    plotinfo.series = [{name: clothing + " cals at " + D + '"', data: series1}]
}
//plotinfo.series = [{data: series1}]
$active_element.highcharts(plotinfo)
```

</br>

# Notes

This app models the arc flash incident energy based on tests of PMH
padmounted switches. For more information, see section 14.8 and 
EPRI 1022697 [2011], and Short and Eblen [2012].

The incident energy in this equipment is higher than predicted by
[IEEE 1584](mdpad.html?1584.md) because of horizontal busbars.
Magnetic fields from the arc current push the arcs and arc energy out
of the enclosure towards the worker. This equipment is also unusual in
that the incident energy is not linear with duration--the heat rate
increases with increasing duration.

# References

EPRI 1022697, *Distribution Arc Flash: Phase II Test Results and
Analysis*, Electric Power Research Institute, Palo Alto, CA, 2011.

Short, T. A. and Eblen, M. L., "Medium-Voltage Arc Flash in Open Air
and Padmounted Equipment," *IEEE Transactions on Industry Applications*,
vol. 48, no. 1, pp. 245-253, Jan.-Feb. 2012.

