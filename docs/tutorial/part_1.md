# Tutorial Part 1: Analysis of a sepsis dataset

## Introduction
We will showcase the use of MLModNet with the GAinS sepsis dataset containing proteomic and transcriptomic datasets for 800 patient-samples.

## Running MLModNet on a sepsis dataset

### Preprocessing files
MLModNet is a modular package with various parts.
First you need to modify the two options files. First, in variability_analysis.py you need to modify lines 11-12: these are the lines telling our script what the working directory is, and where the options file is. Change these two accordingly:
```python
working_direct = "/home/username/MLModNet/code/"
OPTIONS_FILE = f"{working_direct}options/parameters_for_sepsis.tsv"
```

Then modify the actual files, so that they represent your data. The file is tab-delimited and contains options for the variability processing part of the pipeline.
Generally:
- `modality_parameters_file_path` points to the file with the specification of your modality files format. 
- `modality_parameters_file_sep` specifies the delimiter character for that file, by default tab.
- `clinical_file_path` points to the file with clinical metadata about the samples.
- `final_unified_ID` is the name of the column from the clinical metadata file which is the ID to use for subsequent analyses, once it is unified in the preprocessing steps.
- `clinical_file_sep` specifiec the delimiting character, by default tab.
- `random_seed_np` and `random_seed_random` are the random seed values.
- `output_dir_for_variability_plots` points to where the variability plots should be saved.
- `output_dir_for_processed_data` points to where the processed data should be saved.
- `samples_to_use_file` point to a file, which should contain a subset of rows of the clinical metadata file in case a subset is to be analysed.

In particular, for this sepsis analysis, we will use the following parameters:
```
modality_parameters_file_path	"/home/username/MLModNet/code/options/modality_preprocessing_parameters.tsv"
modality_parameters_file_sep	"	"
clinical_file_path	"/home/username/data_combat/CBD-CLIN-00006/COMBAT_clinical_basic_data_freeze_131020.txt"
final_unified_ID	"COMBAT_participant_timepoint_ID"
clinical_file_sep	"	"
random_seed_np	42
random_seed_random	42
output_dir_for_variability_plots	"/home/username/MLModNet/output/variability_plots/"
output_dir_for_processed_data	"/home/username/MLModNet/output/preprocessed_sorted_data_nonNA_only/"
samples_to_use_file	"/home/username/MLModNet/code/full_data_patients.csv"
``` 

Next we need to modify the file with the modality specifications, pointed to by `modality_parameters_file_path`. This file contains:
- `modality_name`
- `file_path`
- `sep`
- `transpose`
- `rename_sample`
- `feature_ID_name`
- `modality_ID_name`

In our case these will take the following values:
```
modality_name	file_path	sep	transpose	rename_sample	feature_ID_name	modality_ID_name
timstof	/home/username/combat_integration_files/CBD-PRT-00002/processed/ints.tsv	"	"	0	1		COMBAT_proteomics_sample_ID
logRNAcpm	/home/username/combat_integration_files/CBD-RNA-00004/Logcpm_143_23063.txt	"	"	0	1		RNASeq_sample_ID
```

In this way we have set up the environment to unify the datasets - so that they all have the same format - and perform variability calculations, to sort the features based on how much more variable they are than expected at their mean level across the population of features.

Let's now perform this reformatting and variability analysis.

To run it for a specific dataset, using log scale for the plotting, and with 0.5 as the cut-off value for gene variability, we will run the following command.
```bash
python variability_analysis.py {dataset} 0.5 --log_scale --regression --modality_parameters_file_path {dataset_file}
```

To do this for all the modalities, we will run the following commands:

```bash
 python /home/username/MLModNet/code/run_variability.py
```
This function runs the variability_analysis.py function for all the lines from the `modality_parameters_file_path` file. (Remember to change the working_direct and options_file pointers if you are working in a different context.)

This should run quickly and produce processed files and associated variability plots. Now we can proceed to the COGENT stability analyses.

### COGENT

COGENT is the lengthiest part of this pipeline. In this part, for each modality, we will determine what network construction procedure results in most perturbation-resistant networks. First, the setup.

`/home/username/MLModNet/COGENT_options.csv` has a bunch of options that you can play around with, but the defaults can be generated through running:

```bash
python /home/username/MLModNet/code/create_COGENT_options_from_processed_files.py
```
This will create a file, called `COGENT_options_for_sepsis.csv` in the `options` folder. It will contain the default options for COGENT with Pearson correlation. (Remember to change the working_direct and options_file pointers if you are working in a different context.)

Now we actually need to run COGENT, and there is a convenient wrapper that uses all the previous files we generated to figure out how to run COGENT.

```bash
python COGENT_python/cogent_subsets_code.py '/home/username/MLModNet/code/options/COGENT_options_for_sepsis.csv'
```

This will take a while, but will calculate the optimal networks and network construction approaches, as well as levels of variability that they exhibit, compared against randomized networks. 

This will generate a bunch of files, which we can now proceed to visualizing. (Remember to change the working_direct and options_file pointers if you are working in a different context.)

We will run `slice_and_surface_plots.py`:
```bash
python /home/username/MLModNet/code/slice_and_surface_plots.py
```

This should produce visualizations of how consistent the networks stay upon minor perturbations with a range of network construction approaches.

Finally, let's create a multilayer network!

### Multilayer Networks!

Now, we can run `multilayer_network_stability.py`:

## Example
Provide a detailed example that applies the concepts taught.

## Conclusion
Summarize what the user has learned in this part.
