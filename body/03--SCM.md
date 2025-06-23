<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>因果推断方法卡片</title>
    <style>
        :root {
            --spacing: 1.2rem;
            --radius: 16px;
        }

        body {
            background: #f0f4ff;
            font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace;
            padding: var(--spacing);
            min-height: 100vh;
        }

        .cards-container {
            display: flex;
            flex-direction: column;
            gap: var(--spacing);
            max-width: 800px;
            margin: 0 auto;
        }

        .method-card {
            padding: var(--spacing);
            border-radius: var(--radius);
            box-shadow: 0 6px 15px rgba(0,0,0,0.1);
            color: white;
            min-height: 180px;
            transition: transform 0.3s ease;
        }

        /* 渐变颜色方案 */
        .method-card:nth-child(1) { background: linear-gradient(135deg, #2A5CBF, #00C1DE) }
        .method-card:nth-child(2) { background: linear-gradient(135deg, #6C5CE7, #A66EFA) }
        .method-card:nth-child(3) { background: linear-gradient(135deg, #00B894, #55E6C1) }

        .method-card:hover {
            transform: translateY(-5px);
        }

        .card-title {
            font-size: 1.4rem;
            margin: 0 0 1rem 0;
            display: flex;
            align-items: center;
            text-shadow: 1px 1px 3px rgba(0,0,0,0.2);
        }

        .card-emoji {
            font-size: 2rem;
            margin-right: 0.8rem;
        }

        .code-block {
            background: rgba(0,0,0,0.15);
            padding: 1rem;
            border-radius: 8px;
            white-space: pre-wrap;
            line-height: 1.5;
            backdrop-filter: blur(3px);
            font-size: 0.9rem;
        }

        .code-comment {
            color: #FFEB3B;
            font-style: italic;
        }

        .code-keyword {
            color: #B2EBF2;
            font-weight: bold;
        }
        
        .code-function {
            color: #FF9E7D;
        }
    </style>
</head>
<body>
    <div class="cards-container">
        <!-- 卡片1: Classic Synthetic Control -->
        <div class="method-card">
            <h2 class="card-title">
                <span class="card-emoji">🧪</span>
                #1. Classic Synthetic Control
            </h2>
            <div class="code-block">
<span class="code-comment">* Load and prepare data</span>
cnuse synth_smoking.dta, clear
tsset state year

label variable cigsale "per-capita cigarette sales (in packs)"
label variable year "Year"

<span class="code-comment">* Run synthetic control with predictors and pre-treatment outcomes</span>
synth cigsale ///
    lnincome age15to24 beer retprice ///
    cigsale(1975) cigsale(1980) cigsale(1988), ///
    trunit(3) trperiod(1988) ///
    mspeperiod(1970(1)1988) resultsperiod(1970(1)2000) ///
    keep(synth_ca.dta) replace fig

<span class="code-comment">* Display weight matrix</span>
mat list e(V_matrix)

<span class="code-comment">* Save the synthetic control graph</span>
graph save Graph synth_ca.gph, replace
            </div>
        </div>

        <!-- 卡片2: Gap Analysis -->
        <div class="method-card">
            <h2 class="card-title">
                <span class="card-emoji">📉</span>
                #2. Gap Analysis
            </h2>
            <div class="code-block">
<span class="code-comment">* Plot the gap in predicted error</span>
use synth_ca.dta, clear
keep _Y_treated _Y_synthetic _time
drop if _time==.
rename _time year
rename _Y_treated  treat
rename _Y_synthetic counterfact
gen gap3 = treat - counterfact

<span class="code-comment">* Create gap plot with reference lines</span>
sort year 
twoway (line gap3 year, lp(solid) lw(thick) lcolor(black)), ///
    yline(0, lpattern(shortdash) lcolor(black)) ///
    xline(1988, lpattern(shortdash) lcolor(black)) ///
    xtitle("", si(medsmall)) ///
    xlabel(#10) ///
    ytitle("Gap in per-capita cigarette sales (in packs)", size(medsmall)) ///
    legend(off)

save synth_3.dta, replace
            </div>
        </div>

        <!-- 卡片3: Synthetic DID -->
        <div class="method-card">
            <h2 class="card-title">
                <span class="card-emoji">🔄</span>
                #3. Synthetic Difference-in-Differences
            </h2>
            <div class="code-block">
<span class="code-comment">* Prepare treatment variable</span>
gen treated = state==3 & year>=1988

<span class="code-comment">* Run SDID with placebo tests</span>
#delimit ;
sdid cigsale state year treated, 
    vce(placebo) reps(100) seed(123)
    graph g1on 
    g1_opt(xtitle("") scheme(plotplainblind)) 
    g2_opt(
        ylabel(0(25)150) 
        xlabel(1970(5)2000) 
        ytitle("Packs of Cigarettes Per Capita") 
        xtitle("") 
        scheme(plotplainblind)
    )
    graph_export(sdid_, .png);
#delimit cr
            </div>
        </div>
    </div>
</body>
</html>