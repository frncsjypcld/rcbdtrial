<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Randomized Complete Block Design (RCBD)</title>

<style>
    * {
        box-sizing: border-box;
    }

    body {
        font-family: Arial, sans-serif;
        max-width: 1100px;
        margin: 30px auto;
        padding: 20px;
        color: #222;
        background: #f5f7fa;
    }

    header {
        text-align: center;
        margin-bottom: 25px;
    }

    h1 {
        font-size: 26px;
        margin-bottom: 10px;
    }

    .author {
        font-size: 14px;
        color: #555;
    }

    section {
        background: white;
        padding: 20px;
        margin-bottom: 20px;
        border-radius: 8px;
        box-shadow: 0 2px 8px #00000010;
    }

    h2 {
        font-size: 19px;
        margin-top: 0;
    }

    .controls {
        display: flex;
        gap: 15px;
        flex-wrap: wrap;
        align-items: end;
    }

    label {
        display: block;
        margin-bottom: 6px;
        font-weight: bold;
    }

    input {
        width: 100%;
        min-width: 75px;
        padding: 9px;
        border: 1px solid #bbb;
        border-radius: 4px;
    }

    button {
        padding: 10px 16px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
        font-weight: bold;
    }

    .primary {
        background: #174ea6;
        color: white;
    }

    .secondary {
        background: #e5e7eb;
        color: #222;
    }

    .table-wrap {
        overflow-x: auto;
        margin-top: 15px;
    }

    table {
        border-collapse: collapse;
        width: 100%;
        min-width: 450px;
    }

    th, td {
        border: 1px solid #ccc;
        padding: 9px;
        text-align: center;
    }

    th {
        background: #eaf0f8;
    }

    td input {
        min-width: 65px;
        text-align: center;
    }

    .note {
        color: #555;
        font-size: 13px;
        line-height: 1.5;
    }

    .error {
        color: #b42318;
        font-weight: bold;
    }

    .formula {
        font-family: Georgia, serif;
        background: #f5f7fa;
        padding: 12px;
        margin: 8px 0;
        overflow-x: auto;
    }

    footer {
        text-align: center;
        color: #666;
        font-size: 12px;
        margin-top: 25px;
    }

    @media print {
        body {
            background: white;
            margin: 0;
        }
        section {
            box-shadow: none;
            border: 1px solid #ddd;
        }
        button, .controls {
            display: none;
        }
    }
</style>
</head>

<body>

<header>
    <h1>Randomized Complete Block Design (RCBD)</h1>
    <p class="author">
        By: Frances Jay B. Pacaldo, LPT, MAED-Math
    </p>
</header>

<section>
    <h2>1. Experimental Design</h2>

    <div class="controls">
        <div>
            <label for="treatments">Number of Treatments (t)</label>
            <input type="number" id="treatments"
                   value="4" min="2" max="20">
        </div>

        <div>
            <label for="blocks">Number of Blocks (b)</label>
            <input type="number" id="blocks"
                   value="5" min="2" max="30">
        </div>

        <button class="primary" onclick="createTable()">
            Generate Data Table
        </button>
    </div>

    <p class="note">
        Enter one observation for every treatment in each block.
        Each block must contain all treatments exactly once.
    </p>
</section>

<section>
    <h2>2. Enter Experimental Data</h2>

    <div id="dataTable" class="table-wrap"></div>

    <p class="note">
        Example: rows represent blocks and columns represent treatments.
        You may edit the sample values before computing.
    </p>

    <button class="primary" onclick="calculateRCBD()">
        Calculate RCBD ANOVA
    </button>

    <button class="secondary" onclick="clearData()">
        Clear Data
    </button>

    <button class="secondary" onclick="window.print()">
        Print Results
    </button>

    <p id="message" class="error"></p>
</section>

<section id="results" style="display:none;">
    <h2>3. Descriptive Summary</h2>
    <div id="summary" class="table-wrap"></div>

    <h2>4. Analysis of Variance (ANOVA)</h2>
    <div id="anova" class="table-wrap"></div>

    <p class="note">
        Decision rule: At the chosen significance level, reject the
        null hypothesis when the calculated F-value exceeds the
        corresponding critical F-value. The p-value provides an
        equivalent significance test.
    </p>
</section>

<section>
    <h2>5. Formulas Used</h2>

    <div class="formula">
        Correction Factor (CF) = G² / N
    </div>

    <div class="formula">
        Total SS = Σx² − CF
    </div>

    <div class="formula">
        Treatment SS = Σ(T² / b) − CF
    </div>

    <div class="formula">
        Block SS = Σ(B² / t) − CF
    </div>

    <div class="formula">
        Error SS = Total SS − Treatment SS − Block SS
    </div>

    <div class="formula">
        MS = SS / df
    </div>

    <div class="formula">
        F-treatment = MS-treatment / MS-error
    </div>

    <div class="formula">
        F-block = MS-block / MS-error
    </div>

    <p class="note">
        G = grand total; N = total number of observations;
        T = treatment total; B = block total.
    </p>
</section>

<footer>
    Randomized Complete Block Design | Statistical Analysis
</footer>

<script>
function getDimensions() {
    const t = Number(document.getElementById("treatments").value);
    const b = Number(document.getElementById("blocks").value);

    if (!Number.isInteger(t) || !Number.isInteger(b) ||
        t < 2 || t > 20 || b < 2 || b > 30) {
        throw new Error(
            "Treatments must be 2–20 and blocks must be 2–30."
        );
    }

    return { t, b };
}

function createTable() {
    try {
        const { t, b } = getDimensions();
        let html = "<table><thead><tr><th>Block</th>";

        for (let j = 0; j < t; j++) {
            html += `<th>Treatment ${j + 1}</th>`;
        }

        html += "</tr></thead><tbody>";

        for (let i = 0; i < b; i++) {
            html += `<tr><th>Block ${i + 1}</th>`;

            for (let j = 0; j < t; j++) {
                const value = 10 + i * 2 + j;
                html += `
                    <td>
                        <input type="number" step="any"
                            id="x_${i}_${j}"
                            value="${value}"
                            aria-label="Block ${i + 1},
                            Treatment ${j + 1}">
                    </td>`;
            }

            html += "</tr>";
        }

        html += "</tbody></table>";

        document.getElementById("dataTable").innerHTML = html;
        document.getElementById("message").textContent = "";
        document.getElementById("results").style.display = "none";
    } catch (error) {
        document.getElementById("message").textContent = error.message;
    }
}

function clearData() {
    try {
        const { t, b } = getDimensions();
        for (let i = 0; i < b; i++) {
            for (let j = 0; j < t; j++) {
                document.getElementById(`x_${i}_${j}`).value = "";
            }
        }
        document.getElementById("results").style.display = "none";
        document.getElementById("message").textContent = "";
    } catch (error) {
        document.getElementById("message").textContent = error.message;
    }
}

// Regularized incomplete beta function for F-distribution probabilities.
function betaFraction(a, b, x) {
    const maxIterations = 200;
    const epsilon = 3e-14;
    const tiny = 1e-300;

    let qab = a + b;
    let qap = a + 1;
    let qam = a - 1;
    let c = 1;
    let d = 1 - qab * x / qap;

    if (Math.abs(d) < tiny) d = tiny;
    d = 1 / d;
    let h = d;

    for (let m = 1; m <= maxIterations; m++) {
        let m2 = 2 * m;
        let aa = m * (b - m) * x /
            ((qam + m2) * (a + m2));

        d = 1 + aa * d;
        if (Math.abs(d) < tiny) d = tiny;
        c = 1 + aa / c;
        if (Math.abs(c) < tiny) c = tiny;
        d = 1 / d;
        h *= d * c;

        aa = -(a + m) * (qab + m) * x /
            ((a + m2) * (qap + m2));

        d = 1 + aa * d;
        if (Math.abs(d) < tiny) d = tiny;
        c = 1 + aa / c;
        if (Math.abs(c) < tiny) c = tiny;
        d = 1 / d;
        const delta = d * c;
        h *= delta;

        if (Math.abs(delta - 1) < epsilon) break;
    }

    return h;
}

function logGamma(z) {
    const p = [
        676.5203681218851, -1259.1392167224028,
        771.32342877765313, -176.61502916214059,
        12.507343278686905, -0.13857109526572012,
        9.9843695780195716e-6, 1.5056327351493116e-7
    ];

    if (z < 0.5) {
        return Math.log(Math.PI) -
            Math.log(Math.sin(Math.PI * z)) -
            logGamma(1 - z);
    }

    z -= 1;
    let x = 0.99999999999980993;

    for (let i = 0; i < p.length; i++) {
        x += p[i] / (z + i + 1);
    }

    const t = z + p.length - 0.5;

    return 0.9189385332046727 +
        (z + 0.5) * Math.log(t) - t + Math.log(x);
}

function regularizedBeta(x, a, b) {
    if (x <= 0) return 0;
    if (x >= 1) return 1;

    const bt = Math.exp(
        logGamma(a + b) - logGamma(a) - logGamma(b) +
        a * Math.log(x) + b * Math.log1p(-x)
    );

    if (x < (a + 1) / (a + b + 2)) {
        return bt * betaFraction(a, b, x) / a;
    }

    return 1 - bt * betaFraction(b, a, 1 - x) / b;
}

function fSurvival(f, df1, df2) {
    if (!Number.isFinite(f) || f < 0) return NaN;
    if (f === 0) return 1;

    const x = df2 / (df2 + df1 * f);
    return regularizedBeta(x, df2 / 2, df1 / 2);
}

function format(value) {
    if (Number.isNaN(value)) return "—";
    if (!Number.isFinite(value)) return "∞";
    return value.toFixed(2);
}

function calculateRCBD() {
    try {
        const { t, b } = getDimensions();
        const data = [];

        for (let i = 0; i < b; i++) {
            const row = [];

            for (let j = 0; j < t; j++) {
                const input = document.getElementById(`x_${i}_${j}`);

                if (!input || input.value.trim() === "" ||
                    !Number.isFinite(Number(input.value))) {
                    throw new Error(
                        `Please enter a valid number for Block ${i + 1}, ` +
                        `Treatment ${j + 1}.`
                    );
                }

                row.push(Number(input.value));
            }

            data.push(row);
        }

        const N = t * b;
        const grandTotal = data.flat().reduce((a, x) => a + x, 0);
        const CF = grandTotal ** 2 / N;
        const sumSquares = data.flat().reduce((a, x) => a + x * x, 0);
        const totalSS = sumSquares - CF;

        const treatmentTotals = Array(t).fill(0);
        const blockTotals = Array(b).fill(0);

        for (let i = 0; i < b; i++) {
            for (let j = 0; j < t; j++) {
                treatmentTotals[j] += data[i][j];
                blockTotals[i] += data[i][j];
            }
        }

        const treatmentMeans = treatmentTotals.map(x => x / b);
        const blockMeans = blockTotals.map(x => x / t);

        const treatmentSS =
            treatmentTotals.reduce((s, x) => s + x * x / b, 0) - CF;

        const blockSS =
            blockTotals.reduce((s, x) => s + x * x / t, 0) - CF;

        // Remove tiny floating-point rounding discrepancies.
        const errorSS = totalSS - treatmentSS - blockSS;

        if (errorSS < -1e-8) {
            throw new Error("The calculated error SS is negative. Check the data.");
        }

        const cleanErrorSS = Math.max(0, errorSS);

        const dfTreatment = t - 1;
        const dfBlock = b - 1;
        const dfError = (t - 1) * (b - 1);
        const dfTotal = N - 1;

        const msTreatment = treatmentSS / dfTreatment;
        const msBlock = blockSS / dfBlock;
        const msError = cleanErrorSS / dfError;

        const fTreatment = msError > 0
            ? msTreatment / msError
            : (msTreatment > 0 ? Infinity : NaN);

        const fBlock = msError > 0
            ? msBlock / msError
            : (msBlock > 0 ? Infinity : NaN);

        const pTreatment = fSurvival(fTreatment, dfTreatment, dfError);
        const pBlock = fSurvival(fBlock, dfBlock, dfError);

        const treatmentRows = treatmentTotals.map((total, j) => `
            <tr>
                <td>Treatment ${j + 1}</td>
                <td>${format(total)}</td>
                <td>${format(treatmentMeans[j])}</td>
            </tr>
        `).join("");

        const blockRows = blockTotals.map((total, i) => `
            <tr>
                <td>Block ${i + 1}</td>
                <td>${format(total)}</td>
                <td>${format(blockMeans[i])}</td>
            </tr>
        `).join("");

        document.getElementById("summary").innerHTML = `
            <table>
                <thead>
                    <tr>
                        <th>Source</th>
                        <th>Total</th>
                        <th>Mean</th>
                    </tr>
                </thead>
                <tbody>
                    ${treatmentRows}
                    ${blockRows}
                    <tr>
                        <th>Grand Total</th>
                        <th>${format(grandTotal)}</th>
                        <th>${format(grandTotal / N)}</th>
                    </tr>
                </tbody>
            </table>
        `;

        const rows = [
            ["Treatments", treatmentSS, dfTreatment,
                msTreatment, fTreatment, pTreatment],
            ["Blocks", blockSS, dfBlock,
                msBlock, fBlock, pBlock],
            ["Error", cleanErrorSS, dfError, msError, null, null],
            ["Total", totalSS, dfTotal, null, null, null]
        ];

        document.getElementById("anova").innerHTML = `
            <table>
                <thead>
                    <tr>
                        <th>Source of Variation</th>
                        <th>SS</th>
                        <th>df</th>
                        <th>MS</th>
                        <th>F-value</th>
                        <th>p-value</th>
                    </tr>
                </thead>
                <tbody>
                    ${rows.map(r => `
                        <tr>
                            <td>${r[0]}</td>
                            <td>${format(r[1])}</td>
                            <td>${r[2]}</td>
                            <td>${r[3] === null ? "—" : format(r[3])}</td>
                            <td>${r[4] === null ? "—" : format(r[4])}</td>
                            <td>${r[5] === null ? "—" :
                                (r[5] < 0.001 ? "< .001" : format(r[5]))}</td>
                        </tr>
                    `).join("")}
                </tbody>
            </table>
        `;

        document.getElementById("results").style.display = "block";
        document.getElementById("message").textContent = "";
    } catch (error) {
        document.getElementById("message").textContent = error.message;
    }
}

createTable();
</script>

</body>
</html>


