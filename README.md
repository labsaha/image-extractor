# ImageExtractor

[![build status](https://travis-ci.org/cta-observatory/image-extractor.svg?branch=master)](https://travis-ci.org/cta-observatory/image-extractor.svg?branch=master)[![Coverage Status](https://coveralls.io/repos/github/cta-observatory/image-extractor/badge.svg?branch=master)](https://coveralls.io/github/cta-observatory/image-extractor?branch=master) [![Code Health](https://landscape.io/github/cta-observatory/image-extractor/master/landscape.svg?style=flat)](https://landscape.io/github/cta-observatory/image-extractor/master)


Package for loading simtel data files, processing and calibrating the event data, and writing the processed data to a custom PyTables HDF5 format. Created for the testing of new machine learning and other analysis techniques for the
[Cherenkov Telescope Array (CTA)](https://www.cta-observatory.org/ "CTA collaboration Homepage") collaboration. Built using Pytables and ctapipe.

Currently under development, intended for internal use only.

## Installation in Anaconda (Recommended)

The following installation method (for Linux) is recommended:

### Installing Anaconda

First, install the Anaconda Python distribution (Python 3.6), which can be found [here](https://www.anaconda.com/download/).

Clone the repository (or fork and clone if you wish to develop) into a local directory of your choice.

```bash
cd software
git clone https://github.com/cta-observatory/image-extractor.git
cd image-extractor
```

Create a new Anaconda environment (defaults to Python 3.6), installing all of the dependencies using requirements.txt.

```bash
conda create -n [ENV_NAME] --file requirements.txt -c cta-observatory -c conda-forge -c openastronomy
```

### Installing ctapipe from source

NOTE: As of v0.5.1, the conda package version of ctapipe appears to be lagging somewhat behind the development version and is missing key features which are used by ImageExtractor. As a result, it is necessary to install ctapipe from source. To do this:

Clone the ctapipe repository (or fork and clone if you wish to develop) into a local directory of your choice.

```bash
cd software
git clone https://github.com/cta-observatory/ctapipe.git
cd ctapipe
```

Activate the Anaconda environment you set up earlier:

```bash
source activate [ENV_NAME]
```

Then to install, simply run:

```bash
python setup.py install
```

You can verify that ctapipe was installed correctly by running:

```bash
conda list
```

and looking for ctapipe.

Do the same for ctapipe-extra (a repository containing additional resources for ctapipe):

```bash
cd software
git clone https://github.com/cta-observatory/ctapipe-extra.git
cd ctapipe-extra
source activate [ENV_NAME]
python setup.py install
```

If both packages are showing up correctly, you should now be able to run/import image\_extractor.py or any of the other scripts/modules from the command line in your environment.

### Installing ImageExtractor using pip

Finally, you can install ImageExtractor using pip.

```bash
source activate [ENV_NAME]
conda install pip

cd image-extractor
/path/to/anaconda/install/envs/[ENV_NAME]/bin/pip install .
```
where /path/to/anaconda/install is the path to your anaconda installation directory and ENV\_NAME is the name of your environment.

The path to the environment directory for the environment you wish to install into can be found quickly by running

```bash
conda env list
```

## Installation in Python (Not recommended):

The same installation procedure can be done in a non-Anaconda version of Python, however this is not recommended due to the small chance of dependency issues or other bugs. Alternatively, the installation can be done in a virtualenv environment.

## Dependencies

See requirements.txt for the full list of dependencies.

The main dependencies are:

for image_extractor:

* PyTables
* NumPy
* ctapipe

for additional scripts in scripts directory:

* Pillow
* ROOT
* matplotlib

## Usage

### From the Command Line:

To create a HDF5 dataset file out of a collection of .simtel.gz files, run:

```bash
image_extractor.py [runlist] [output_file] [--ED_cuts_dict_file ED_CUTS_DICT_FILE] [--max_events MAX_EVENTS] [--shuffle [SEED]] [--split [SPLIT_LIST]] [--debug]
```
on the command line.

ex:

```bash
image_extractor.py runlist.txt ./dataset.h5 --ED_cuts_dict_file ./bins_cuts_dict.pkl --debug
```

* runlist - A list of filepaths (relative or absolute) for the simtel.gz files (one per line) to process. Lines beginning with a '#' are ignored and can be used for comments. 
* output_file - The path to the HDF5 file into which you wish to write your data.
* ED_cuts_dict_file - Optional path to a pickled Python dictionary of a specified format (see prepare_cuts_dict.py) containing information from EventDisplay on which events (run_number,event_number) passed the cuts, what their reconstructed energies are, and which energy bin they are in.
* max_events - Optional argument to specify the maximum number of events to save from the simtel file
* shuffle [SEED]- Optional flag to randomly shuffle the data in the Events table after writing. Can provide an optional seed value to get a reproduceable result.
* split [SPLIT_LIST]- Optional flag to split the Events table into separate training/validation/test tables for convenience. Can provide a list (3 arguments) which give the split proportions between train, val, and test. A split proportion of 0 indicates that the corresponding table will not be created. Split proportions must sum to 1.
* debug - Optional flag to print additional debug information.

NOTE: The shuffle and split flags as well as other non-default ImageExtractor constructor options are not currently being maintained as carefully as they are not part of the standard CTA ML data format. Use at your own risk!

NOTE: The split option is currently set to use a hardcoded default split setting (0.9 training, 0.1 validation). This can be modified in the script if desired.

NOTE: The splitting and shuffling handled by the split and shuffle options are NOT done in-place, so they require a temporary disk space overhead beyond the normal final size of the output .h5 file. Because only the Events tables are copied/duplicated and the majority of the final file size is in the telescope arrays, this overhead is likely insignificant for the 'mapped' storage format. However, it is much larger for the 'all' storage format, as the telescope images are stored directly in the Event tables and are therefore duplicated temporarily. Also, it is worth noting that this overhead becomes larger in absolute terms as the absolute size of the output files becomes larger.

### In a Python script:

If the package was installed with pip as described above, you can import and use it in Python like:

ex:

```python
from image_extractor import image_extractor

#max events to read per file
max_events = 50

data_file = "/home/computer/user/data/simtel/test.simtel.gz"

#create an ImageExtractor with default settings
extractor = image_extractor.ImageExtractor(output_path,tel_type_list=['MSTS'])

extractor.process_data(data_file,max_events)

```

### ED_cuts_dict files

ED_cuts_dict.pkl files are the result of running the EventDisplay analysis on the simtel files through to the MSCW stage, then applying the cuts specified in the config file on the array/event-level parameters. The default cuts were selected to match those used by the Boosted Decision Tree analysis method for Gamma-Hadron separation currently being used in EventDisplay for VERITAS data. A new one can be created (for example, to match a new set of simtel files) by running the standard ED analysis on all simtel files up to the MSCW stage, then passing the MSCW.root files into prepare_bins_dict.py.

## Examples/Tips

* Vitables is very helpful for viewing and debugging PyTables-style HDF5 files. Installation/download instructions can be found in the link below. NOTE: It is STRONGLY recommended that vitables be installed in a separate Anaconda environment with Python version 2.7 to avoid issues with conflicting PyQt5 versions. See this issue thread for details: [https://github.com/ContinuumIO/anaconda-issues/issues/1554](https://github.com/ContinuumIO/anaconda-issues/issues/1554)

## Known Issues/Troubleshooting

* ViTables PyQt5 dependency confict (pip vs. conda): [relevent issue thread](https://github.com/ContinuumIO/anaconda-issues/issues/1554)
* Conda version of ctapipe does not support most recent features. Use dev version instead: [installation instructions](https://cta-observatory.github.io/ctapipe/getting_started/index.html#get-the-ctapipe-software)

## Links

* [Cherenkov Telescope Array (CTA)](https://www.cta-observatory.org/ "CTA collaboration Homepage") - Homepage of the CTA collaboration
* [Deep Learning for CTA Analysis](https://github.com/bryankim96/deep-learning-CTA "Deep Learning for CTA Repository") - Repository of code for studies on applying deep learning to CTA analysis tasks. Maintained by groups at Columbia University and Barnard College.
* [ctapipe](https://cta-observatory.github.io/ctapipe/ "ctapipe Official Documentation Page") - Official documentation for the ctapipe analysis package (in development)
* [ViTables](http://vitables.org/ "ViTables Homepage") - Homepage for ViTables application for Pytables HDF5 file visualization





 
