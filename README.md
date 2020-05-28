# PXD017823_Real-Time-Search

Re-analysis of data from PXD017823 exploring real time search for TMT.

## Re-analysis

### Phil Wilmarth, OHSU

### May 27, 2020

---

## Any evidence for a mass measurement problem?

![Regular_deltamass_2plus](images/Regular_deltamass_2plus.png)

![RTS_deltamass_2plus](images/RTS_deltamass_2plus.png)

The 2+ deltamasses look very similar for either acquisition mode. There does not seem to be any evidence that the regular acquisition mode data has any mass measurement issues.

![Regular_deltamass_3plus](images/Regular_deltamass_3plus.png)

![RTS_deltamass_3plus](images/RTS_deltamass_3plus.png)

The 3+ deltamasses are also similar with no indications of any issues.

![Regular_deltamass_4plus](images/Regular_deltamass_4plus.png)

![RTS_deltamass_4plus](images/RTS_deltamass_4plus.png)

The 4+ deltamasses have fewer PSMs and the there is some peak broadening with increasing charge. There does not seem to be any indications that there is any significant mass measurement issue to be correcting. Thermo has improved their peak calling algorithms over the years. There may have been some issues many years ago that have become urban myths. It is probably not worth the computational time to test monoisotopic masses. The publication also uses a linear discriminant function for improved correct and incorrect discrimination. The deltaCN score in Comet needs a sufficient number of scored matches to be reliable. Trying to use very narrow tolerance searches of less than 20 PPM might be problematic. Wider tolerances of 50 to 100 PPM might be safer and would be plenty fast.

---

## Dataset Details

Each experiment was 12 fractions.

#### Overall numbers of scans:

Mode|Total MS2 Scans|Total MS3 Scans|MS3 fraction
----|---------------|---------------|------------
Regular SPS MS3|358,870|358,867|100%
RTS w/ close-out|462,711|41,647|9%

#### Filtered scan counts (1% PSM FDR):

Mode|Total|Non-empty MS3|Empty/Missing MS3|Fraction Non-empty
----|-----|-------------|-----------------|------------------
Regular SPS MS3|106,989|106,835|154|99.9%
RTS w/ close-out|116,671|30,604|86,067|26.2%

We can break it down by fractions, too.

![Reg_filtered_scans](images/Reg_filtered_scans.png)

![RTS_filtered_scans](images/RTS_filtered_scans.png)

Need a discussion of what RTS and protein close-out are doing.

> some data on the total intensities for each mode.

---

## Individual Proteins

Figure 4F shows two proteins where the acquisition mode resulted in different quantitative measurements. There are some data differences between the Schweppe et al. publication and my pipeline. The quantitative data I extract for the reporter ions are the peak heights. Peak heights are often a good proxy for peak areas (if peaks have the same shapes) and are commonly used for quantitative mass spectrometry measurements. The Gygi lab (and Proteome Discoverer) favor signal to noise ratios. These are signal quality metrics and not quantitative measures. S/N ratios do not have any units. They are compressed in range because of the division. If you have a measurement with noise and you have a noise estimate, the thing to do is to **subtract** the noise from the signal. This is actually already done on the data that is logged by Orbitrap instruments. FDR filtering differs a little between any pipelines. Protein inference and what peptides are considered usable for quantitative roll up is not well detailed in the publication, but that also differs by pipeline (probably more than for PSM FDR filtering).  

That said, what did my analysis have for the expression of these two proteins?

![Endoglin_protein](images/Endoglin_protein.png)

![ALPK2_protein](images/ALPK2_protein.png)

Note that ALPK2 is only identified by a single PSM. My pipeline does not report single peptide protein IDs. I was able to search through the top hit summaries and find the data. The data does not have any normalizations done for ALPK2. The single peptide from each acquisition mode was not the same, by the way.

The summmed protein values for Endoglin above are very similar to what is shown in Figure 4F. Endoglin had 4 and 3 peptides, respectively. We can look at the intensities for the individual peptides, too.

![Endoglin_peptides](images/Endoglin_peptides.png)

It is probably the one R.GPITSAAELNDPQSILLR.L pepide in the regular SPS MS3 data that drives the different expression pattern between the regular acquisition and the RTS acquisition.

Do these two edge cases have any real bearing on the discussion? Probably not. There can be interference in the m/z regions where reporter ions are (immonium and other low m/z ions). There can also be interference in the isolation window (co-eluting peptides and low-level background when precursor is close to noise levels). There can be biological reasons why specific peptides from specific parts of a protein may have expression levels at odds with more a bulk protein expression estimate. Generally speaking, summing up PSM reporter ions tends to stabilize the peptide and protein values (see the [dilution series notebook](https://pwilmart.github.io/TMT_analysis_examples/MAN1353_peptides_proteins.html)).

---

## Notebooks are also saved in [HTML format here](https://pwilmart.github.io/PXD017823_Real-Time-Search/)
