Relationship between ribosomal protein position and tau association
================
Shon Koren

This is code demonstrating the relationship between human ribosomal
protein (RP) structural position and association with the microtubule
associated protein tau. RP structurual position is based on an
“Interface Index” algorithm based on the number of amino acids (AA)
interfacing with either rRNA or water (Natchiar et al., 2017, Shigeoka
et al., 2019).  

<center>

\(Interface Index = \displaystyle \frac{{\sum AA}_{rRNA}}{\sum(AA_{rRNA}+AA_{water})}\)

</center>

Internal RP proteins have more AA interfacing with rRNA than water
resulting in a higher Interface Index. A common cutoff for RP structural
position is 0.60 (Shigeoka et al., 2019).

  Plotting for the relative frequency of RP interface indices reveals
most RPs are external rather than internal, as expected.  

<img src="test_files/figure-gfm/unnamed-chunk-2-1.png" style="display: block; margin: auto;" />

Next, load the data from Hsieh et al., 2019 which used label free
quantification mass spectrometry (LFQ MS) to investigate the tau
interactome when immunoprecipitated from human postmortem brain tissue.
In this study, tissue samples from the brain of patients with
Alzheimer’s disease (AD) were compared to control, non-demented
patients (n = 4 per group). Before data loading, I filtered out
interface index data from any RPs not detected by MS, limiting data to
roughly 50 out of 80 potential RPs.

As noted by the Hsieh et al., 2019 study among many others including
from the Dr. Jose Abisambra lab, tau associates with many RPs. When we
plot the data from Hsieh 2019, we see that translational or ribosomal
proteins are highly enriched (log2 FC \> 2) in AD over control samples.

<img src="test_files/figure-gfm/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

There are two main possibilities dictating tau-RP association ignoring
the association of tau to rRNA or other RNA: 1. Tau binds to
extra-ribosomal proteins moreso than RPs present in stable ribosomes, or
2. Tau binds to RPs while present in ribosomes

If (1) is true, then tau would likely not favor any internal or external
RP. If (2) is true, tau would likely favor associating with external RPs
and prevent association with more internal RPs.

To assess whether there is a tendency for tau to bind to RPs based on
their structural position, the two sample Kolmogorov-Smirnov (two-sample
K-S) test can be used to determine whether two distributions occur
independent of one another. Since a common cutoff for internal RPs is an
Interface Index of 0.60, we can separate RPs based on that statistic.
The two-sample K-S test can then be used to test whether the tau
association of RPs in AD are independent of their structural position in
the ribosome:

``` r
ff = 'RP.interface.csv'
rp = as.data.frame(read.csv(ff, as.is=TRUE,header=TRUE,sep=',', check.names=FALSE, fileEncoding = "UTF-8-BOM"))
colnames(rp)[1] = 'Symbol'
colnames(rp)[3] = 'AD.CT.FC'

rp.filt.in = rp %>% filter(Interface > .6)
rp.filt.ex = rp %>% filter(Interface < .6)
ks.test(rp.filt.in$AD.CT.FC, rp.filt.ex$AD.CT.FC)
```

    ## 
    ##  Two-sample Kolmogorov-Smirnov test
    ## 
    ## data:  rp.filt.in$AD.CT.FC and rp.filt.ex$AD.CT.FC
    ## D = 0.38235, p-value = 0.2952
    ## alternative hypothesis: two-sided

Since the p value of the two-sample K-S test \> 0.05, ribosomal
structural position does not correlate with whether RPs associate with
tau more in AD. This is more easily viewed when plotted:

``` r
p1 = ggstatsplot::ggscatterstats(
  data = rp,
  x = Interface,
  y = AD.CT.FC,
  type = "np",
  xlab = "RP Interface Index\nHigher = Interior",
  ylab = "Tau-IP Intensity AD-CT (log2 FC)",
  label.var = Symbol,
  label.expression = pval > 1.3,
  results.subtitle = FALSE,
  marginal = FALSE,
  conf.level = 0,
  title = "Relationship between RP interface index and tau association",
  subtitle = "External RPs (Interface Index < 0.60) do not associate with tau greater than internal RPs:
  Two-sample Kolmogorov-Smirnov test: D = 0.38235, p-value = 0.2952",
  caption = "RP interface data from Natchiar et al., 2017, Shigeoka et al., 2019\nTau-IP data from Hsieh et al., 2019",
  bf.message = FALSE,
  centrality.parameter = "none"
)
p1 + geom_vline(xintercept = 0.60, color = "red", linetype = "dashed", size = 1)
```

<img src="test_files/figure-gfm/unnamed-chunk-7-1.png" style="display: block; margin: auto;" />

The end result of this analysis reveals that tau associates with RPs
regardless of their structural position. This suggests that tau might
associate with RPs prior to their incorporation into the full ribosome,
or based on some other factor.
