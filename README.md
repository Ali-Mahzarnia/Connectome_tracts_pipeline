
# Mrtrix APOE Pipeline GitHub Manual:

The Mrtrix APOE pipeline is a step-by-step process for analyzing diffusion MRI data and generating connectomes. Below are the detailed steps and commands used in the pipeline:

Step 1: Data Input

Read the necessary 4D, anatomical, bvec, and bval files.
Step 2: Preprocessing

Use the output of "dwigrdcheck" to swap the x and y components of the bvec file and flip the signs of all the components.
Step 3: Data Format Conversion

Convert the anatomical and 4D image to the mif format for Mrtrix usage, resulting in T1_mif and out_mif files.
Step 4: Compute Diffusion Tensor Metrics

Use "dwi2tensor" and "tensor2metric" commands to generate diffusion tensor-related metrics (dt, fa, dk, mk, md, ad, rd).
Step 5: Create Subject-Specific Masks

Use samba to create subject-specific label images.
Exclude cerebrospinal fluid (CSF) regions based on the atlas to generate a mask called "mask_of_label".
The dwigradcheck step is performed at this stage, as it requires the mask for correct bvec transformation. The transformation is then applied in Step 2 for all other subjects.
Step 6: Estimate Basis Functions

Use the "dwi2response" command with the dhollander algorithm to estimate basis functions for white matter (wm), gray matter (gm), and CSF.
Step 7: Compute Fiber Orientation Distribution (FOD) Files

Utilize the "dwi2fod" command with the estimated basis functions to compute FOD files: wmfod_mif, gmfod_mif, and csffod_mif.
Normalize the fod files to wmfod_norm_mif using the "mtnormalise" command.
Step 8: Generate Subset of Tracts

Create 10 million tracts using the gmwmSeed_coreg_mif as the seed image (representing the entire masked brain).
Subset and select 2 million tracts for each subject, using a fractional anisotropy (fa) cutoff of 0.1 and a maximum length of 1000 mm. Set a minimum length of 0.1 to avoid excessive noise.
Commands:

os.system('tckgen -backtrack -seed_image ' + gmwmSeed_coreg_mif + ' -maxlength 1000 -cutoff 0.1 -select 10000000 ' + wmfod_norm_mif + ' ' + tracks_10M_tck + ' -force')
os.system('tckedit ' + tracks_10M_tck + ' -number 2000000 -minlength 0.1 ' + smallerTracks + ' -force')
Step 9: Generate Connectomes
For each subject, we create 5 types of connectomes using the subject-specific labels:
a. SIFT Node Means:

Use the "tck2connectome" command with options -scale_invnodevol and -tck_weights_in, and provide sift_1M_txt as the weighting factor for each streamline.
b. SIFT:

Similar to SIFT Node Means, but without -scale_invnodevol.
c. Distances Connectome:

Use -scale_length and -stat_edge mean options to create a connectome based on streamline lengths.
d. FA Connectome:

Use -scale_file and -stat_edge mean options with mean_FA_per_streamline as the weighting factor for each streamline.
e. Plain Connectome:

Create a symmetric connectome with zero diagonal using options -symmetric and -zero_diagonal, which are already used in the previous four connectome commands.
Note: For the generation of mean_FA_per_streamline, use the "tcksample" command with the -stat_tck mean option to compute statistics from values along each streamline.

By following the above steps and executing the corresponding commands, the Mrtrix APOE pipeline can be used to process diffusion MRI data and obtain subject-specific connectomes for further analysis.
