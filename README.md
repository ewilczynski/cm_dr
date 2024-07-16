# Guideline for developing a calculation module for the toolbox

## Table of contents

- [Introduction](#Introduction)

- [Connect a calculation module into the toolbox](#Connect-a-calculation-module-into-the-toolbox)

- [Retrieve the calculation module fundamentals](#Retrieve-the-calculation-module-fundamentals)

- [Calculation module architecture](#Calculation-module-architecture)

- [Handling calculation module inputs](#Handling-calculation-module-inputs)

- [Handling calculation module outputs](#Handling-calculation-module-outputs)

- [Calculation module development in a local environment](#CM-development-guidelines-in-a-local-environment)

- [ Adding layers for CM ](#Adding-layers-for-CM)



## Introduction
**Requirements**

- Having GIT command installed on your computer
- Having a Python version >= 3.5
- Having a Gdal version >= 2.0

**Calculation module definition**

A calulation Module (CM) is a plugin for toolbox which is able to extend toolbox functionality.


## Connect a calculation module into the toolbox


```
Registration and hearthbeat:
_______________________________
│                    │        │      1. Once in the network, the CM continuously tries to register until it get a response from the HTAPI.
│         HTAPI      │ CM DB  │      2. Once registered the HTAPI will send requests to the CM in order to know if it's still alive or not.   
│                    │        │      3. While alive a CM can be computed from the frontend using the interface GUI it describe on its SIGNATURE.   
│____________________│________│
                        │    │
                        │    │
                        │    │    _________
                        │    └── │   CM2  │
                        │        │________│
                        │  __________
                        └─│  CM1    │
                          │________ │


```


The CM can run on its own, but when it is on the same network as the toolbox API (HTAPI), it will be automatically detected.
Using Celery queue to register, HTAPI contains heartbeat that will check at anytime if a calculation is running or not. In other words, the architecture is working in realtime and detects new CMs.



<code><ins>**[To Top](#table-of-contents)**</ins></code>




## Retrieve the calculation module fundamentals

The architecture of the repositories is illustrated below. Each CM inherits from the base calculation module (cm base; upstream):

```
GIT Repository architecture:
      ___________
      │ CM base │ upstream
      │_________│
        │
        │
        │   __________
        └── │   CM1  │ origin
        │   │________│
        │
        │
        │   __________
        └── │   CM2  │ origin
            │________│

```


In order to create a repository, follow the belowing steps.



1. Create a repository on your GitHub account and assign a name to it, e.g. **name_of_my_module**.\
Do **NOT** initialize the repository with a license nor a readme file.
Create an empty folder on your computer with the same name and go inside the folder.\

2. Use these following git commands to retrieve the code of **base_calculation_module**:

``` bash
  git init
  git remote add origin https://github.com/YourUsername/name_of_my_module.git # add a remote link to your repository
  git remote add upstream https://vlhtuleap.hevs.ch/plugins/git/git-eranet/base_calculation_module.git # add a remote link to the base calculation module (BCM)
  git pull upstream master
  git add .
  git commit -m "first commit" # update changes
  git push -u origin master # push the changes (minimum code for run)
```

3. To start coding, switch branch from master to develop:
```bash
  git checkout -b develop
```

4. After coding, push your changes on the branch develop:
```bash
  git add .
  git commit -m "message to describe the changes"
  git push origin develop
```


5. Update your code with the base calculation module (BCM) code
```bash
git pull upstream master
```

*If you encounter any issue like GIT conflict please contact EASILab (support@easilabdev.ch)*


6. Release a version of your CM


  After testing your calculation module you can update the release branch (master branch)
```bash
git fetch && git checkout master # retrieve master branch
git merge develop # update the changes from develop to master
git push origin master # push changes on master branch
```
7. And tag your version to take a snapshot of it
```bash
git tag -a number_of_the_version
```

<code><ins>**[To Top](#table-of-contents)**</ins></code>

*******************************   

## Calculation module architecture:


The architecture of a sample CM is illustrated below:



```
    cm/
    ├── app/  
    │   ├── api_v1/
    │   │    ├── __init__.py
    │   │    ├── calculation_module.py
    │   │    ├── my_calculation_module_directory   
    │   │    ├── errors.py
    │   │    └── transactions.py
    │   │
    │   │
    │   │── decorators/
    │   │   ├── __init__.py
    │   │   ├── caching.py
    │   │   ├── json.py
    │   │   ├── paginate.py
    │   │   └── rate_limit.py
    │   │
    │   │
    │   ├── __init__.py
    │   │── constant.py
    │   │── logging.conf
    │   │── utils.py
    │   │
    │   │
    │   │── config/
    │   │      ├── __init__.py
    │   │      ├── development.py
    │   │      ├── production.py
    │   │      └── transactions.py
    │   │
    │   │── tests/
    │   │      ├── __init__.py
    │   │      ├── test_client.py
    │   │      └── test.py
    │   ├── __init__.py
    │   ├── aync_consumer.py
    │   ├── Dockerfile.py
    │   ├── gunicorn-config.py
    │   ├── requirements.txt
    │   ├── run.py
    │   ├── run_cm_services.sh
    │   └── test.py
    │    
    ├── .gitiginore
    ├── docker-compose-der.yml
    ├── LICENCE
    └── README.md
```

* `app/requirements.txt` - the list of Python framework  (PyPi) requirements.

* `app/api_v1/calculation_module.py` - here you can call your CM
* `app/api_v1/my_calculation_module_directory` - all additionnal files for running the calculation module must be added in this directory


* `app/api_v1/transactions.py` - contains all the requests that enable to interact with the CM
* `app/constant.py` - contains the constants of the applications the most important constant is the <code><ins>**[SIGNATURE](#Signature-definition)**</ins></code>

<code><ins>**[To Top](#table-of-contents)**</ins></code>

***************************************************

## Handling calculation module inputs


In this section, the management of different input types accepted by the base calculation module is explained.

#### SIGNATURE definition
In order to identify each calculation module, the system need a SIGNATURE that CM provider should add.
SIGNATURE describes the parameters needed by the calculation module. This signature can be found in `constant.py`. It must be modified by the developer.
See below an example for a signature:


```
    SIGNATURE = {

        "category": "Demand",
        "cm_name": CM_NAME,
        "wiki_url": "https://wiki.hotmaps.hevs.ch/en/CM-Scale-heat-and-cool-density-maps",
        "layers_needed": [],
        "type_layer_needed": [
            {"type": "heat", "description": "You can choose the layer of type 'heat'."}
        ],
        "type_vectors_needed": [
            {"type": "industrial_database_emissions", "description": "You can choose the layer of type 'industrial_database_emissions'."}
        ],
        "cm_url": "Do not add something",
        "cm_description": "this computation module allows to divide the HDM",
        "cm_id": CM_ID,
        'inputs_calculation_module': INPUTS_CALCULATION_MODULE
    }
```

<code><ins>**[To Top](#table-of-contents)**</ins></code>

#### Signature fields

The signature contains some parameters that are needed by the main webservice HTAPI for the data exchange:

- **category**: category of the calculation module ;
- **cm_name**: name of the calculation module that will be displayed on the frontend (GUI)
- **wiki_url**: url of the page in the wiki


[here](#Retrieving-list-of-layers-available-for-CM )


When the HTAPI detects a CM, it will send a python dictionary named  **inputs_raster_selection**. This dictionary contains the name of the layer (for example *heat_tot_curr_density*) and the name of the files generated by the HATPI.

- **type_layer_needed**: 
    - type : Each layer has a type necessary to the CM. It is particularly needed in order to handle the  <code><ins>**[symbology](#symbology )**</ins></code>of the layer. the different types are :
       * *heat*,
       * *gross_floor_area*,
       * *building_volumes*,
       * *solar_optimal_total*
   - description : Description displayed in the frontend to help the user to choose the layer 

  example:
```bash
     "type_layer_needed": [
        {"type": "heat", "description": "You can choose the layer of type 'heat'."}
    ]
```

- **type_vectors_needed**: vectors needed to run the calculation module, for example:
    - type : Each layer has a type necessary to the CM. 
       * Layer name in gitlab or database name
       * *industrial_database_emissions*,
       * *wwtp_power*,
       * *agricultural_residues_view*,
   - description : Description displayed in the frontend to help the user to choose the layer 
  example:
```bash
     "type_vectors_needed": [
        {"type": "industrial_database_emissions", "description": "You can choose the layer of type 'industrial_database_emissions'."}
     ]
```

  Now if you request *inputs_vector_selection['industrial_database_emissions']*, the output will be:
```bash
    {
        "industrial_database_emissions": "/path/to/data.csv"
    }
```

  When the HTAPI detects a CM, it will send a python dictionary named  **inputs_vector_selection**. 
  This dictionary contains the path to the vector file (for example *industrial_database_emissions*). 

- **cm_description**: description of the purpose of the CM that will be displayed on the frontend GUI ;

- **cm_id**: unique identifier that is defined by the WP4 leader ;

- **authorized_scale**: this parameter will disable all scales not declared in it.

  You will find below the different scales allowed:
   - NUTS 0
   - NUTS 1
   - NUTS 2
   - NUTS 3
   - LAU 2
   - Hectare

  example:
 ```bash
   "authorized_scale":["NUTS 2","NUTS 0","Hectare"]
 ```

<code><ins>**[To Top](#table-of-contents)**</ins></code>

### Calculation module input types shown in the front-end



The purpose of this part is to give the ability to the developer to build his own user interface.
The JSON payload is an array of inputs and will be used to modify automatically the user interface. For example:

```
    INPUTS_CALCULATION_MODULE=  [
        { 'input_name': 'Reduction factor',
          'input_type': 'input',
          'input_parameter_name': 'multiplication_factor',
          'input_value': 1,
          'input_unit': 'none',
          'input_min': 1,
          'input_max': 10
            , 'cm_id': CM_ID
          }
    ]
```

<code><ins>**[To Top](#table-of-contents)**</ins></code>

#### Inputs parameters




- **input_name**: name of the CM that will be displayed on the frontend GUI

- **input_type**: the input is the graphical control element that the user needs in order to enter data. There are five possible inputs, see https://getuikit.com/docs/form for more information about the implementation of the frontend GUI.


  - input: this is a textbox in which the user can enter a value

 ![alt text][logoinput]

  - select: this is a drop down menu that allows the user to choose one value from a list

 ![alt text][logoselect]

 - radio: this allows the user to select only one of the predefined choices

 ![alt text][logoradio]

 - checkbox: this allows the user to choose between two choices mutually exclusive options

 ![alt text][logocheckbox]

 - range: this allows the user to set a value by moving an indicator

 ![alt text][logorange]


- **input_parameter_name**: input parameter name that the CM needs to retrieve for calculation

- **input_value**: default value for the input that will be displayed on the user interface

- **input_min & max**: range of the input values needed, this will prevent from mistakes in the calculation

- **input_priority**: this parameter allows to categorize input in the user inteface. The  needed value is an integer from 0 to 4:
  - 0: input always visible
  - 1 to 4: the input is on a dropdown menu


Find below two examples of inputs, one with a type *input* and the other one with the type *select*:

```python
     {  'input_name': 'Reduction factor',
        'input_type': 'input',
        'input_parameter_name': 'multiplication_factor',
        'input_value': 1,
        'input_priority': 0,
        'input_unit': 'none',
        'input_min': 1,
        'input_max': 10, 'cm_id': CM_ID  # Do no change this value
      },
      { 'input_name': 'select your input',
        'input_type': 'select',
        'input_parameter_name':'item name' ,
        'input_value': ["List item 1",
                       "List item 2",
                       "List item 3",
                       "List item 4"],
        'input_unit': 'none',
        'input_min': 'none',
        'input_max': 'none', 'cm_id': CM_ID
      }
```

<code><ins>**[To Top](#table-of-contents)**</ins></code>

*******************************   

## Handling calculation module outputs
In order to show the outputs of your CM in the front-end, your CM should respect the guidelines defined by base_calculation_module.
The purpose of this part is to give developers the ability to build different kinds of outputs (graphic, layers, indicators). All the outputs should be retunred in form of a dictionary ("result" dictionary). Find below an example:


```python
  "result": {
      "indicators": [
            {"unit": "MWh","name": "Heat demand indicator with a factor divided by 2","value": 281244.5},
            {"unit": "MWh","name": "Heat demand indicator with a factor divided by 3","value": 187496.3},
            {"unit": "MWh","name": "Heat demand indicator with a factor divided by 4","value": 140622.25}
      ],
      "name": "test_calculation_module",
      "raster_layers":[
            {"name": "heat density layer divided by 2","path": output_raster_path_tif_1,"type": "heat"},
            {"name": "heat density layer divided by 3","path": output_raster_path_tif_2,"type": "heat"},
            {"name": "gross floor area layer divided by 4","path": output_raster_path_tif_3,"type": "gross_floor_area"}
      ],
      "vector_layers":[
            {"name": "wwtp_1", "path": output_shp_zipped_1, "type": "wwtp"},
            {"name": "wwtp_2", "path": output_shp_zipped_2, "type": "wwtp"}
      ],
      "csv_files":[
            {"name": "heat_load_1", "path": output_csv_file_1},
            {"name": "heat_load_2", "path": output_csv_file_2},
            {"name": "heat_load_3", "path": output_csv_file_3}
      ],
      "graphics": [
            {"type": "bar",
             "data": {
                "labels": ["Default dataset","FR569","DE562","AT130","FR125"],
                "datasets": [{  "label": "Calculation module chart",
                                "backgroundColor": [ "#3e95cd","#8e5ea2","#3cba9f","#e8c3b9","#c45850" ],
                                "data": [2478,5267,734,784,433]   }]  }
            }
      ]
  }
```

<code><ins>**[To Top](#table-of-contents)**</ins></code>

### Indicators

In `transaction.py`, the CM provider can modify the output in order to display as many indicators as they want on the front-end. This indicator will be displayed on the RESULT panel of the front-end.


##### Structure of the indicator output

- **indicators (array):** array of indicators
    - **unit (string):** unit of the indicator
    - **name (string):** name of the indicator
    - **value (number):** value of the indicator

<code><ins>**[To Top](#table-of-contents)**</ins></code>

### Layers

#### Raster layers


**To-Do:** I suggest to add "symbology" section for the custom type in the raster and vector layers and not after them. Regarding "type", please mention which types are available (for raster and vector) and can be chosen?

##### Structure of the raster output

- **raster_layers (array):** array of raster layers
    - **name (string):** name to be displayed on the frontend
    - **path (string):** path generated for the geotif file
    - **type (string):** type of the layer generated


In order to generate a path, developers should use the function *generate_output_file_tif()*, which needs the output directory as an argument. This function should be imported to the `calculation_module.py`. The path must be generated on the first lines of *calculation()* function found in `calculation_module.py`.  This funciton For example:




```python
  output_tif_1 = generate_output_file_tif(output_directory)
```

All the layers outputs must be retrieved and added to the **raster_layers** array after they have been generated by the calculation module provider functions.


#### Vector layers
#### Symbology

**To-Do:** here, first explain what is symbology and where it is being used and what is its impact (where can the developer expect to see it)


##### Raster

There are two types of symbologies handled, both are recognized with the *type* field of the layer:
  - the predefined symbology : you do not need to create a new symbology because it has already been defined for the type you have chosen. Raster layers have four defined symbologies: *heat*, *gross_floor_area*, *building_volumes*, *solar_optimal_total* ;




  - the custom symbology: if ever the symbology of your layer does not already exist, it is possible for you to create it. You simply have to set the *type* on "custom" and add a new field *symbology*. This new field must contain values for:
    - the RGB colors (between 0 and 255 for each of the three)
    - the opacity (between 0 and 1)
    - the range of the color
    - the label for the range

Find below an example for the two symbology types:

```python
  "raster_layers":[
      {"name": "heat density layer divided by 2", "path": output_raster_1, "type": "heat"},
      {"name": "district heating coherent areas", "path": output_raster_2, "type": "custom",
               "symbology": [
                    {"red":255, "green":121, "blue":121, "opacity":0.8, "value":"100", "label":"DH Areas"}
               ]
      }
  ]
```

##### Vector

In the output of the calculation module, vector layers have the exact same management of their symbology as raster layers. Either the symbology is already defined because the type exists, or you have to define it and set the *type* on "custom". Additionally, **the name of the vector layer should include the term "shapefile"**. Here is an example:

```python
  "vector_layers":[
      {"name": "wwtp_1 - shapefile", "path": output_shp_zipped_1, "type": "wwtp"},
      {"name": "wwtp_2_custom - shapefile", "path": output_shp_zipped_2, "type": "custom",
               "symbology": [
                    {"red":255, "green":255, "blue":255, "opacity":0.8, "value":"100", "label":"250 kW"},
                    {"red":255, "green":252, "blue":188, "opacity":0.8, "value":"200", "label":"500 kW"},
                    {"red":255, "green":244, "blue": 0 , "opacity":0.8, "value":"300", "label":"750 kW"}
               ]
      }
  ]
```

The difference between vector and raster layers is that there is an extra step to add a custom symbology for vectors. In addition to the definition of this symbology in the output, you also have to add three new fields to the properties of each feature in your shapefile: *color* for the outline of your vector, *fillColor* for the filling and *opacity*. Both *color* and *fillColor* attributes must be hexadecimal values (for example *#c51b8a*), and the opacity must be a value between 0 and 1.

<code><ins>**[To Top](#table-of-contents)**</ins></code>

##### Structure of the vector output

- **vector_layers (array):** array of vector layers
  - **name (string):** name to be displayed on the frontend
  - **path (string):** path generated for the vector file
  - **type (string):** type of the layer generated


The path must be generated on the first lines of *calculation()* function found in `calculation_module.py` and uses the function *generate_output_file_shp()* which needs the output directory as an argument. For example:

```python
  output_shp_1 = generate_output_file_shp(output_directory)
```

Before sending **result** back to the HTAPI, the CM must generate a zip file, that will be handled by the HTAPI. To do so the CM provider must use the following function and write the output on the json array.

```python
  output_shp_zipped_1 = create_zip_shapefiles(output_directory, output_shp_1)
```

All the layer outputs must be retrieved and added to the **vector_layers** array after they have been generated by the calculation module provider functions and compressed with the function *create_zip_shapefiles()*.



<code><ins>**[To Top](#table-of-contents)**</ins></code>

### Graphics

The purpose of this part is to give the ability to the developer to create graphics that will be displayed on the frontend (GUI).

##### Structure of the graphics output

- **graphics (array):** array of graphics

    - **type (string) :**  type of the graphic that will be displayed. The possible types are *bar*, *line*, *radar*, *pie*, *polarArea*, *bubble*:

        - **line:** a line chart or line graph is a type of chart which displays information as a series of data points called 'markers' connected by a straight line segment:
        ![alt text][line]

        - **bar:** a bar chart or bar graph is a chart or graph that presents categorical data with rectangular bars with heights or lengths proportional to the values that they represent:
        ![alt text][bar]

        - **radar:** a radar chart is a way of showing multiple data points and the   variation between them:
        ![alt text][radar]

        - **pie:** a pie chart is divided into segments, the arc of each segment shows the proportional value of each piece of data:
        ![alt text][pie]

        - **polarArea:** polar area charts are similar to pie charts, but each segment has the same angle - the radius of the segment differs depending on the value:
        ![alt text][polarArea]

  - **xLabel:** define the text displayed on x-axis,
  - **yLabel:** define the text displayed on y-axis (ex:Heat power(MW)),
  - **data:** contains label and datasets
      - **labels (string[]) :** x axis labels  only x axis.

      - **datasets (array):** set of data with its configuration
          -  **label (string) :** serie's label
          -  **backgroundColor (string[]) :** background color of each value to display
          -  **data (number[]) :** values of the serie



<code><ins>**[To Top](#table-of-contents)**</ins></code>

*******************************************************************

### Retrieving list of layers available for CM

Please find in the link below the list of layers available as input for a CM (ressource name column):

https://docs.google.com/spreadsheets/d/1cGMRWkgIL8jxghrpjIWy6Xf_kS3Dx6LqGNfrCBLQ_GI/edit#gid=1730959780


*******************************************************************

## CM development guidelines in a local environment

##### Install requirements

Install all the libraries needed to run the CM
```bash
  cd cm
  pip install -r requirements.txt
```

##### Run the CM API

```bash
  cd cm
  python run.py
```

The documentation can be checked at http://0.0.0.0:5001/apidocs/

##### Test your CM

Before writing any line of code test the existing one, run the tests

```bash
  cd cm
  python test.py
```

##### Modify constant.py

If the test ran without any error, `constant.py` must be changed in order to
  - give a name to your CM
  - build the frontend user interface of the CM
  - assign a unique CM_ID (please contact EASILab (support@easilabdev.ch))
  - modify the SIGNATURE to describe your CM
  - etc

##### Write your code on calculation_module.py

`calculation_module.py` is a bridge between the CM functions and the CM architecture. The new code must be added to the directory **my_calculation_module_directory**


**To-Do:** I suggest to include where the unit-test should be written. Otherwise, runing the test.py returns error.

### Running my CM with docker

In the root directory:
```bash
  docker-compose up -d --build
```

## Adding layers for CM 

1. Create a repository in GitLab (To-Do: also provide the link to the GitLab) with the name of the layer in lower case
 `layer_name/data/layer_name.tif`, the repository name must be the same as the layer
2. This data must be uploaded in the server in the following place `var/hotmaps/repositories/layer_name/data/layer_name.tif`  
This is usually done by the data integration when a new layer in added to the Gitlab repository
3. This data must be known by frontend by adding the layer in the file `src/app/layers.ts`


<code><ins>**[To Top](#table-of-contents)**</ins></code>



[polarArea]: https://i0.wp.com/belajarphp.net/wp-content/uploads/2016/10/chartsJs-pola-area-chart.png?w=810&ssl=1 ""
[pie]: https://i2.wp.com/belajarphp.net/wp-content/uploads/2016/10/chartjs-pie-dognut-charts.png?ssl=1 ""
[radar]: https://i2.wp.com/belajarphp.net/wp-content/uploads/2016/10/chartJs-radar-chart.png?ssl=1 ""
[line]: https://i1.wp.com/belajarphp.net/wp-content/uploads/2016/10/chartJS-line-chart.png?ssl=1 ""
[bar]: https://i0.wp.com/belajarphp.net/wp-content/uploads/2016/10/chartJS-bar-chart-1.png?w=946&ssl=1 ""
[logoinput]: https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Textbox2.gif/220px-Textbox2.gif ""
[logocheckbox]: https://upload.wikimedia.org/wikipedia/commons/2/2f/Checkbox2.png   ""
[logoselect]: https://upload.wikimedia.org/wikipedia/commons/d/d1/Drop-down_list_example.PNG ""
[logorange]: https://upload.wikimedia.org/wikipedia/commons/e/ed/Slider_%28computing%29_example.PNG ""
[logoradio]: https://upload.wikimedia.org/wikipedia/commons/c/cb/Radio_button.png ""
