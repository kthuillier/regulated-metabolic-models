# Regulated metabolic model of Escherichia coli K-12 MG1655

Description of the processes used to update the regulated metabolic networks proposed by [\[Covert et al., 2004\]](https://www.nature.com/articles/nature02456) to be usable with modern rFBA simulation tools.

## Original files

**Metabolic network:** Model iJR904 from the BiGG Models database ([link](http://bigg.ucsd.edu/models/iJR904)).
It is a metabolic network of *Escherichia coli str. K-12 substr. MG1655*.
It is composed of 761 metabolites, 1075 reactions and 904 genes.

**Regulatory rules:** Set of regulatory rules defined in [Covert et al., 2004]. Rules are provided as a *.xls* file provided by the aforementioned paper supplementary materials ([link](https://static-content.springer.com/esm/art%3A10.1038%2Fnature02456/MediaObjects/41586_2004_BFnature02456_MOESM2_ESM.xls)).

## Updated regulated metabolic model

### Metabolic network

The metabolic network of iJR904 from the BiGG platform needs modification to be usable.

List of modifications:
* Create a new compartment `boundary``
    ```xml
    <listOfCompartments>
        <compartment id="e" name="extracellular space" constant="true"/>
        <compartment id="c" name="cytosol" constant="true"/>
        <compartment id="b" name="boundary" constant="true"/>
    </listOfCompartments>
    ```
* Add *boundary* metabolites associated with all metabolites in the `extracellular space`.
    > For instance, given the metabolite
    ```xml
    <species metaid="M_glc__D_e" sboTerm="SBO:0000247" id="M_glc__D_e" name="D-Glucose" compartment="e" hasOnlySubstanceUnits="false" boundaryCondition="false" constant="false" fbc:charge="0" fbc:chemicalFormula="C6H12O6">
        <sbml:annotation xmlns:sbml="http://www.sbml.org/sbml/level3/version1/core">
            ...
        </sbml:annotation>
      </species>
    ```
    > We add the associated `boundary` metabolite in the `boundary` compartment and with `boundaryCondition="true"`
    ```xml
    <species metaid="M_glc__D_b" sboTerm="SBO:0000247" id="M_glc__D_b" name="D-Glucose" compartment="b" hasOnlySubstanceUnits="false" boundaryCondition="true" constant="false" fbc:charge="0" fbc:chemicalFormula="C6H12O6">
        <annotation xmlns:sbml="http://www.sbml.org/sbml/level3/version1/core">
            ...
        </annotation>
    </species>
    ```

* Set all exchanges reactions has reversible.
    In the iJR904's default model, not all exchange reactions are reversible, contrary to what the [RegulonDB](https://regulondb.ccg.unam.mx/index.jsp) and [EcoCyc](https://ecocyc.org/ECOLI/NEW-IMAGE?type=REACTION&object=TRANS-RXN0-537) databases indicate.
    > For instance, the exchange reaction `R_EX_gly_e` (transport of glycine) is: `M_gly_e -> M_gly_b` (bounds: 0;99999).
    > The external glycine could never be absorbed by the bacteria.
    > In accordance with [EcoCyc](https://ecocyc.org/), it was transformed into: `M_gly_e <-> M_gly_b` (bounds: -99999;99999).

    Necessary to reproduce the simulations made by [\[Covert et al., 2004\]](https://www.nature.com/articles/nature02456).
    Without it, most experimental conditions do not allow biomass production.

* All gene IDs have been re-formatted such that: `G_bxxx` becomes `bxxx`.

### Regulatory network

From the *.xls* file provided in the Supplementary Material of [\[Covert et al., 2004\]](https://www.nature.com/articles/nature02456), we build an *SBML-Qual* file of the regulatory rules.

In this section, we describe the conversion process from the *.xls* file to the *SBML-Qual* file.

#### Cleaning

The first step consists of cleaning the regulatory rules and uniforming the species IDs and Boolean rule notations into a new *csv* file.

**Steps:**
1. Remove all genes that do not have a regulatory rule
2. Manually format all the regulatory rules `R` to match the following syntax:
    > R &nbsp;&nbsp;&nbsp;:= !R | (R1 | ... | Rn) | (R1 & ... & Rn) | ID | [N OP Real]
    > N &nbsp;&nbsp;&nbsp;:= MetaboliteID | ReactionID
    > OP := > | < | >= | <= | == | !=
    > ID &nbsp;&nbsp;:= GeneID | ProteinID | MetaboliteID | "String"

    For example, the `gntP = (Crp AND NOT (glcn(e)>0))` is rewritten as `gntP = (Crp & ![glcn(e) > 0])`
3. Match the reaction and metabolite references according to their IDs in the metabolic network (see previous section).
    For example, `glcn(e)` is transformed into `M_glcn_e`.
4. Replace all gene names used in the Boolean rules with their official IDs.
    For example, `ArcA` is replaced by `b4401`
5. External stimuli are kept in double quotes, *e.g.* `"Stringent"`.
    This model takes into account 9 stimuli: "stringent", "LB media", "heat shock", "high osmolarity", "stress", ""oxidative stress", "rich medium", "high NAD" and "dipyridyl".

**/!\\ Warning /!\\**
* Some regulatory rules are not correctly bracketed.
    + **Identification method:** Parse all regulatory rules and check if they are bracket mismatches.
    + **Solution:** Brackets have been added considering Boolean operator priority.
        > ! << | << &
* Different names have been used to describe the same gene.
    + **Identification method:** Compare the set $S$ of species occurring in regulatory rules with the set $S'$ of species having regulatory rules, being a reaction or a *boundary* metabolite, or a stimulus.
        All species in $S \setminus S'$ are potential naming errors.
    + **Solution:** For all the candidate species, check manually on [RegulonDB](https://regulondb.ccg.unam.mx/index.jsp) if it exists a *synonym gene ID* in the regulatory rules (*i.e.* in $S'$).
        If yes, replace all occurrences of the naming error with its synonym ID.
        If not, **panic**.
        List of applied corrections:
        > AllS -> b0504
        > AppY -> b0564
        > CitA -> b0619
        > CitB -> b0620
        > Cra -> b0080
        > FeaB -> b1385
        > GatR -> (b2087 | b2090) // GatR does not exist in the model, but GatR_1 and GatR_2 exist. Both have the same regulatory rule.
        > GutR -> b2707
* Metabolites missing from the metabolic network
    + **Identification method:** As for naming errors (see previously).
    + **Solution:** Currently, no accurate solution to solve this issue. Their values are always set to 0 instead.
        List of applied missing metabolites:
        > M_5dglcn_b, M_btn_b, M_cbi_b, M_h2o2_b, M_ppa_b, M_thym_b

#### From CSV to SBML-Qual

Conversion from the cleaned *csv* file to the *SBML-Qual* file is made using a custom *Python* JuPytR notebook.

**/!\\ Warning /!\\**
* Species states are currently always set as:
  + for metabolites:
    ```xml
    <qual:qualitativeSpecies qual:compartment="b" qual:constant="false" qual:id="MetaboliteID" qual:maxLevel="1" qual:initialLevel="0">
        <notes>
            <body xmlns="http://www.w3.org/1999/xhtml">
                <p>STATE 0:[0,0]</p>
                <p>STATE 1:]0,+inf[</p>
                <p>STATE 2:ND</p>
            </body>
        </notes>
    </qual:qualitativeSpecies>
    ```
  + for reactions:
    ```xml
    <qual:qualitativeSpecies qual:compartment="b" qual:constant="false" qual:id="ReactionID" qual:maxLevel="3" qual:initialLevel="0">
        <notes>
            <body xmlns="http://www.w3.org/1999/xhtml">
                <p>STATE 0:]-inf,0[</p>
                <p>STATE 1:[0,0]</p>
                <p>STATE 2:]0,+inf[</p>
                <p>STATE 3:ND</p>
            </body>
        </notes>
    </qual:qualitativeSpecies>
    ```
* Regulatory rules of form `1`, *i.e.* always activated, or `0`, *i.e.* always inhibited, are manually added.
* Constraints of the form `[N OP B]` are always parsed as `[N OP 0]`. Almost all the metabolite concentration and/or reaction flux constraints are according to 0.
    Rules are manually changed if `B` $\neq 0$.
    It is converted into the *sbml-qual* format such that:
    + If `N` is a metabolite `MetaboliteID`:
        * `OP := >`:
        ```xml
        <apply>
            <eq/>
            <ci> MetaboliteID </ci>
            <cn type="integer"> 1 </cn>
        </apply>
        ```
        * `OP := >=`:
        ```xml
        <apply>
            <or/>
            <apply>
                <eq/>
                <ci> MetaboliteID </ci>
                <cn type="integer"> 1 </cn>
            </apply>
            <apply>
                <eq/>
                <ci> MetaboliteID </ci>
                <cn type="integer"> 0 </cn>
            </apply>
        </apply>
        ```
        * `OP := <`:
        ```xml
        <apply>
            <and/>
            <apply>
                <eq/>
                <ci> MetaboliteID </ci>
                <cn type="integer"> 1 </cn>
            </apply>
            <apply>
                <eq/>
                <ci> MetaboliteID </ci>
                <cn type="integer"> 0 </cn>
            </apply>
        </apply>
        ```
        * `OP := <=`:
        ```xml
        <apply>
            <eq/>
            <ci> MetaboliteID </ci>
            <cn type="integer"> 0 </cn>
        </apply>
        ```
        * `OP := ==`:
        ```xml
        <apply>
            <eq/>
            <ci> MetaboliteID </ci>
            <cn type="integer"> 0 </cn>
        </apply>
        ```
        * `OP := =!`:
        ```xml
        <apply>
            <eq/>
            <ci> MetaboliteID </ci>
            <cn type="integer"> 1 </cn>
        </apply>
        ```
    + If `N` is a reaction `ReactionID`:
        * `OP := >`:
        ```xml
        <apply>
            <eq/>
            <ci> ReactionID </ci>
            <cn type="integer"> 2 </cn>
        </apply>
        ```
        * `OP := >=`:
        ```xml
        <apply>
            <or/>
            <apply>
                <eq/>
                <ci> ReactionID </ci>
                <cn type="integer"> 1 </cn>
            </apply>
            <apply>
                <eq/>
                <ci> ReactionID </ci>
                <cn type="integer"> 2 </cn>
            </apply>
        </apply>
        ```
        * `OP := <`:
        ```xml
        <apply>
            <eq/>
            <ci> ReactionID </ci>
            <cn type="integer"> 0 </cn>
        </apply>
        ```
        * `OP := <=`:
        ```xml
        <apply>
            <or/>
            <apply>
                <eq/>
                <ci> ReactionID </ci>
                <cn type="integer"> 0 </cn>
            </apply>
            <apply>
                <eq/>
                <ci> ReactionID </ci>
                <cn type="integer"> 1 </cn>
            </apply>
        </apply>
        ```
        * `OP := ==`:
        ```xml
        <apply>
            <eq/>
            <ci> ReactionID </ci>
            <cn type="integer"> 0 </cn>
        </apply>
        ```
        * `OP := =!`:
        ```xml
        <apply>
            <or/>
            <apply>
                <eq/>
                <ci> ReactionID </ci>
                <cn type="integer"> 0 </cn>
            </apply>
            <apply>
                <eq/>
                <ci> ReactionID </ci>
                <cn type="integer"> 2 </cn>
            </apply>
        </apply>
        ```

    There are only 4 metabolite concentration constraints that are not according to 0:
    > [M_k_b > 1.0]
    > [M_phe__L_b > 10.0]
    > [M_pi_b < 4e-09]
    > [M_nh4_b > 2.0]
    They are manually corrected in the generated *SBML-Qual* file.

* All stimuli are added into a new compartment `env`.
    Stimulus `pH` is manually added to the list of species:
    ```xml
    <qual:qualitativeSpecies qual:compartment="env" qual:constant="false" qual:id="pH" qual:maxLevel="3" qual:initialLevel="0">
        <notes>
            <body xmlns="http://www.w3.org/1999/xhtml">
                <p>STATE 0:[0,4[</p>
                <p>STATE 1:[4,7[</p>
                <p>STATE 2:[7,+inf]</p>
                <p>STATE 3:ND</p>
            </body>
        </notes>
    </qual:qualitativeSpecies>
    ```

## Model validation

The model has been validated by reproducing [\[Covert et al., 2004\]](https://www.nature.com/articles/nature02456) phenotype predictions ([experimental conditions](https://static-content.springer.com/esm/art%3A10.1038%2Fnature02456/MediaObjects/41586_2004_BFnature02456_MOESM3_ESM.xls), [phenotype predictions](https://static-content.springer.com/esm/art%3A10.1038%2Fnature02456/MediaObjects/41586_2004_BFnature02456_MOESM5_ESM.xls)).

Regulated Flux Balance Analysis (rFBA) has been run using the tool [*FlexFlux*](http://lipm-bioinfo.toulouse.inrae.fr/flexflux/documentation.html) [\[Marmiesse et al., 2015\]](https://bmcsystbiol.biomedcentral.com/articles/10.1186/s12918-015-0238-z).

**/!\\ Simulation In Progress /!\\**

### Regulatory network

Some rFBA simulations of our model do not match the predicted behaviors described in [\[Covert et al., 2004\]](https://www.nature.com/articles/nature02456).

#### Medium with `N-Acetyl-D-glucosamine`

##### Problem

RFBA simulations of our model in media composed of `N-Acetyl-D-glucosamine` (`M_acgam_b`) do not reproduce the predicted behaviors.
With only `N-Acetyl-D-glucosamine` as a carbon source, the reaction `R_G1PACT` (glucosamine-1-phosphate N-acetyltransferase) is necessary for the model to grow.
The sequence of regulatory rules controlling the activation of `R_G1PACT` is:
> `NagC := NOT ([M_acgam_b > 0] | [R_AGDC > 0])` ;
> `b3730 := NagC` ;
> `R_G1PACT := b3730`.
In this case, `R_G1PACT` is always inhibited if there is `N-Acetyl-D-glucosamine` in the medium.
Therefore, no biomass can be produced considering this set of regulatory rules.

##### Bibliographic review

Review of the references used to build the regulatory rules of `NagC` and `b3730`.

- For `NagC`:
In \[F. C. Neidhardt et al., 1996\], it is said (vol.1, ch.75, p.1160)
> Nag and its analogs (including the antibiotic streptozotcin) are taken up through the Nag PTS.
> The structural gene encoding nagE is part of a nag regulon which has been sequenced completely in E. coli.
> Two nag operons are clustered on the chromosome and transcribed from two divergent promoters which share a central CRP binding site.
> The first operon expresses negA at a semiconstitutive level, while the second contains, in addition to structural genes for catabolic enzymes, the gene nagC, encoding a repressor proteins.
> NagC binds Nag 6-phosphate (and perhaps D-glucosamine 6-phosphate) as the inducer (195-197, 293, 295).

where `Nag 6-phosphate` is equivalent to `R_AGDC` and `D-glucosamine 6-phosphate` is equivalent to `M_acgam_b` in our model. \
The references are:
- **195:** J. A. Plumbridge. Sequence of the *nagBACD* operon in *Escherichia coli* K12 and pattern of transcription within the *nag* regulon. Mol. Microbiol. 3:505-515 (1989).
- **196:** J. A. Plumbridge. Repression and induction of the *nag* regulon of *Escherichia coli* K12: the roles of *nagC* abd *nagA* in maintenance of the uninduced state. Mol. Microbiol. 35:2053-2062 (1991).
- **197:** J. A. Plumbridge. CAP and *Nag* repressor binding to the regulatory regions of the *nagE-B* and *manX* genes of *Escherichia coli*. J. Mol. Biol. 217:661-679 (1991)
- **293:** A. P. Vogler and J. W. Lengeler. Analysis of the *nag* regulon from *Escherichia coli* K12 and *Klebsiella pneumoniae* and of its regulation. Mol. Gen. Genet. 219:97-105 (1989).
- **295:** A. P. Vogler, S. Trentmann ad J. W. Lengeler. Alternative route for biosynthesis of amino sugars in *Escherichia coli* K12 mutants by means of a catabolic isomerase. J. Bacteriol. 171:6586-6592 (1989).

It is worth noting that the binding of `NagC` with `M_acgam_b` is not sure (and is not highlighted in the aforementioned references).

In [EcoCyc](https://ecocyc.org/gene?orgid=ECOLI&id=EG10636), `NagC` is inhibited by `M_acgam_b` and possibly controlled by other external factors (undefined).

- For `b3730`:
The regulatory rule `b3730 := NagC` is confirmed in [\[J. Plumbridge, 2001\]](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC29661/):
> In the absence of an environmental supply of amino sugars,
NagC represses the divergent nagE-BACD operons, which are
necessary for the utilisation of GlcNAc as a carbon source (1), and activates the expression of one promoter of the glmUS operon that is part of the biosynthetic pathway for the formation of GlcN-6-phosphate (GlcN-6-P) and UDP-GlcNAc (2).

and
> The glmUS operon, which is regulated positively by NagC

where `glmUS` is equivalent to `b3730`.

##### Proposition

**Undefined**

## References
\[F. C. Neidhardt et al., 1996\] F. C. Neidhardt, R. Curtiss et al. Escherichia coli and Salmonella: cellular and molecular biology. ASM Press (1996). ISBN: 9781555810849-1555810845.\
[\[J. Plumbridge, 2001\]](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC29661/) Plumbridge J. DNA binding sites for the Mlc and NagC proteins: regulation of nagE, encoding the N-acetylglucosamine-specific transporter in Escherichia coli. Nucleic Acids Res. 2001 Jan 15;29(2):506-14. doi: 10.1093/nar/29.2.506. PMID: 11139621; PMCID: PMC29661.\
[\[Covert et al., 2004\]](https://www.nature.com/articles/nature02456) Covert, M., Knight, E., Reed, J. et al. Integrating high-throughput and computational data elucidates bacterial networks. Nature 429, 92–96 (2004). https://doi.org/10.1038/nature02456\
[\[Marmiesse et al., 2015\]](https://bmcsystbiol.biomedcentral.com/articles/10.1186/s12918-015-0238-z) Marmiesse, L., Peyraud, R. & Cottret, L. FlexFlux: combining metabolic flux and regulatory network analyses. BMC Syst Biol 9, 93 (2015). https://doi.org/10.1186/s12918-015-0238-z\