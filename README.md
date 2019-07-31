# The Multiple Flying Sidekicks Traveling Salesman Problem (mFSTSP)

This repository provides a collection of mFSTSP test problems, as well as the source code to solve mFSTSP instances. The mFSTSP is a variant of the classical TSP, in which one or more UAVs coordinate with a truck to deliver parcels in the minimum possible time. The code provided here consists of both the mixed integer linear programming (MILP) implementation and a heuristic to solve larger problems. 

The repository accompanies the following paper, which is currently undergoing a second round of reviews:
> C. Murray and R. Raj, The Multiple Flying Sidekicks Traveling Salesman Problem: Parcel Delivery with Multiple Drones (July 27, 2019). Available at SSRN: https://ssrn.com/abstract=3338436 or http://dx.doi.org/10.2139/ssrn.3338436

The paper provides details on the mFSTSP definition, a corresponding MILP formulation, and the heuristic.


## mFSTSP Repository Contents
This repository contains **test problems**, **solutions**, and the **source code required to run the mFSTSP solver**.  These elements are described in detail below.  

1. A collection of **test problems (and solutions)** that were used in the analysis described in the [mFSTSP paper](https://ssrn.com/abstract=3338436).

   1. [`InstanceInfo.csv`](InstanceInfo.csv) contains a summary of all 100 "base" test problems, including the number of customers, geographic region (Buffalo, NY or Seattle, WA), and the number of customers within 5 or 10 miles of the depot.  This file may be useful if you are looking for problems with certain properties (e.g., 50-customer problems, or problems in Seattle). 
   
   2. [`performance_summary_archive.csv`](performance_summary_archive.csv) provides information about solutions generated for each test problem in the [mFSTSP paper](https://ssrn.com/abstract=3338436). See the ["Archived Problem Solutions"](#Archived-Problem-Solutions) section below for details on the contents of this file.
 
      - **NOTE:** [`performance_summary.csv`](performance_summary.csv) is an empty/placeholder file.  It will be populated only if/when you run the solver code.  This file currently exists solely to initialize the column headings.
       
   3. The [`Problems`](Problems) directory contains 100 sub-directories (one for each "base" problem).  Each sub-directory contains the following groups of files:
      - There are 2 files that are required by the solver.  These 2 files must be present in order to run the solver, as they are **inputs** to either the MILP model or the heuristic:
         - `tbl_locations.csv` describes all nodes in the problem.  `nodeType 0` represents the depot, `nodeType 1` represents a customer.  The depot is always always assigned to `nodeID 0`.  Each node has a corresponding latitude and longitude (specified in degrees).  The altitude is always 0.  Customer nodes have a corresponding non-zero parcel weight (in [pounds]).  There is no parcel associated with the depot. 
         - `tbl_truck_travel_data_PG.csv` contains the directed truck travel time and distance information from one node to another.  All time and distance values were obtained by pgRouting, using OpenStreetMaps data.
      - There is one file that was created by our "problem generator":
         - `map.html` is a webpage showing the locations of all nodes (one depot and multiple customers) associated with this problem.  This is for informational purposes only; it is neither an input to, nor an output from, the solvers.
      - Finally, there are multiple files that were generated by the solvers (MILP, heuristic, or both) when we were preparing the [mFSTSP paper](https://ssrn.com/abstract=3338436).  These are **outputs** from the solver (and are, therefore, not required *before* solving a problem).  
         - Numerous files of the form `tbl_solutions_<UAVtype>_<# of UAVs>_<solutionMethod>.csv`, where 
            - `<UAVtype>` indicates the speed and range of the UAVs.  This will be `101`, `102`, `103`, or `104`.  See below for more information.
            - `<# of UAVs>` indicates the number of available UAVs.  The [mFSTSP paper](https://ssrn.com/abstract=3338436) considered 1-4 UAVs.
            - `<solutionMethod>` will be either `IP` (if Gurobi was used to solve the MILP) or `Heuristic` (if the heuristic was employed).
            
            Each of these files contains details about solutions corresponding to a specified combination of `<UAVtype>`, `<# of UAVs>`, and `<solutionMethod>`.
         
   4. The `Problems` directory also contains four CSV files, of the form `tbl_vehicles_<ID>.csv` that provide information on UAV specifications, where:
      - ID [`101`](Problems/tbl_vehicles_101.csv): High speed, low range;
      - ID [`102`](Problems/tbl_vehicles_102.csv): High speed, high range;
      - ID [`103`](Problems/tbl_vehicles_103.csv): Low speed, low range; and
      - ID [`104`](Problems/tbl_vehicles_104.csv): Low speed, high range.
      
      Each test problem can be solved with any of these UAV specifications by specifying the appropriate command-line argument.  This is described in the "[Running the Solvers](#Running-the-Solvers)" section below. 

2. Although there are numerous Python scripts in this repository, [`main.py`](main.py) is the only one you will directly interact with.  This script makes use of the other Python scripts.  See "[Running the Solvers](#Running-the-Solvers)" below for instructions on generating solutions.

----

## Installation and Setup

This section provides instructions for running the two mFSTSP solvers (either an exact MILP or a heuristic).

### Compatability

The mFSTSP source code is compatible with both **Python 2** and **Python 3**.  
It has been tested on **Windows**, **Linux**, and **Mac**.

### Prerequisites
- Both the exact and hueristic methods require [Gurobi](http://gurobi.com) as the MILP solver.
- The `pandas` Python package is also required.

- For **Linux/Mac**, issue the following terminal command to install `pandas`:
   ```
   pip install pandas
   ```
   *If you receive errors related to "access denied", try running `sudo pip install pandas`.*

- For **Windows**, there are two options:
   1. If you installed Python through [Anaconda](https://www.anaconda.com/), `pandas` is already included.
   2. Otherwise, enter the following at a command prompt:
      ```
      py -m pip install pandas
      ```

### Download Problems and Source Code

There are two options for importing this repository's contents to your computer.
1. Download a .zip archive of the repository by either:
   - [Clicking this link](https://github.com/optimatorlab/mFSTSP/archive/master.zip), or
   - Clicking the green "Clone or download" button above and choosing "Download ZIP". 
   
   In either case, unzip the archive on your computer to the location of your choice.

2. Use `git` from the command-line.
   1. Change directories to your preferred download location, replacing `<the directory of your choice>` with a path to that location:
   ```
   cd <the directory of your choice>   
   ```
   2. Clone the repository.  This will create a directory named `mFSTSP`.  Within that directory will be the entire contents of this repository.
   ```
   git clone https://github.com/optimatorlab/mFSTSP.git
   ```
  
### Running the Solvers



1. Open a terminal (command prompt) window.

2. Change directories to the location where the `mFSTSP` directory is saved.
   - For **Linux/Mac**:  We will assume that you have downloaded the repository to a directory named `mFSTSP` within your `Downloads` directory.  If you saved the `mFSTSP` directory elsewhere, you'll need to modify the paths below accordingly.
      ```
      cd ~/Downloads/mFSTSP
      ```
   
      *(Your path will differ if you saved the repository contents to a different directory.)*
      
   - For **Windows**:  We will assume that you have downloaded the repository to a folder named `mFSTSP` on the "D:\" drive.  If you saved the `mFSTSP` directory elsewhere, you'll need to modify the paths below accordingly.
      ```
      D:
      cd Projects\mFSTSP
      ```

      *(Your path will differ if you saved the repository contents to a different directory.)*
   
3. The mFSTSP solvers are invoked by running the [`main.py`](main.py) Python script with 10 arguments.  This will have the following structure:
   ```
   python main.py <problemName> <vehicleFileID> <cutoffTime> <problemType> <numUAVs> <numTrucks> <requireTruckAtDepot> <requireDriver> <Etype> <ITER>
   ```

   The command-line arguments, which must be specified in this order, are:
   - `problemName` - The name of the directory containing the data for a particular problem instance.  This directory is in the format of a timestamp, and must exist within the [`Problems`](Problems) directory.
   - `vehicleFileID` - Indicates the speed and range of the UAVs.  See the [note above](#mFSTSP-Repository-Contents), which describes the definitions of `101`, `102`, `103`, and `104`.
   - `cutoffTime` - The maximum allowable runtime, in [seconds].  If the problem is to be solved via heuristic, this value represents the maximum runtime of Phase 3.  A cutoff time of `-1` indicates that the solver should ignore any runtime limits.
   - `problemType` - Indicates if the problem is to be solved via the exact MILP formulation (`1`) or via the heuristic (`2`).
   - `numUAVs` - Number of UAVs available in the problem (e.g., `3`).
   - `numTrucks` - Number of trucks available in the problem.  The code currently only supports `1` truck. Assigning `-1` to this argument ignores its value and considers 1 truck.
   - `requireTruckAtDepot` -  Indicates whether the truck is required to be at the depot when UAVs are launched from the depot:  `0` (false) or `1` (true).  This problem "variant" is described in Section 3 of the [mFSTSP paper](https://ssrn.com/abstract=3338436).
   - `requireDriver` - Indicates if UAVs can launch from and return to the truck without the driver (thus allowing the driver to serve a customer while UAVs are launching/landing): `0` (false) or `1` (true).  This problem "variant" is described in Section 3 of the [mFSTSP paper](https://ssrn.com/abstract=3338436).
   - `Etype` - Indicates the endurance model that was employed.  Options include: `1` (nonlinear), `2` (linear), `3` (fixed/constant time), `4` (unlimited), and `5` (fixed/constant distance).  Details on these models are found in Section 4 of the [mFSTSP paper](https://ssrn.com/abstract=3338436).
   - `ITER` - Indicates the number of iterations to be run for each value of "LTL".  If `problemType == 1` (MILP), `ITER` is ignored (and can be assigned a value of `-1`).  Otherwise, `ITER` can be any integer `1` or greater for the heuristic.  NOTE: In the paper, `ITER` is assumed to be `1`.
   
   **Example 1 -- Solving the mFSTSP via the MILP formulation:**
   
   ```
   python main.py 20170608T121632668184 101 3600 1 3 -1 1 1 1 -1
   ```

   **Example 2 -- Solving the mFSTSP via a heuristic:**

   ```
   python main.py 20170608T121632668184 101 5 2 3 -1 1 1 1 1
   ```
   
4. The solver is finished when you receive a message like this in the terminal:
   ```
   See '[performance_summary.csv](blah)' for statistics.

   See 'Problems/20170608T121632668184/tbl_solutions_101_3_Heuristic.csv' for solution summary.
   ```

   - The solver appends a row to the end of [`performance_summary.csv`](performance_summary.csv).  This row contains the objective function value, total run time, number of customers assigned to the truck, number of customers assigned to UAVs, etc.

   - The solver also generates a file of the form `tbl_solutions_<UAVtype>_<# of UAVs>_<solutionMethod>.csv`.  This file will appear within the subdirectory corresponding to the `problemName` within the [`Problems`](Problems) directory (e.g.,  [`Problems/20170608T121632668184`](Problems/20170608T121632668184)) in the above example.  The solutions file contains the objective function value and a detailed schedule for the truck and UAV(s).
      - **NOTE**: If you re-run the solver, solution details will be appended to the bottom of the applicable `tbl_solutions_<UAVtype>_<# of UAVs>_<solutionMethod>.csv` file.

## Archived Problem Solutions

This repository contains solutions to the problem instances, as discussed in the analysis section of the [mFSTSP paper](https://ssrn.com/abstract=3338436). 

[`performance_summary_archive.csv`](performance_summary_archive.csv) provides summary information about these solutions. 
This file contains the following columns:
- `problemName` - The name of the problem instance (written in the form of a timestamp).  This is a subdirectory name within the [`Problems`](Problems) directory.
- `vehicleFileID` - Indicates the speed and range of the UAVs.  See the note below, which describes the definitions of `101`, `102`, `103`, and `104`.
- `cutoffTime` - The maximum allowable runtime, in [seconds].  If the problem was solved via heuristic, this value represents the maximum runtime of Phase 3.
- `problemType` - Indicates if the problem was solved via MILP (`1`) or heuristic (`2`).
- `problemTypeString` - A text string describing the solution approach (`mFSTSP IP` or `mFSTSP Heuristic`).
- `numUAVs` - # of UAVs available (`1`, `2`, `3`, or `4`).
- `numTrucks` - This can be ignored; it will always have a value of `-1`.  However, in each problem there is actually exactly 1 truck.
- `requireTruckAtDepot` - A boolean value to indicate whether the truck is required to be at the depot when UAVs are launched from the depot.  This problem "variant" is described in Section 3 of the [mFSTSP paper](https://ssrn.com/abstract=3338436).
- `requireDriver` - A boolean value to indicate if the driver is required to be present when UAVs are launched/retrieved by the truck at customer locations.  This problem "variant" is described in Section 3 of the [mFSTSP paper](https://ssrn.com/abstract=3338436).
- `Etype` - Indicates the endurance model that was employed.  Options include: `1` (nonlinear), `2` (linear), `3` (fixed/constant time), `4` (unlimited), and `5` (fixed/constant distance).  Details on these models are found in Section 4 of the [mFSTSP paper](https://ssrn.com/abstract=3338436). 
- `ITER` - Indicates the number of iterations to be run for each value of "LTL".  If `problemType == 1` (MILP), `ITER` will be -1 (not applicable).  Otherwise, `ITER` will be `1` for the heuristic.  NOTE: This feature is not described/implemented in the [mFSTSP paper](https://ssrn.com/abstract=3338436).
- `runString` - This contains the command-line arguments used to solve the problem.  It is included here to allow copy/pasting.
- `numCustomers` - The total number of customers.
- `timestamp` - The time at which the problem was solved.
- `ofv` - The objective function value, as obtained by the given solution approach.
- `bestBound` - If `problemType == 1` (MILP), this is the best bound provided by Gurobi.  Otherwise, `bestBound = -1` for the heuristic (as no bounds are available).
- `totalTime` - The total runtime of the MILP or heuristic.
- `isOptimal` - A boolean value to indicate if the solution was provably optimal.  This only applies if `problemType == 1` (MILP); there is no proof of optimality for the heuristic.
- `numUAVcust` - The number of customers assigned to the UAV(s).
- `numTruckCust` - The number of customers assigned to the truck.
- `waitingTruck` - The amount of time, in [seconds], that the truck spends waiting on UAVs.
- `waitingUAV` - The amount of time, in [seconds], that the UAVs spend waiting on the truck.

[`performance_summary_archive.csv`](performance_summary_archive.csv) contains records for the following classes of problems:
1. Each 8-customer problem (there are 20 total) was run using four different settings of `vehicleFileID` (101, 102, 103, and 104) and four different settings of `numUAVs` (1, 2, 3, and 4).  Each problem was solved using both the full MILP formulation (as an "exact" method) and the heuristic described in the paper.  This resulted in a total of 640 problem solutions (20 * 4 * 4 * 2).  
    
2. Each of the 10-, 25-, 50-, and 100-customer problems (80 total) was run using four different settings of `vehicleFileID` and `numUAVs`.  These problems were solved only via the heuristic.  There are a total of 1280 solutions to these problems (80 * 4 * 4 * 1).
    
Details of each solution may be found in the applicable `tbl_solutions_<UAVtype>_<# of UAVs>_<solutionMethod>.csv` file contained within the `problemName` subdirectory of the [`Problems`](Problems) directory (e.g., [`Problems/20170608T121632668184`](Problems/20170608T121632668184)).  

When a problem is run again, using the same settings (e.g., the same `vehicleFileID`, `numUAVs` and `problemType`), the solution details are appended to the applicable `tbl_solutions_<UAVtype>_<# of UAVs>_<solutionMethod>.csv` file.


## Contact Info

For any queries, please send an email to r28@buffalo.edu.
