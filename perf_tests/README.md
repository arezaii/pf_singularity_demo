****************
* Parflow Solver Performance Tests
* 08/08/2019
* Ahmad Rezaii
**************** 

## OVERVIEW

 The performance test framework allows a user to run various domains against several solver configurations 
 in single or batch runs. Statistics will be gathered at the end of each domain simulation.


## INCLUDED FILES

 
 * collect_stats.tcl - script to collect stats from all the test cases in a domain
 * purge_log_files.tcl - script to purge all the log files generated from a run
 * exec_test_suite.tcl - wrapper script to run all tests in a domain from root of test directory
 * run_test.tcl - script to run a single solver configuration for a test domain
 * run_tests.tcl - wrapper script to run all tests for a domain from within domain directory
 * global_config.tcl - store location of parflow test directory to use for LW and LW_var_dz testing
 * README.md - this file
 * pfbdiff.py - python script to compare output files for differences
 * Domain SubFolders - Each problem domain is given its own folder   
   * LW - Folder containing the tests for the LW test domain
   * LW_var_dz - Folder containing the tests for the LW variable dz test domain
   * MediumWatershed - Parking lot simulation over large, irregular domain
   * UpperCO - Large'ish watershed with varied topography (Pull data from cyverse using get_cyverse_data.sh)
 * Domain SubFolders/caseXX - Each domain has a set of solver configurations to test
 * Domain SubFolders/caseXX/solver_params.tcl - Solver configuration to apply for this test_config
 * Domain SubFolders/DomainName.tcl - adapted TCL script to use when testing
 * Domain SubFolders/results.csv - the collected results from the
 * Domain SubFolders/test_config.tcl - Domain specific information, where to find the test directory, runname, etc.
 * Domain SubFolders/validate_results.tcl - Script to run any post test validations of output data
 

## RUNNING

 You must have Parflow installed and an environment variable named $PARFLOW_DIR which points to the directory where the 
 bin folder is located (standard Parflow requirements).
 
 Update the file `global_config.tcl` with the location of the parflow source test directory `~/pfdir/parflow/test` for example.
 
 Additionally, you will need the data for the Little Washita tests from the Parflow source directory. The default location
 in the Parflow source folder is tests/washita. The folder structure should be identical source.
 
 Assuming the first two conditions have been met, you can run an individual test by changing directory into the Domain SubFolder,
 then running 
 ```bash
 $ tclsh run_test caseName P Q R T 
 ```
 where caseName is the name of a folder in the Domain Subfolder, P Q R are integers which define the computation grid,
 and T is the StopTime you want the simulation to use. 
 
 To run all the test cases for a Domain, change directory to this folder (test directory) and run 
 ```bash
 $ tclsh exec_test_suite.tcl domainName P Q R T
 ```
 where domainName is the name of a Domain SubFolder and P Q R are integers which define the computation grid,
 and T is the StopTime you want the simulation to use.
 
  

## TO ADD A NEW DOMAIN
 
 Each new domain should be added as a Domain SubFolder to the main test directory. 
 Test cases are defined as folders in the Domain SubFolder and require a solver_params.tcl file.
 A domain run script (DomainName.tcl) should be edited to use supplied parameters for P Q R T.
 Additional modifications to DomainName.tcl required:
 * Comment out existing settings for solver configurations set by solver_params.tcl
 * Create a caseDefault test folder and place the original solver configurations in the solver_params.tcl there.
 * Copy any other test folders from existing test domain into new domain
 * Move the validation portion of the script to the validate_results.tcl script.
 * Insert the following code block to set the solver configurations at runtime:
 ```tcl
 ###Test Settings
pfset Solver.Nonlinear.UseJacobian                       $UseJacobian 
pfset Solver.Nonlinear.EtaValue                          $EtaValue
pfset Solver.Linear.Preconditioner                       $Preconditioner

if {[info exists PCMatrixType]} {
	pfset Solver.Linear.Preconditioner.PCMatrixType          $PCMatrixType
}

if {[info exists MaxIter]} { 
  pfset Solver.Linear.Preconditioner.$Preconditioner.MaxIter         $MaxIter
}

if {[info exists MaxLevels]} { 
  pfset Solver.Linear.Preconditioner.$Preconditioner.MaxLevels         $MaxLevels
}

if {[info exists Smoother]} { 
  pfset Solver.Linear.Preconditioner.$Preconditioner.Smoother         $Smoother
}

if {[info exists RAPType]} {
  pfset Solver.Linear.Preconditioner.$Preconditioner.RAPType          $RAPType
}
```
 

 