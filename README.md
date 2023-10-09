This repository hosts code and data for testing [C-ESM-EP](https://github.com/jservonnat/C-ESM-EP), and includes a version of C-ESM-EP as a git module (in directory [C-ESM-EP](C-ESM-EP))

## Principles for handling code and data

The principle of testing a C-ESM-EP version is to run the copy of a reference comparison and to compare its outputs with the reference results of that reference comparison. 

The reference comparison is included here as directory [reference_comparison](reference_comparison) and its reference results as directory [reference_results](reference_results). 

The reference results are dependant on the C-ESM-EP code version, on the underlying software combination, and on hardware. Regarding the dependency with the C-ESM-EP version, handling C-ESM-EP as a git (sub)module allows to tag C-ESM-EP version together with the results version. Because C-ESM-EP code also includes commands for choosing the underlying software environment (in file setenv_C-ESM-EP.sh), the dependency to that environment is also automatically managed. Regarding the dependency to hardware, one may handle multiple reference results versions as multiple branches of this repository. For the time being, branch `master` applies to ESPRI cluster `spirit`.

After cloning this repository (and possibly checking out the desired branch), and if you want to test with the recorded C-ESM-EP version, you must force getting its code by :

    git submodule update --init --recursive
	
When saving a new version of reference results, you should update the content of submodule C-ESM-EP with the (git-clean) version that was used (if changed); that version should also exist in the main C-ESM-EP repository. Commiting at top level will record C-ESM-EP's commit.
	
## Basis for the tests	

In reference results, the outputs undergoing comparison are the comparison-level html files for the atlas, which themselves refer to the individual graphic files (.png). 

The PNG files are compared using ImageMagick with command `compare -compose src -metric AE`. Because the PNG files also include (as a metadata) the CliMAF Reference Syntax expression (CRS) which unambiguously define the content of the graphic, the CRS are also compared when the graphics show a difference.


## Main steps for the tests

The main test script is [check_ref_comparison.sh](check_ref_comparison.sh) which executes the following : 

- create a test comparison which is the copy of the reference comparison
- launch the test comparison job (configured for sending a mail upon success or failure)
- launch a check job, which starts on termination of the test comparison job, and is configured for sending a mail; a normal termination of the job means a success of the comparison. 

The check job in turn executes script [compare_results.sh](compare_results.sh); for every component with a difference in some graphs, it generates an hmtl file showing both version for each such graph, and also, if their CRS (CliMAF Reference Syntax value) are not the same, both CRS. The location for such html files is a subdirectory `diff` of the test comparison's output directory, and it is indicated in check job's output (see example further below). This script heavily relies on CliMAF module tests/tools_for_tests.py.

## Executing main script, and output example

Executing the whole test implies the series of commands in script [run_example](run_example), which allows to select the C-ESM-EP version used. The script output indicates useful locations :

    Check of C-ESM-EP run for comparison 'reference_comparison' was launched as comparison 'test_comparison' in
       /home/ssenesi/cesmep/test_comparison
    Its outputs will be compared with reference results in 
       /home/ssenesi/cesmep/tests/reference_results
    Test success will appear as exit status of job 'Check_C-ESM-EP_ref_comparison' (jobid = 473578), both through:
      - a mail to user@ipsl.fr 
      - and in file /home/ssenesi/cesmep/tests/Check_C-ESM-EP_reference_comparison-473578.out

Check job output lies in the script execution dir (see the line just above), and the filename begins with "Check_C-ESM-EP_". Here is an example of such an output:

    Comparing outputs for components :
    AtlasExplorer
    4 pictures are different. See html diff file https://thredds-su.ipsl.fr/thredds/fileServer/ipsl_thredds/ssenesi/C-ESM-EP/test_comparison_ssenesi/AtlasExplorer/diffs/atlas_AtlasExplorer_test_comparison.html
    Atmosphere_zonmean
    53 pictures are different. See html diff file https://thredds-su.ipsl.fr/thredds/fileServer/ipsl_thredds/ssenesi/C-ESM-EP/test_comparison_ssenesi/Atmosphere_zonmean/diffs/atlas_Atmosphere_zonmean_test_comparison.html


