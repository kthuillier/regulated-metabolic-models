# Regulated Metabolic Networks

This repository contains regulated metabolic networks in *SBML* (version 3) **[1]** and *SBML-qual* **[2]** file formats.
All models are compatible with the rFBA **[3]** simulation tool [`FlexFlux`](http://lipm-bioinfo.toulouse.inrae.fr/flexflux/index.html) **[4]**, and the [`Cobra Toolbox`](https://opencobra.github.io/cobratoolbox/stable/index.html) **[5]**.

## Regulated metabolic networks

It contains 3 regulated metabolic networks.

|             Bacteria              |     Model      | Introduced in | #Reactions | #Genes | #Regulatory rules |
| :-------------------------------: | :------------: | :-----------: | :--------: | :----: | :---------------: |
|              E.coli               | *core-carbon*  |    **[3]**    |     20      |   11    |         11         |
| E.coli *str. K-12 substr. MG1655* | *medium-scale* |    **[7]**    |     113      |   105    |         218         |
| E.coli *str. K-12 substr. MG1655* | *large-scale*  |    **[8]**    |     1075      |   1011    |         1777         |

## Content descriptions

Models are grouped by bacteria.
Each model is structured as follows:
```
- bacteria
|- model
|  |- csv                        <- Human-friendly description
|  |  |- metabolites.csv            of the model in CSV format
|  |  |- reactions.csv           
|  |  |- regulations.csv
|  |- sbml                       <- Model description in SBML
|  |  |- metabolic_network.sbml  <- SBML (v3)
|  |  |- regulatory_network.sbml <- SBML-qual
|- experiments
|  |- JSON files
|- description.json              <- File describing the full 
|                                   instance
```

For the `large-scale` model, a `readme.md` file is provided.
It describes the reconstructing protocol from the model description in **[8]**.

### CSV file descriptions

Each model is associated with three CSV files describing the metabolites, reactions, and regulatory rules.
They are structured as follows:
- `metabolites.csv`
> ID,Fullname,Compartment,BoundaryCondition

- `reactions.csv`
> ID,GeneAssociation,Fullname,Reaction,[LowBound,UpBound]

- `regulations.csv`
> ID,Fullname,Rule

### JSON Experiment descriptions

For each model, we provide experimental conditions for rFBA simulations to reproduce the figures and described dynamics in its introductory paper.

Each experimental condition is described in a JSON file having the following structure:
```json
{
    "ID": "<Experiment ID>",
    "title": "<A full title>",
    "SimulationParameters": { // Simulation parameters
        "initBiomass": 0.003, // <- Initial biomass
        "timeStep": 0.01,     // <- rFBA timestep length
        "nSteps": 1200        // <- Simulation duration
    },
    "initConcentrations": {   // Initial substrate 
        "Met_ext-1": 0.3,     // concentrations (mM)
        "Met_ext-2": 9999,    // Format:  
        "Met_ext-3": 0        // "Metabolite_ID": Concentration
    },
    "initRegs": {                   // Initial gene state
        "RegulatoryProteinID": 0,   // Format:
        "GeneID": 1                 // "Gene_ID": state (0|1)
    },
    "constraints": {          // Special constraints
        "ko": [               // 1) Knockout Mutation
            "GeneID",         //    Gene always set to 0
        ],
        "bounds": {           // 2) Special flux bounds
            "Reaction_ID": [  // Format:
                LB,           // "Reaction": [LB, UB]
                UB
            ]
        }
    }
}
```

## References

> **[1]** Hucka, M., Finney, A., Sauro, H. M., Bolouri, H., Doyle, J. C., Kitano, H., Arkin, A. P., Bornstein, B. J., Bray, D., Cornish-Bowden, A., et al. (2003). The systems biology markup language (sbml): a medium for representation and exchange of biochemical network models. Bioinformatics, 19(4):524–531.

> **[2]** Chaouiya, C., Bérenguier, D., Keating, S. M., Naldi, A., Van Iersel, M. P., Rodriguez, N., Dräger, A., Büchel, F., Cokelaer, T., Kowal, B., et al. (2013). Sbml qualitative models: a model representation format and infrastructure to foster interactions between qualitative modelling formalisms and tools. BMC systems biology, 7:1–15

> **[3]** Covert, M. W., Schilling, C. H., and Palsson, B. (2001). Regulation of gene expression in flux balance models of metabolism. Journal of Theoretical Biology, 213(1):73–88.

> **[4]** Marmiesse, L., Peyraud, R., and Cottret, L. (2015). Flexflux: combining metabolic flux and regulatory network analyses. BMC systems biology, 9(1):1–13.

> **[5]** Laurent Heirendt, Sylvain Arreckx, Thomas Pfau, Sebastian N. Mendoza, Anne Richelle, Almut Heinken, Hulda S. Haraldsdottir, Jacek Wachowiak, Sarah M. Keating, Vanja Vlasov, Stefania Magnusdottir, Chiam Yu Ng, German Preciat, Alise Zagare, Siu H.J. Chan, Maike K. Aurich, Catherine M. Clancy, Jennifer Modamio, John T. Sauls, Alberto Noronha, Aarash Bordbar, Benjamin Cousins, Diana C. El Assal, Luis V. Valcarcel, Inigo Apaolaza, Susan Ghaderi, Masoud Ahookhosh, Marouen Ben Guebila, Andrejs Kostromins, Nicolas Sompairac, Hoai M. Le, Ding Ma, Yuekai Sun, Lin Wang, James T. Yurkovich, Miguel A.P. Oliveira, Phan T. Vuong, Lemmer P. El Assal, Inna Kuperstein, Andrei Zinovyev, H. Scott Hinton, William A. Bryant, Francisco J. Aragon Artacho, Francisco J. Planes, Egils Stalidzans, Alejandro Maass, Santosh Vempala, Michael Hucka, Michael A. Saunders, Costas D. Maranas, Nathan E. Lewis, Thomas Sauter, Bernhard Ø. Palsson, Ines Thiele, Ronan M.T. Fleming, Creation and analysis of biochemical constraint-based models: the COBRA Toolbox v3.0, Nature Protocols, volume 14, pages 639–702, 2019 doi.org/10.1038/s41596-018-0098-2.

> **[6]** Kerian Thuillier, Caroline Baroukh, Alexander Bockmayr, Ludovic Cottret, Loïc Paulevé, and Anne Siegel (2021). Learning Boolean Controls in Regulated Metabolic Networks: A Case-Study. In: Computational Methods in Systems Biology. CMSB 2021. Lecture Notes in Computer Science, vol 12881. Springer, Cham.
> 
> **[7]** Covert, M. and Palsson, B. (2002). Transcriptional regulation in constraints-based metabolic models of escherichia coli. The Journal of biological chemistry, 277:28058–64. 

> **[8]** Covert, M. W., Knight, E. M., Reed, J. L., Herrgard, M. J., and Palsson, B. O. (2004). Integrating high-throughput and computational data elucidates bacterial networks. Nature, 429(6987):92–96.

