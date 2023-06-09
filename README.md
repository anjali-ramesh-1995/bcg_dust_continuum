# The Dust Continuum Morphology of Brightest Cluster Galaxies

## Project Description

In this project, we analyze the physical properties of objects (galaxies) in radio-wavelength images (fits data type) using statistical methods like 2D Gaussian modeling, Chi-square fitting, and object surface profile modeling. For a more detailed description of the science, refer to paper in prep. 

### Some preliminary steps and information

1. git clone https://github.com/anjali-ramesh-1995/bcg_dust_continuum.git
2. In the bcg_parameter_file.py file replace "/Users/arames52/" wherever found to folder path you cloned the repository to
3. Assuming conda is installed - conda activate bcg_project/
4. conda install -r requirements.txt
5. **src** folder has all the codes
6. **notebook** folder contains data and plots 
7. Activate the environment and run ...


## Data Description

### Primary Data 

Calibrated measurement sets obtained after executing *scriptforPI.py from the downloaded raw observations from ALMA Science Archive with project id 018.1.00828.S and 2019.1.01027.S (P.I. Noble). 

### Supporting Data

Multi-wavelength Flux Catalog - Provided the coordinates of the objects, some functions in data_ingestion.py help in querying catalog information and image cutouts from several surveys and telescope archive data like Dark Energy Survey (DES), the Spitzer Space Telescope, Herschel Space Observatory. For more information look at data_ingestion.py.

### bcg_parameter_file.py
Specifies all the parameters that are used


## Data Analysis

Note - helper_functions.py (hf), bcg_parameter_file.py (bpf), data_ingestion.py (di)   are imported into most scripts

### Data Reduction and Imaging with CASA

* Input - Calibrated measurement sets located at "/Users/arames52/bcg_dust_continuum/notebook/data/Measurement_Sets/"
* Code - casa_imaging.py 
* Functions
    1. get_tclean_parameters() {hf} - Return list of weighting and tapering options read from weighting_tapering_values.txt
    2. folder_creation() {hf}  - creates folders for output storage
    3. alma_imaging - given measurement set, and list of weights and tapering schemes - 
        * run TCLEAN with defined parameters in bpf
        * convert .pbcor image file to .fits for convenient usage in python
    
* Note - If auto-masking algorithm to be performed, change to interactive = True and usemask = "auto-multithresh"

### 2D Gaussian Profile Modeling with CASA

* Input - .fits image data produced from different imaging parameters
* Code - gaussian_profile_fitting.py (gpf)
* Functions - 
    1. region_params() - returns region that encloses the object
    2. imfit_routine() - returns a dictionary of imfit calculated on Briggs, naturally weighted and tapered images
    3. imfit_single_image() - returns a dictionary with the imfit routine performed on naturally weighted images
    4. The regions were manually selected such that it encompasses 3 sigma source.
    5. Input file notebook/data/Derived_Data/bcg_regions.txt

* Note - Run this in CASA terminal

### Image Properties - RMS, S/N - with CASA

* Input - .fits image data
* Code - image_properties.py
* Functions - 
    1. signal_rms() - runs CASA's IMSTAT to calculate properties in a given region
* BCG region file - bcg_regions.txt - if region radius has to be changed, then modifications to be done on this file

* Note - Run on CASA terminal

### Multi-wavelength catalog download and compilation

Download object information from different surveys and telescope and save as csv file 

# Web Scraping Multi-Wavelength BCG Fluxes

## Sources
1. DES query interface - https://datalab.noirlab.edu/query.php?name=des_dr2.main
    - Python module astro-datalab can be used to query the data
2. Spitzer SWIRE - https://irsa.ipac.caltech.edu/cgi-bin/Gator/nph-dd - for direct download
    - However, we can use Python's curl function to directly download from the catalog database
3. Herschel HerMES - https://irsa.ipac.caltech.edu/cgi-bin/Gator/nph-dd
    - urllib.request can be used to download give the catalog url (specified in params.py)
4. ALMA flux - Calculated from IMFIT

## Input files/parameters
1. Coordinate file of sources in the format name|ra|dec
    - Read as csv file
2. HerMES Tar file download link

# Code Description

## data_ingestion.py

 **Replace /Users/arames52/ by the directory in which you have stored the bcg_dust_continuum folder**

### download_des_catalog(ra, dec, fov)
- Given the ra, dec and fov (default value 0.005 degrees) of target object, we download the magnitudes of sources within 20 arcseconds of the bcg coordinate.
- The output is a dictionary with keys being bcg name and values being dataframe of object magnitudes
- This is stored as a pickle file

### download_swire_catalog(name, ra, dec, bcg)
- Given the catalog name, ra, dec and id of the target source, we download the flux of objects within 20 arcseconds of the target.
- The output file is stored as csv files for individual bcgs under data/catalogs/SWIRE_catalog/

### download_hermes_catalog(catalog_url)
- We download the DR3 xID24 catalog of all HerMES sources
- This is stored for each field (CDFS, ES1, XMM)


## multiwavelength_catalog.py

Reads the downloaded DES, SWIRE, HerMES and ALMA data and forms a combined multiwavelength flux catalog for all 26 BCGs


### catalog_matching(c, df)
- Takes in the bcg skycoord and catalog dataframe and returns the source and nearest neighbours
### mag_to_mJy()
- Converts DES magnitudes to mJy units
        *  mJy is the desired unit for SED fitting
### vega_to_mjy()
- Converts Sparcs Vega magnitudes to mJy units
### multiwavelength_catalog()
- Given bcg name, returns multiwavelength flux dictionary and nearest neighbours

### Surface Brightness Profile Fitting with Statmorph

dust_continuum_morphology.py

* Functions - read_alma_fits(), image_cutout(), compute_sersic(), save_sersic_results()

### compute_sersic()
- Reads fits files, creates PSF, image cutout, weightmap and segmentation map
- Return statmorph morphological parameter
- Run save_sersic_results() to run statmorph on all bcgs and save the stamps

## results_table.py

- sfr_mstar_results() - provides the results from Cigale fit given BCG nomenclature
- sersic_results() - provides Sersic properties of bcgs
- main_table() - compiles all analysis results together and save master_data as csv file


## Cigale run
- example cigale parameter file : pcigale.ini
- flux catalogs in notebook/data/cigale_flux_catalog/







