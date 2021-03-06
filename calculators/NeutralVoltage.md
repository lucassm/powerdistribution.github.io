
# Neutral-to-earth voltages from line-to-ground faults

This app models the neutral-to-earth voltage (NEV) from a fault on an overhead
distribution line. The circuit modeled is illustrated below.

<img src="NeutralVoltage.svg" style="width:100%" class="img-responsive"/>

<br/>

<!-- Script loader -->

```yaml
         #:  script=scriptloader
- lib/numeric-1.2.6.min.js
- lib/math.min.js
```

<!-- Conductor data  -->

```yaml 
         #: name=d
rac: [3.551, 2.232, 1.402, 1.114, 0.882, 0.7, 0.556, 0.441, 0.373, 0.35, 0.311, 0.278, 0.267, 0.235, 0.208, 0.197, 0.188, 0.169, 0.135, 0.133, 0.127, 0.12, 0.109, 0.106, 0.101, 0.0963]
gmr: [0.0055611962035177, 0.00700459393067038, 0.00882262274842038, 0.00990159326021141, 0.0111125174323268, 0.0124715326552536, 0.0139967498560307, 0.0157084948536593, 0.0171990576740366, 0.0177754680514267, 0.0197856043349646, 0.0209605660328388, 0.0214852445181602, 0.0227611387971986, 0.0243123406199979, 0.0249209197027924, 0.0255447325512619, 0.0270616982108416, 0.0308759703782212, 0.0311314761296609, 0.0319107497292355, 0.0327095298674806, 0.0343675751093677, 0.0349387277474913, 0.0361096666226405, 0.0367097709735484]
conductors: [6 AAC, 4 AAC, 2 AAC, 1 AAC, 1/0 AAC, 2/0 AAC, 3/0 AAC, 4/0 AAC, 250 AAC, 266.8 AAC, 300 AAC, 336.4 AAC, 350 AAC, 397.5 AAC, 450 AAC, 477 AAC, 500 AAC, 556.5 AAC, 700 AAC, 715.5 AAC, 750 AAC, 795 AAC, 874.5 AAC, 900 AAC, 954 AAC, 1000 AAC]
```

<!-- Input form -->

```yaml
         #:  jquery=dform
class : form
html:
  - type: div
    class: row
    html:
      - type: div
        class: col-md-4
        html:
          - name: phase
            type: select
            bs3caption : Phase
            selectvalue: 350 AAC
            choices: [6 AAC, 4 AAC, 2 AAC, 1 AAC, 1/0 AAC, 2/0 AAC, 3/0 AAC, 4/0 AAC, 250 AAC, 266.8 AAC, 300 AAC, 336.4 AAC, 350 AAC, 397.5 AAC, 450 AAC, 477 AAC, 500 AAC, 556.5 AAC, 700 AAC, 715.5 AAC, 750 AAC, 795 AAC, 874.5 AAC, 900 AAC, 954 AAC, 1000 AAC]
      - type: div
        class: col-md-4
        html:
          - name: neutral
            type: select
            bs3caption : Neutral
            selectvalue: 2/0 AAC
            choices: [6 AAC, 4 AAC, 2 AAC, 1 AAC, 1/0 AAC, 2/0 AAC, 3/0 AAC, 4/0 AAC, 250 AAC, 266.8 AAC, 300 AAC, 336.4 AAC, 350 AAC, 397.5 AAC, 450 AAC, 477 AAC, 500 AAC, 556.5 AAC, 700 AAC, 715.5 AAC, 750 AAC, 795 AAC, 874.5 AAC, 900 AAC, 954 AAC, 1000 AAC]
      - type: div
        class: col-md-4
        html:
          - name: conductorSeparation
            type: number
            step: 1
            min: 1.0
            bs3caption : Phase-to-neutral separation, ft
            value: 5.0
  - type: div
    class: row
    html:
      - type: div
        class: col-md-4
        html:
          - name: systemVoltage
            type: number
            step: 1.0
            min: 1.0
            bs3caption : System voltage (L-L), kV
            value: 12.5
      - type: div
        class: col-md-4
        html:
          - name: totalLengthMi
            type: number
            step: 1
            min: 0.5
            bs3caption : Line length, miles
            value: 6.0
      - type: div
        class: col-md-4
        html:
          - name: faultLoc
            type: number
            step: 10
            min: 10
            max: 100
            bs3caption : Fault location, % line length
            value: 100
  - type: div
    class: row
    html:
      - type: div
        class: col-md-4
        html:
          - name: faultI
            type: number
            step: 1
            min: 1.0
            bs3caption : Available fault current at sub, kA
            value: 10.0
      - type: div
        class: col-md-4
        html:
          - name: Zsub
            type: number
            step: 0.1
            min: 0.0
            bs3caption : Substation ground resistance, ohms
            value: 1.2
  - type: div
    class: row
    html:
      - type: div
        class: col-md-4
        html:
          - name: groundsPerKFeet
            type: number
            step: 1
            min: 0.0
            bs3caption : Grounds per 1000 feet
            value: 5.0
      - type: div
        class: col-md-4
        html:
          - name: averageGround
            type: text
            step: 5
            min: 0.0
            bs3caption : Average ground resistance, ohms
            value: "default"
      - type: div
        class: col-md-4
        html:
          - name: rho
            type: number
            step: 50
            min: 0.0
            bs3caption : Earth resistivity, ohm-m
            value: 100.0
```


<h2>Results</h2>

<!-- Main calculations -->

```js

sq = function(x) {
  return x * x;
}

nidx = _.map(d.conductors, String).indexOf(neutral)
pidx = _.map(d.conductors, String).indexOf(phase)

Yaddline = function(Y, Zseries, from, to) {
    var n = Zseries.x.length - 1;
    var Yseries = Zseries.inv();
    Y.setBlock([from,from], [from+n,from+n], Y.getBlock([from,from], [from+n,from+n]).add(Yseries));   // diagonal
    Y.setBlock([to,to],     [to+n,to+n],     Y.getBlock([to,to],     [to+n,to+n]).add(Yseries));
    Y.setBlock([from,to],   [from+n,to+n], Y.getBlock([from,to], [from+n,to+n]).sub(Yseries));   // off diagonal
    Y.setBlock([to,from],   [to+n,from+n], Y.getBlock([to,from], [to+n,from+n]).sub(Yseries));
    return Y;
}

Yaddshunt = function(Y, Zshunt, busnum) {
    var n = Zshunt.x.length;
    var Yshunt = Zshunt.inv();
    Y.setBlock([busnum,busnum], [busnum+n,busnum+n], Y.getBlock([busnum,busnum], [busnum+n,busnum+n]).add(Yshunt));   // diagonal
    return Y;
}

Yaddshort = function(Y, i, j) {
    var Ybig = 1e5
    Y.x[i][i] = Y.x[i][i] + Ybig
    Y.x[j][j] = Y.x[j][j] + Ybig
    Y.x[i][j] = Y.x[i][j] - Ybig
    Y.x[j][i] = Y.x[j][i] - Ybig
    return Y;
}

YaddlineRX = function(Y, R, X, i, j) {
    var k = sq(R) + sq(X)
    Y.x[i][i] = Y.x[i][i] + R/k
    Y.x[j][j] = Y.x[j][j] + R/k
    Y.x[i][j] = Y.x[i][j] - R/k
    Y.x[j][i] = Y.x[j][i] - R/k
    Y.y[i][i] = Y.y[i][i] - X/k
    Y.y[j][j] = Y.y[j][j] - X/k
    Y.y[i][j] = Y.y[i][j] + X/k
    Y.y[j][i] = Y.y[j][i] + X/k
    return Y;
}

numberOfSections = 50
subBus = 1
totalLength = totalLengthMi * 5.28  // ohms/kfeet
sectionLength = totalLength/numberOfSections  // kfeet
if (averageGround == "well") {
    rodlen = 200/3.28
    averageGround = rho/2/Math.PI/rodlen*(Math.log(4*rodlen/(8.6*0.0254) - 1))
} else if (isNaN(Number(averageGround))) {
    rodlen = 8/3.28
    averageGround = rho/2/Math.PI/rodlen*(Math.log(4*rodlen/(5/8*0.0254) - 1))
} else {
    averageGround = Number(averageGround)
}
Zgrnd = averageGround/(groundsPerKFeet * totalLength) * numberOfSections  // ohms (average ground per section)
d_ab = conductorSeparation  // feet (distance between the phase and neutral)

r_p = d.rac[pidx]/5.28  // ohms per 1000 feet
gmr_p = d.gmr[pidx]     // feet
r_n = d.rac[nidx]/5.28  // ohms per 1000 feet
gmr_n = d.gmr[nidx]     // feet

Z = numeric.t(numeric.identity(2), numeric.identity(2))
Z.x[0][0] = r_p + 0.01807
Z.y[0][0] = 0.0529 * math.log10(278.9 * math.sqrt(rho) / gmr_p)
Z.x[1][1] = r_n + 0.01807
Z.y[1][1] = 0.0529 * math.log10(278.9 * math.sqrt(rho) / gmr_n)
Z.x[0][1] = Z.x[1][0] = 0.01807
Z.y[0][1] = Z.y[1][0] = 0.0529 * math.log10(278.9 * math.sqrt(rho) / d_ab)
Zcond = Z.mul(sectionLength)   // actual ohms
numberOfConductors = 2

// preallocate the Ybus
n = (numberOfSections + 1) * numberOfConductors
Y = numeric.t(numeric.rep([n,n], 0.0), numeric.rep([n,n], 0.0))

// make the Ybus from the impedances
for (var i = 0; i < n - numberOfConductors - 1; i = i + numberOfConductors) {
    Y = Yaddline(Y, Zcond, i, i+numberOfConductors);
}

// add the shunt grounds
for (var i = 2*numberOfConductors - 1; i < n - 1; i = i + numberOfConductors) {
    Y.set([i, i], Y.get([i, i]).add(numeric.t(1/Zgrnd, 0.0)))
}

// add the sub ground
var i = 1
Y.set([i, i], Y.get([i, i]).add(numeric.t(1/Zsub, 0.0)))

// bond the phase wire to the neutral at fault location
faultNode = Math.floor(faultLoc * numberOfSections / 100) + 1
nfault = 2 * faultNode
Y = Yaddshort(Y, nfault - 2, nfault - 1)


// Add the source impedance
Y = YaddlineRX(Y, 0.0546 * systemVoltage/Math.sqrt(3) / faultI,  0.9985 * systemVoltage/Math.sqrt(3) / faultI, 0, 1)

// make current injections (Isrc)
Isrc = numeric.t(numeric.rep([n], 0.0), numeric.rep([n], 0.0))
Isrc.y[0] = -1000*faultI
Isrc.y[1] =  1000*faultI

// Find the voltages:
V = Y.inv().dot(Isrc)

// Find the voltage drops and conductor currents
Vdrops = numeric.t(numeric.rep([2,numberOfSections], 0.0), numeric.rep([2,numberOfSections], 0.0))
for (var i = 0; i < numberOfSections; i++) {
    Vdrops.set([0,i], V.get([i * 2    ]).sub(V.get([i * 2 + 2])))
    Vdrops.set([1,i], V.get([i * 2 + 1]).sub(V.get([i * 2 + 3])))
}
Ycond = Zcond.inv()
I = Ycond.dot(Vdrops)

vn = V.abs().x.filter(function(num,idx){ if( idx % 2 ) return num;})

println("NEV at the fault = " + math.format(vn[faultNode - 1]) + " V" + "  (" + math.format(vn[faultNode - 1] / systemVoltage / 1000 * Math.sqrt(3)) + " pu)")
println("Substation NEV = " + math.format(V.abs().x[1]) + " V")

```

<br/>
<div class = "row">
<div class = "col-md-6">
    Neutral-to-earth voltages, V
    <div id="graph1" style='width:100%; height:25em;'></div>
    <div class="text-center">Distance from the substation, miles</div>
</div>
<div class = "col-md-6">
    Currents, A
    <div id="graph2" style='width:100%; height:25em;'></div>
    <div class="text-center">Distance from the substation, miles</div>
</div>
</div>

<!-- Graphs -->

```js
    x = _.range(0, (numberOfSections+1) * sectionLength / 5.28, sectionLength / 5.28)
    seriesvn = _.zip(x,vn)
    $.plot($('#graph1'), [seriesvn]);
    
    seriesI = [{label: "Phase",
                data: _.zip(x,I.abs().x[0])},
               {label: "Neutral",
                data: _.zip(x,I.abs().x[1])},
               {label: "Ground",
                data: _.zip(x,I.getRow(0).add(I.getRow(1)).abs().x)}
               ]
    $.plot($('#graph2'), seriesI);
```


## Notes

The ground rods and resistances on the line should also include customer grounds. 
The customer grounds normally have a larger impact than the utility grounds.

If the "Average ground resistance" is "default", each ground is calculated
assuming an 8-ft ground rod and using the earth resistivity given. 
For other values, enter a custom numeric value.

For impedance models, this app uses a simple
implementation of the equations outlined in section 2.4. The frequency is fixed
at 60 Hz. For more sophisticated line modeling and voltage drop calculations,
see [OpenDSS](http://smartgrid.epri.com/SimulationTool.aspx) or a transient
program like [EMTP-RV](http://emtp.com) or [ATP](http://emtp.org).
