---
layout: post
title:  "GiveWell's Change Our Mind contest and water quality interventions"
date:   2023-02-17
---

[Some time ago](https://statmodeling.stat.columbia.edu/2023/01/25/water-treatment-and-child-mortality-a-meta-analysis-and-cost-effectiveness-analysis/) I wrote about a new meta-analysis pre-print where we estimated that providing safe drinking water led to a 30% mean reduction in deaths in children under-5, based on data from 15 RCTs. Today I want to write about water, but from a perspective of cost-effectiveness analyses (CEA).

A few months ago [GiveWell](https://www.givewell.org/) (GW), an effective altruism non-profit, hosted a [Change Our Mind](https://blog.givewell.org/2022/12/15/change-our-mind-contest-winners/) contest. Its purpose was to critique and improve on GW's process/recommendations on how to allocate funding. This type of contest is obviously a fantastic idea (if you're distributing tens of millions of dollars to charitable causes, even a fraction of percent improvement to efficiency of your giving is worth paying good money for) and GW also provided pretty generous rewards for the top entries. There were two winners and I think both of them will be interesting to the readers of this blog:

1. [Noah Haber's "GiveWell's Uncertainty Problem"](https://www.metacausal.com/givewells-uncertainty-problem/)
2. An examination of [cost-effectiveness of water quality interventions](https://forum.effectivealtruism.org/posts/6cJM2pWH8dz9TnBRy/an-examination-of-givewell-s-water-quality-intervention-cost) by Matthew Romer and Paul Romer Present (MRPRP henceforth)

Soon I will post separately on the uncertainty analysis by Haber, but today I want to write a bit on MRPRP's analysis.

As I wrote last time, back in April 2022 GW [recommended a grant of $65 million](https://blog.givewell.org/2022/04/06/water-quality-overview/) for clean water, in a "major update" to their earlier assessment. The decision was based on a pretty comprehensive analysis by GW, which estimated cost-benefit of specific interventions aimed at improving water quality in specific countries.[^mortality] (I'm flattered to say that they also cited our meta-analysis as a motivation for updating their assessment.) MRPRP re-do the GiveWell's analysis and find effects that are 10-20% smaller in some cases. This is still highly cost effective, but (per the logic I already mentioned) even small differences in cost-effectiveness will have large real-world implications for funding, given that funding gap for provision of safe drinking water is calculted in hundreds of millions of dollars.

However, my intention is not to argue the numbers, but to ponder one interesting feature of various cost-effectiveness assessments, which is combining different sources of evidence. 

[^mortality]:There many benefits of clean water interventions that a decision maker should consider (and the GW/MRPRP analyses do): in addition to reductions in deaths there are also medical costs, developmental effects, and reductions in disease. For this post I am only concerned with how to model reductions in deaths.

When trying to estimate how clean water reduces mortality in children, we can estimate these reductions due to clean water either by [looking at direct experimental evidence (e.g. in our meta-analysis)](https://wwiecek.github.io/2023/01/18/water-meta-analysis.html) or indirectly: first you look at the estimates of reductions in disease (diarrhea episodes), then at evidence on how it links to mortality. The direct approach is the ideal (it is the ultimate outcome we care about; it is objectively measured and clearly defined, unlike diarrhea), but deaths are rare. That is why researchers studying water RCTs historically focused on reductions in diarrhea and often chose not to capture/report deaths. So we have many more studies of diarrhea.

Let's say you go the indirect evidence route. To obtain an estimate, we need to know or make assumptions on (1) the extent of self-reporting bias (e.g. "courtesy" bias), (2) how many diseases can be affected by clean water, and (3) the potentially larger effect of clean water on severe cases (leading to death) than "any" diarrhea. Each of these are obviously hard.

Once we have the two estimates (indirect and direct), then what? I describe GW process in footnotes (I personally think it's not great but also not directly relevant to this post).[^givewellind] Suffice to say that they use the indirect evidence to derive a "plausibility cap", the maximum size of the effect they are willing to admit into the CEA. MRPRP do it differently, by putting distributions on parameters in direct and indirect models and then running both in Stan to arrive at a combined, inverse-weighted estimate. For example, for point (2) above (which diseases are affected by clean water), they look at a range of scenarios and put a Gaussian distribution with a mean at the most probable scenario and the most optimistic scenario being 2 SDs away.

[^givewellind]:GW's process is, roughly, as follows: (1) Meta-analyse data from mortality studies, take a point estimate, adjust it for internal and external validity to make it specific to relevant contexts where they want to consider their program (e.g. baseline mortality, predicted take-up etc.). (2) Using indirect evidence they hypothesise what is the maximum impact on mortality ("plausibility cap"). (3) _If the benefits from direct evidence exceed the cap, they set benefits to the cap's value._ Otherwise use direct evidence.

[^combine]:For example, as far as I saw, neither model accounts for the fact that some of our evidence on mortality and diarrhea comes from the same sources. Maybe I missed something, but in any case I am abstracting from this to keep this general.

A priori such a model-averaging approach seems better than an arbitrary cap. However, depending on how you weigh direct vs indirect evidence models, you can have ~50% reduction or ~40% increase in the estimated benefits compared to GW's previous analysis. To keep this snappy, I put a more extensive numerical example in a footnote,[^cap] but for one of the programs MRPRP estimate of benefits is ~20% lower than GW's, because in their model 3/4 of the weight is put on the (lower variance) indirect evidence model and it dominates the result. 

[^cap]:To illustrate with numbers, I will use GW's analysis of Kenya Dispensers for Safe Water (a particular method of chlorination at water source), one of several programs they consider. (The impact of using MRPRP approach on other programs analysed by GiveWell is much less.) In GW's analysis, the direct evidence model gave 6.1% mortality reduction, but plausibility cap was 5.6%, so they set it to 5.6%. Under the MRPRP model, the direct evidence suggests about 8% reduction, compared to 3.5% in the indirect evidence model. The unweighted mean of the two would be 5.75%, but because of the higher uncertainty on the direct effect the final (inverse-variance weighted) estimate is a 4.6% reduction. That corresponds to putting 3/4 of weight on indirect evidence. If we applied the "plausibility cap" logic to the MRPRP estimates, rather than weighing two models, the estimated reduction in mortality for Kenya DSW program would be 8% rather than 4.6%, a whooping 40% increase on GW's original estimate. 

So on one hand I feel like the adjustments make sense. In the long term the answer is to collect more data on mortality. However, putting 75% weight on a model of indirect evidence rather than the one with a directly measured outcome strikes me as a strong assumption and the opposite of my intuition. (Similarly, why would you use Gaussians to encode various beliefs in the indirect evidence models? I had a look at changing this one assumption in Stan and got to quite different results. [My notes are here.](https://wwiecek.github.io/2023/01/10/give-well-cea.html)) 

Or, more generally, when averaging over two models that are somewhat hard to compare, how should we think about model uncertainty? I think it would be a good idea in principle to penalise both models, because there are many unknown unknowns in water interventions. But how to make this penalty "fair" across two different types of models, when they vary in complexity and assumptions?