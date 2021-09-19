<p align="center">
    <img src="images/example_image.png" width="200" title="Image sample" />
    <img src="images/actin.png" width="200" title="Actin" />
    <img src="images/tubulin.png" width="200" title="Tubulin" />
    <img src="images/DAPI.png" width="200" title="DAPI" />
</p>

# pybbbc
This is a python interface to the BBBC021 dataset of cellular images ([Caie et al., Molecular Cancer Therapeutics, 2010](http://dx.doi.org/10.1158/1535-7163.MCT-09-1148)), available from the Broad Bioimage Benchmark Collection ([Ljosa et al., Nature Methods, 2012](http://dx.doi.org/10.1038/nmeth.2083)).

The dataset is made of 13,200 fields of view imaged in three channels of resolution 1024x1280 and the corresponding metadata: site, well, replicate, plate, compound, concentration and mechanism of action (MoA).

The images are of MCF-7 breast cancer cells treated for 24 h with a collection of 113 small molecules at eight concentrations. The cells were fixed, labeled for DNA, F-actin, and Β-tubulin, and imaged by fluorescent microscopy. The complete description of the dataset can be found [here](https://bbbc.broadinstitute.org/BBBC021).

## Installation

### With pip

```bash
pip install git+https://github.com/giacomodeodato/pybbbc.git
```

### For development (with conda)

```bash
git clone git+https://github.com/giacomodeodato/pybbbc.git
cd pybbbc
conda env update
conda activate pybbbc
pre-commit install
```

## Usage
An instance of the dataset can be easily created with the following code:
```python
from pybbbc import BBBC021

bbbc021 = BBBC021()
```
### Indexing
Standard indexing (i.e. ```bbbc021[3]``` or ```bbbc021[2:5]```) can be used to access samples and subsets. A single sample is a tuple ```(image, metadata)``` where the metadata is a tuple itself of plate metadata and compound metadata using the following structure:
```python
metadata = (
    ( # plate metadata
        site,
        well,
        replicate,
        plate
    ),
    ( # compound metadata
        compound,
        concentration,
        moa
    ),
    image_idx
)
```
where `image_idx` is the absolute index of the image in the BBBC021 dataset
without filtering applied.

### Filtering
The instance of a dataset can be intuitively filtered during the initiation using metadata keyword arguments as follows:
```python
# get the samples without known MoA
bbbc021_null = BBBC021(moa='null')
```
```python
# get only samples with known MoAs
bbbc021_moa = BBBC021(moa=[moa for moa in BBBC021.MOA if moa != "null"])
```
or using a list of metadata values:
```python
# get the samples with the colchicine and latrunculin B compounds
bbbc021 = BBBC021(compound=['colchicine', 'latrunculin B'])

# get the samples with only a selection of Mechanisms of Action
bbbc021 = BBBC021(moa=['Actin disruptors', 'DMSO', 'Microtubule destabilizers'])
```
The dataset can be filtered using any metadata keyword: ```site```, ```well```, ```replicate```, ```plate```, ```compound```, ```concentration``` and ```moa```. The unique values of the most relevant keywords are stored as static attributes: ```BBBC021.PLATES```, ```BBBC021.COMPOUNDS``` and ```BBBC021.MOA```. Other useful static attributes are ```BBBC021.IMG_SHAPE```, ```BBBC021.CHANNELS``` and ```BBBC021.N_SITES```.

### Sub-datasets
The individual sub-datasets and their corresponding samples can be accessed as well after initiation:
```python
bbbc021 = BBBC021()

img = bbbc021.images[0]

# bbbc021.wells
# bbbc021.sites
# bbbc021.replicates
# bbbc021.plates
# bbbc021.compounds
# bbbc021.concentrations
# bbbc021.moa
```

### View the metadata `DataFrame`s

The metadata is compiled into two Pandas `DataFrame`s, `image_df` and `moa_df`,
which contain only metadata from the selected subset of the BBBC021 dataset.

`image_df` contains metadata information on an individual image level.
Each row corresponds to an image in the subset of BBBC021 you selected:

```python
> bbbc021_moa.image_df

      site well  replicate        plate compound  concentration  \
0        1  B05          1  Week5_28901     AZ-J       1.000000
1        1  B04          1  Week5_28901     AZ-J       3.000000
2        1  B03          1  Week5_28901     AZ-J      10.000000
3        1  B11          1  Week5_28901    taxol       0.300049
4        1  C11          1  Week5_28901    taxol       0.300049
...    ...  ...        ...          ...      ...            ...
3843     4  C02          1  Week4_27481     DMSO       0.000000
3844     4  D02          1  Week4_27481     DMSO       0.000000
3845     4  E11          1  Week4_27481     DMSO       0.000000
3846     4  F11          1  Week4_27481     DMSO       0.000000
3847     4  G11          1  Week4_27481     DMSO       0.000000

                          moa  image_idx  relative_image_idx
0                  Epithelial         45                   0
1                  Epithelial         46                   1
2                  Epithelial         47                   2
3     Microtubule stabilizers         48                   3
4     Microtubule stabilizers         49                   4
...                       ...        ...                 ...
3843                     DMSO      13195                3843
3844                     DMSO      13196                3844
3845                     DMSO      13197                3845
3846                     DMSO      13198                3846
3847                     DMSO      13199                3847

[3848 rows x 9 columns]
```

`image_idx` corresponds to the absolute index of the image in the full BBBC021 dataset.
`relative_image_idx` is the index you would use to access the given image as in:

`image, metadata = your_bbbc021_obj[relative_image_idx]`

`moa_df` is a metadata `DataFrame` which provides you with all the compound-concentration pairs in the selected BBBC021 subset:

```python
> bbbc021_moa.moa_df

        compound  concentration                        moa
0           ALLN       3.000000        Protein degradation
1           ALLN     100.000000        Protein degradation
2           AZ-A       0.099976   Aurora kinase inhibitors
3           AZ-A       0.300049   Aurora kinase inhibitors
4           AZ-A       1.000000   Aurora kinase inhibitors
..           ...            ...                        ...
99   vincristine       0.029999  Microtubule destabilizers
100  vincristine       0.099976  Microtubule destabilizers
101  vincristine       0.300049  Microtubule destabilizers
102  vincristine       1.000000  Microtubule destabilizers
103  vincristine       3.000000  Microtubule destabilizers

[104 rows x 3 columns]
```

## Data download
The raw data can be downloaded after importing the BBBC021 dataset as follows:
```python
from pybbbc import BBBC021

BBBC021.download()
```
This will automatically create the data folder in `~/.cache/` (by default) and download images and metadata (this may take several hours). Alternatively, it is possible to specify the ```data_path``` parameter to set the destination folder.

## Dataset creation
After downloading the raw data, the preprocessed dataset can be created by calling the static method ```make_dataset()```.
```python
from pybbbc import BBBC021

# BBBC021.download()
BBBC021.make_dataset()
```
This function will execute the following steps:
* Merge the metadata regarding the compound and the Mechanism of Action (MoA)
* Compute and apply the illumination correction function to the images
* Scale pixel intensities in the \[0, 1\] range discarding the top and bottom 0.1 percentiles
* Create a virtual HDFS dataset (using h5py) associating the images to the corresponding metadata

### Illumination correction
In order to compute the illumination correction function, the average of all the images taken at the same site and belonging to the same plate is calculated. This image is then smoothed using a Gaussian filter with sigma=500. A robust minimum is calculated as the 0.02 percentile of all the positive values of the average image, and all the pixel with smaller values are truncated. This image is finally divided by the robust minimum in order to have only values greater or equal to one so that the illumination correction can be applied by dividing the original image by the illumination function.

```python
def correct_illumination(images, sigma=500, min_percentile=0.02):

    # calculate average of images belonging to the same site and plate
    img_avg = images.mean(axis=0)

    # apply Gaussian filter
    img_mask = gaussian_filter(img_avg, sigma=sigma)

    # calculate robust minimum
    robust_min = np.percentile(img_mask[img_mask > 0], min_percentile)

    # clip and scale pixel intensities
    img_mask[img_mask < robust_min] = robust_min
    img_mask = img_mask / robust_min

    # return corrected images
    return images / img_mask
```
