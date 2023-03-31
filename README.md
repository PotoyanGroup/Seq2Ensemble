# [Seq2Ensemble](https://potoyangroup.github.io/Seq2Ensemble/) 
This is a brief set of instructions for the Seq2Ensemble web tool used to generate protein conformational ensembles consistent with Residual Dipolar Coupling (RDC) data.

Introduction

Seq2Ensemble is available online as a Colab notebook. Colab is a web-based interactive python computing environment, or “notebook”, offered free of charge by Google. A notebook contains blocks of python code that can be executed by pressing the “play” buttons or Shift+Enter on highlighted cells. For more details on Google Colab visit https://colab.research.google.com/.

Seq2Ensemble is simply a set of python code blocks that will take a protein structure file (pdb format) and a file containing the experimental RDCs and perform:

1.	coarse-grained molecular dynamics (cgMD) on the protein structure. 
2.	r.m.s.d. based K-means clustering on the protein conformations produced by the cgMD trajectory to generate subsets of structures that may represent a valid ensemble.
3.	Nonlinear least squares fitting of ensembles against the experimental RDCs.

By comparing the fits obtained from different (sized) ensembles, you can determine which set of conformations best represents your protein in solution. 

Note:
The layout of Seq2Ensemble may change and additional functionality made available.  Regardless, the prompts in the notebook should be adequate for most people to utilize the resource if these instructions no longer match the notebook format.


Instructions

1.	Go to https://potoyangroup.github.io/Seq2Ensemble/
	Note: Work in Google Chrome for the best performance

2.	Generate a PDB file with AlphaFold2 (if you already have a PDB go to step 3)
	-Click on the “Open in Colab” link below “From sequence to structure via AlphaFold”.
Note: This will take you to a ColabFold (Mirdita, M. et al. ColabFold: making protein folding accessible to all. Nat. Methods 19, 679–682 (2022))
-Follow the instructions at the bottom of the ColabFold page.
In brief, paste your FASTA sequence into the “query_sequence” field, click on the “Runtime” dropdown at the top of the page, and select “Run all”. 
Note: This may take some time but your result.zip file should automatically download. If it does not, click on the folder icon on the left of the page, highlight the result.zip file, click on the 3 dots to the right, and select “Download”. Retrieve your pdb file from the zip and return to https://potoyangroup.github.io/Seq2Ensemble/.

3.	Generate a conformational ensemble
-Click on the “Open in Colab” link below “From structure to conformational ensemble“.
-Familiarize yourself with the notebook:
You should see a page that looks like the figure below. If it doesn’t, the small arrows left of the cells will display more information.
On the left, there is a folder icon. All of the files generated in or uploaded to the notebook can be found here or within one of the subfolders. If for some reason the session fails, you may need to “Restart runtime” under the “Runtime” menu or “Disconnect and delete runtime” before restarting.
 

	-Start your session by clicking the “play” button under “Run this cell to set everything up”
Note: It may take 3 to 7 minutes for the python libraries, the OpenAwsem CG simulation software, and its dependencies to download and install.
When cells are running a revolving animation will show around the “stop” button. When it finishes, the “play” button will return, typically with a green checkmark next to it.

-Click the “play” button underneath “Upload a PDB file”.
-Click the “Choose Files” button and select your PDB file.

-Input the parameters under “Run Molecular Dynamics with the AWSEM Coarse-Grained Force-Field for Proteins” 
	Enter the desired temperature in Kelvin and the desired number of steps.
	Structures are saved every 2000 steps.
-Click on the Play icon to start the cgMD run
Note: The initialization will take a few minutes but shortly you will see the timesteps quickly advance. Tens of thousands of timesteps can run in a matter of minutes. cgMD simulations produce uncorrelated conformations much faster than all-atom simulations and it is up to you, the complexity of your system, and the amount of colab “compute units” you have available to decide on how long the simulation needs to run. The frame below the cell will grow until a scroll bar appears. If you need to navigate up or down the page, make sure you are using the other scroll bar, farthest to the right.

-After the simulation is complete, click the “play” button beneath “Install Analysis Tools”. 
Note: This step is necessary to run the subsequent cells.

	-Choose the number of clusters or n member ensembles to generate and click “play”.
This step will cluster your trajectory according to the backbone r.m.s.d. with respect to the average structure. Conformations with similar r.m.s.d. will fall into the same cluster. From each cluster, a single “centroid” that exhibits the mean r.m.s.d. of the cluster will be extracted as the representative structure. If you set n_clusters to 10, you will end up with 10 ensembles. Cluster 1 will be a 1-member ensemble, cluster 5 will be a 5-member ensemble, and so on. Note that each subsequent clustering iteration will take longer, so if you have a long trajectory and choose to generate 20 clusters, it may take a long time.
	
-Under “Least Squares Fitting of RDCs to Ensembles”, click “play”, “Choose File” and upload your RDC file.
File format: At the moment, your RDC file should contain only two tab separated columns. The first column should be the residue ID and the second column should be RDC values. The residue IDs must match the residues in your PDB file and your PDB should be indexed from 1. For instance, if the canonical residue IDs start with residue 5 in your structure file, you will need to subtract all the residue IDs in your RDC file by 4.
The least squares fitting will iterate through all the ensembles generated in the previous clustering step. This step can be very time consuming with a large number of n_clusters. After all the fittings have been completed, plots of the back-calculated ensemble RDCs vs experimental RDCs will display with their corresponding R factors describing the goodness of the fits.

	- Click the “play” button under “Zip and Download the Results”
Before archiving the results, amino acid sidechains will be modeled onto the coarse grained backbone structures composing your ensembles. After this, a “seq_to_ensemble_output.zip” file should automatically download. This file contains: (i) Folders “all_atom_cluster_1” to n with the ensemble structures in pdb format, and (ii) Folder rdc_fitting_results with the files “ensemble_rdcs.csv” (calculated vs experimental RDCs), “ensemble_R_values.csv” (list of R-factors), and a “par” folder (the alignment tensor parameters per each ensemble member).


NOTE: You also have the option to energy minimize the side chains of the structures composing an ensemble.
Press “play” under “What are the best ensembles?” and you will be shown the ensemble ids and corresponding R values in order of the best fits (lower R is better).

Under “Energy Minimize an Ensemble”, enter the id of the fit whose structures you’re interested in analyzing more closely.  A backbone restrained energy minimization in implicit solvent will be performed and each of the minimized structures will be saved to a new folder, archived, and downloaded as “minimized_cluster_x.zip”.  If you compare these to the “all_atom_cluster_x” structures, you will notice some of the side chains have moved.

Enjoy!


Potential Issues
The Runtime menu at the top has an option to “change runtime type”.  This should automatically be set to “GPU”, but if your simulation is running slowly, it might be set to “CPU” and should be changed.

cgMD simulation temperature may not correspond exactly to physical temperatures.  If you do not obtain a nice fitting ensemble, you may need to adjust your run temperature by 10-20K or run your simulation longer.
If you run into errors that can’t be resolved by restarting the notebook, please post them to the issues page at https://github.com/PotoyanGroup/Seq2Ensemble/issues

