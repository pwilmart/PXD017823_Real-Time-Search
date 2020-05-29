# PXD017823_Real-Time-Search

Re-analysis of data from PXD017823 exploring real time search for TMT.

## Re-analysis of data from [Schweppe et al. 2020](https://pubs.acs.org/doi/full/10.1021/acs.jproteome.9b00860)

> Devin K. Schweppe, Jimmy K. Eng, Qing Yu, Derek Bailey, Ramin Rad, Jose Navarrete-Perea, Edward L. Huttlin, Brian K. Erickson, Joao A. Paulo, and Steven P. Gygi, 2020. Full-Featured, Real-Time Database Searching Platform Enables Fast and Accurate Multiplexed Quantitative Proteomics. Journal of proteome research, 19(3), pp.2026−2034.

> [First RTS paper](https://pubs.acs.org/doi/10.1021/acs.jproteome.8b00899) - Erickson, B.K., Mintseris, J., Schweppe, D.K., Navarrete-Perea, J., Erickson, A.R., Nusinow, D.P., Paulo, J.A. and Gygi, S.P., 2019. Active instrument engagement combined with a real-time database search for improved performance of sample multiplexing workflows. Journal of proteome research, 18(3), pp.1299-1306.

### Phil Wilmarth, OHSU

### May 27, 2020

---

## Overview

The two papers listed above demonstrate the use of real time searching (RTS) to control data acquisition in [SPS MS3 TMT](https://pubs.acs.org/doi/abs/10.1021/ac502040v) experiments. The first paper used a simple binomial scoring model to show that searches could be completed in millisecond time scales and that now instrument control APIs could be used to react to search MS2 scans in real time.

The motivations for this are mostly due to the time the [Tribrid Orbitraps](https://pubs.acs.org/doi/abs/10.1021/ac403115c) spend on the MS3 scan for reporter ion quantification in TMT experiments. It was argued that many of the SPS MS3 linked scans did not produce usable data. If the instruments not-so-short attention span could be focused on identifiable peptides, maybe the amount of useful data per unit of instrument time could be improved.

The second paper by Schweppe et al. built upon the first paper in a few ways. The in-house binomial search engine (more of a Briggs and Stratton engine) was replaced by a souped up and stripped down version of the [Comet search engine](http://comet-ms.sourceforge.net/) (Comet on nitro?). Within reason, this allows more realistic search parameters and protein FASTA files. Monoisotopic peak refinement and a running false discovery rate method for score cutoff determination were also implemented along with RTS in a package called Orbiter. Part of the running FDR method was a linear discriminant combination of Comet score values to better separate incorrect from correct matches.

Bold claims that you can get all of the data in half the time with Orbiter RTS are also made. That sounds pretty good, right? Who doesn't love a good half off sale? Does it make sense that using RTS is twice as good? We usually skip 1+ ions so that solvent ions and polymers don't waste instrument time. The fraction of acquired MS2 scans that we can identify on Orbitraps these days is quite a bit higher than we had on low resolution ion traps. Is the situation still so bad that there are low hanging factors-of-two fruit?

When you get old and have lots of grey hair, you start thinking every door knock, phone call, and email is out to get you. That makes you just a wee bit skeptical. The data from Schweppe et al. was deposited in PRIDE (project [PXD017823](https://www.ebi.ac.uk/pride/archive/projects/PXD017823)), so I downloaded the data and re-processed it with my [pipeline](https://github.com/pwilmart/PAW_pipeline). Two sets of samples were presented in the paper: yeast samples and human cell line samples. Only the human cell line data (presented in Figure 4) was in the PRIDE archive.

The instrument was a Thermo Orbitrap Fusion Lumos Tribrid using typical SPS MS3 settings. Proteins were digested with trypsin and labeled with 10-plex TMT. High pH reverse phase was used to create a 12-fraction experiment. A conventional SPS MS3 method was used with 180-minute low pH RP separations for one dataset. The same 12-fractions were also ran with a 90-minute gradient using the Orbiter RTS acquisition software. However, the real time search use and the gradient length were not the only differences. A dynamic-exclusion like feature called protein close-out was also employed. Protein close-out was mentioned in a figure caption of the first RTS paper. It was described as limiting MS3 scans for each protein once data from 4 PSMs had been acquired. I think this is applied per LC run. It is not clear exactly what other criteria might be in play. Obviously, we have to have a successful sequence assignment so we know the protein. It is not mentioned if shared peptides are excluded (and in what context shared is define in), or if some sort of reporter ion quality metric has to be met.

Details aside, I suspect protein close-out might be pretty important to the claim of all of the data in half the time. There are no data for RTS with and without protein close-out in the archive, so its effect cannot be evaluated.  

## Any evidence for a mass measurement problem?

The PAW pipeline does not recommend using narrow parent ion tolerance database searching (a common shortcoming in nearly all proteomic data analyses). Instead wider tolerance searches are used and histograms of the deltamasses between the measured MH+ values and the theoretical MH+ masses of the assign peptide sequences are used to create conditional discriminant score histograms. This allows some quality control of the mass calibration and resolution of the instrument.

Maybe we can see something funny about the deltamasses that would suggest that we have a monoisotopic mass calling issue that needs fixing. Why fix something if it is not broken?  

![Regular_deltamass_2plus](images/Regular_deltamass_2plus.png)

![RTS_deltamass_2plus](images/RTS_deltamass_2plus.png)

The 2+ deltamasses look very similar for either acquisition mode. There does not seem to be any evidence that the regular acquisition mode data has any mass measurement issues.

![Regular_deltamass_3plus](images/Regular_deltamass_3plus.png)

![RTS_deltamass_3plus](images/RTS_deltamass_3plus.png)

The 3+ deltamasses are also similar with no indications of any issues.

![Regular_deltamass_4plus](images/Regular_deltamass_4plus.png)

![RTS_deltamass_4plus](images/RTS_deltamass_4plus.png)

The 4+ deltamasses have fewer PSMs and the there is some peak broadening with increasing charge.

There does not seem to be any indications that there is any significant mass measurement issue to be correcting. Thermo has improved their peak calling algorithms over the years. There may have been some issues many years ago that have become urban myths. It is probably not worth the computational time to test monoisotopic masses. The publication also uses a linear discriminant function for improved correct and incorrect discrimination. The deltaCN score in Comet needs a sufficient number of scored matches to be reliable. Trying to use very narrow tolerance searches of less than 20 PPM might be problematic. Wider tolerances of 50 to 100 PPM might be safer and would be plenty fast.

---

## Dataset Details

Each experiment was 12 fractions. We can tally the number of MS2 and MS3 scans logged by the instrument in each acquisition mode. These are the starting details.

#### Overall numbers of scans:

Mode|Total MS2 Scans|Total MS3 Scans|MS3 fraction
----|---------------|---------------|------------
Regular SPS MS3|358,870|358,867|100%
RTS w/ close-out|462,711|41,647|9%

The PAW pipeline uses a linear discriminant function transformation of the Comet search results similar to the functional form used in the [PeptideProphet paper](https://pubs.acs.org/doi/abs/10.1021/ac025747h). Scores are subdivided into classes based on deltamass, charge state, modification state, and (possibly) number of tryptic termini. The [target/decoy method](https://www.nature.com/articles/nmeth1019?report=reader) is used to set score thresholds to achieve a 1% PSM FDR.

#### Filtered scan counts (1% PSM FDR):

Mode|Total|Non-empty MS3|Empty/Missing MS3|Fraction Non-empty
----|-----|-------------|-----------------|------------------
Regular SPS MS3|106,989|106,835|154|99.9%
RTS w/ close-out|116,671|30,604|86,067|26.2%

The identification rates were 30% for the regular SPS MS3 data and 25% for the RTS data. In all quantitative shotgun proteomics experiments, protein inference should occur before proper quantitative protein summaries derived from the quantitative PSM data can be done. Many observed peptides can match to more than one protein. This needs to be determined before individual PSMs can be categorized as informative or non-informative for expression summaries. Low abundance peptides can be identified due to the trapping nature of the instruments. Identification and quantification are somewhat independent. The criteria for quantification is stricter than for identification. Identified peptides may have no or poor reporter ion signals even if an MS3 scan was acquired. The RTS mode adds another wrinkle. It does not run your LC backwards to undo the time spent acquiring an MS2 scan. If that scan does not yield a good enough score for an identification, the best the method can do is to skip the MS3 scan and try the next precursor. There will be orphan MS2 scans (MS2 scans without any associated MS3 scan) when doing RTS acquisition.

A non-empty tally means we had an MS2 scan and an associated MS3 scan and the reporter ions in the MS3 scan were sufficiently good signals. An empty tally means that either an MS2 scan did not have an MS3 scan or that the MS3 scan had poor reporter ion signals. We have a dramatic drop in the number of MS3 scans for the RTS experiment. That frees up instrument time to sample more precursors (we had 29% more MS2 scans).

We also sum up about 3.5 times more PSMs into protein totals for the regular SPS MS3 versus the RTS method with protein close-out. The average sum of all reporter ions per channel was 8.61E9 for the regular SPS MS3 method compared to 2.70E9 for the RTS method (3.14 times larger). I do not know what fraction of the MS2 scans that did not have MS3 scans come from an MS2 with a low score (not identifiable) versus MS2 scans for proteins that were on the protein close-out list.    

We can break it down by fractions, too. That might give us some insight into the protein close-out effect.

![Reg_filtered_scans](images/Reg_filtered_scans.png)

With the regular SPS MS3 method, we have essentially an equal number of MS3 scans as MS2 scans for each first dimension fraction. There are very few MS3 scans without reporter ion signals. A little more detective work may be needed to understand the trends in the RTS data. Data without protein close-out would be helpful.

![RTS_filtered_scans](images/RTS_filtered_scans.png)

The RTS data is quite different. The steady decline in non-empty PSM count is likely a result of the protein close-out feature. As the instrument acquires fewer MS3 scans, there is an increase in the number of MS2 scans. The MS2 scan counts were flatter for the regular SPS MS3 method.

---

## Individual Proteins (two of them)

Figure 4F shows two proteins where the acquisition mode resulted in different quantitative measurements. There are some data differences between the Schweppe et al. publication and my pipeline. The quantitative data I extract for the reporter ions are the peak heights. Peak heights are often a good proxy for peak areas (if peaks have the same shapes) and are commonly used for quantitative mass spectrometry measurements. The Gygi lab (and Proteome Discoverer) favor signal to noise ratios. These are signal quality metrics and not quantitative measures. S/N ratios do not have any units. They are compressed in range because of the division. If you have a measurement with noise and you have a noise estimate, the thing to do is to **subtract** the noise from the signal. This is actually already done on the data that is logged by Orbitrap instruments. FDR filtering differs a little between any pipelines. Protein inference and what peptides are considered usable for quantitative roll up is not well detailed in the publication, but that also differs by pipeline (probably more than for PSM FDR filtering).  

That said, what did my analysis have for the expression of these two proteins?

![Endoglin_protein](images/Endoglin_protein.png)

![ALPK2_protein](images/ALPK2_protein.png)

Note that ALPK2 (alpha-protein kinase 2) is only identified by a single PSM. My pipeline does not report single peptide protein IDs. I was able to search through the top hit summaries and find the data. The data does not have any normalizations done for ALPK2. The single peptide from each acquisition mode was not the same, by the way.

The summed protein values for Endoglin above are very similar to what is shown in Figure 4F. Endoglin had 4 and 3 peptides, respectively. We can look at the intensities for the individual peptides, too.

![Endoglin_peptides](images/Endoglin_peptides.png)

It is probably the one R.GPITSAAELNDPQSILLR.L pepide in the regular SPS MS3 data that drives the different expression pattern between the regular acquisition and the RTS acquisition.

Do these two edge cases have any real bearing on the discussion? Probably not. There can be interference in the m/z regions where reporter ions are (immonium and other low m/z ions). There can also be interference in the isolation window (co-eluting peptides and low-level background when precursor is close to noise levels). There can be biological reasons why specific peptides from specific parts of a protein may have expression levels at odds with more a bulk protein expression estimate. Generally speaking, summing up PSM reporter ions tends to stabilize the peptide and protein values (see the [dilution series notebook](https://pwilmart.github.io/TMT_analysis_examples/MAN1353_peptides_proteins.html)).

---

## What about the rest of the proteins?

There are three different cell lines (HCT116, MCF7, and HEK293) so we can compare the cells for differential expression using [edgeR](https://academic.oup.com/bioinformatics/article/26/1/139/182458) and [Jupyter](https://jupyter.org/) notebooks. The cell lines are different from each other. The variance is low for cell cultures (around 5% for median CVs). That combination means that most proteins are statistically different between cell lines (75-80% of the 7469 proteins). The increased sampling for the regular SPS MS3 method had a little lower CVs and had about 4% more differential candidates than the RTS acquisition mode (with protein close-out). I don't think any of these testing results are quite Nature/Science/Cell magazine quality.

Since the same samples were run in both experiments, we can create a mock reference channel and match the data between acquisition methods. The `Schweppe_RTS_by-method` helps to show that the actual reporter ion data is rather different between the two acquisition modes.

#### The notebooks have all of the details:

- **[Schweppe_RTS_by-method](Schweppe_RTS_by-method.ipynb)** - This notebook compares the regular SPS MS3 acquisition method with the new real time search (RTS) acquisition method. Note that the RTS acquisition used here also a used protein close-out feature to limit redundant acquisition of peptide from abundant proteins.
- **[PXD017823_RTS_comparisons_IRS](PXD017823_RTS_comparisons_IRS.ipynb)** - This notebook compares cell line expression with edgeR for the RTS data (not the IRS adjusted values).
- **[PXD017823_Regular_comparisons](PXD017823_Regular_comparisons.ipynb)** - This notebook compares cell line expression with edgeR for the regular SPS MS3 data (not the IRS adjusted values).
- **[PXD017823_RTS_comparisons_IRS](PXD017823_RTS_comparisons_IRS.ipynb)** - This notebook compares cell line expression with edgeR for the RTS data after IRS was applied.
- **[PXD017823_Regular_comparisons_IRS](PXD017823_Regular_comparisons_IRS.ipynb)** - This notebook compares cell line expression with edgeR for the regular SPS MS3 data after IRS was applied.

#### Notebooks are also saved in [HTML format here](https://pwilmart.github.io/PXD017823_Real-Time-Search/)

---

## Conclusions

Does RTS seem to work? I think so. That is the first question to answer. I would have liked to see more A/B datasets in the second RST paper exploring the multiple topics one at a time more carefully. Is it a refinement paper or another proof of principle paper? We are all too smart for our own good. The dust doesn't have time to settle on the question of does RTS work before we start imagining ways to exploit the concept. I do think RST (and some related things that go with that) can strongly bias your data. That is at the heart of how you can exploit it, of course. I would probably want to know a lot about my samples (such as from some regular SPS MS3 runs) before deciding on an RTS method. RTS could be nice for scaling up studies to include more replicates where less instrument time per plex could be used.

The RAW files seem very similar to RAW files from the regular SPS MS3 method. The actual instrument method you start the run with does not have and MS3 scans in the mix. It is just a standard shotgun ID experiment (survey scans and MS2 scans). Then the MS2 scan stream is spied on by the computer running Comet. When scans return sufficiently good matches, the control computer highjacked to insert SPS MS3 scans. The notches selected in the MS2 scan are based on the assigned peptide sequence rather than intensity. Then the SPS MS3 data enters the RAW file data stream. There is the chance that an SPS MS3 scan can happen outside of the main instrument cycle (i.e. after the next survey scan). That scan structure would be different from the regular SPS MS3 data. Normally MS3 scans occur after MS2 scans, but always before the next MS1 scan. There were some precursor m/z values associated with the MS2 scans and their triggered MS3 scans that were not exactly the same. I am not sure if this was because of the monisotopic mass corrections or just some small bug in Orbiter. I also processed some demo data from Thermo with their implementation of RTS in [this repository](https://github.com/OHSU-Proteomics/Eclipse_to_Fusion_comparison). I did not see any precursor mass discrepancies in that data.

I love TMT data on the Tribrids. It is really amazing. You really need the SPS MS3 step to make TMT work. Real time search and intelligent acquisition methods have arrived. Hold my jet pack, I need to set up labs on the bottom of the ocean and on the moon (if you grew up in the sixties, you might get the jokes...). This all seems like something from the future, for sure. I think it may be ignoring the elephant in the room. Cutting the instrument time in half will result in even more incomprehensive datasets. [def: incomprehensive - incomprehensible comprehensive data] We really need to put more effort into understanding and interpreting these types of data. The current strategies of filtering unbiased results with highly biased annotation information seems fundamentally wrong. And probably skews the results to things we already knew. I do science to learn **new** things.

---

Phil Wilmarth <br />
May 28, 2020
