# Node.js Benchmarking 

Instructions and tools for setting up and running Node.js benchmarks - core benchmarks and AcmeAir.

## Running core benchmarks

### What is needed

* A local clone of [node](https://github.com/nodejs/node) and/or [node-chakracore](https://github.com/nodejs/node-chakracore) repo (depending on which one you want to test)
* The pre-requisites necessary to build Node. See https://github.com/nodejs/node/blob/master/BUILDING.md
* A local clone of this repo.

Build node. Add the directory where you cloned this repo to your path.
Then, from the root of the Node repo, run `run_benchmarks.bat` or `run_benchmarks.sh` （for linux you may need to change authority to run benchmarks.sh,run `sudo chmod 777 run_benchmarks.sh`）, redirecting output to file. These scripts run the subset of benchmarks that work properly on Windows (some benchmarks - e.g. http - haven't been ported). Run the script a couple of times, redirecting to different files each time, adding `1`, `2`, etc. to the end of filename. For example:

```
run_benchmarks >Win1
run_benchmarks >Win2
run_benchmarks >Win3
```

## Processing core benchmarks results
Compile `LogProcessor/BenchmarkProcess/benchmark.cpp` (Visual Studio solution is provided). This tool converts core benchmarks output to tab-separated files. Usage:

`benchmark directory_with_bechmark_results first_group_name [nth_group_name...]`

`group_name_1` will be used as filename base. This tool will search for `first_group_name1`, `first_group_name2`, etc. files in `directory_with_bechmark_results`. The average results for all those files will be calculated. You can provide more than one group. For each successive group, the percentage change in performance will also be computed.

## Running acme-air bechmarks

### What is needed
* [nodejs/benchmarking](https://github.com/nodejs/benchmarking)
* [acmeair-nodejs](https://github.com/acmeair/acmeair-nodejs)
* [acmeair-driver](https://github.com/acmeair/acmeair-driver)

You will also have to install:
* [MongoDB](https://www.mongodb.org/)
* [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* [JMeter](http://jmeter.apache.org/download_jmeter.cgi)
* [json-simple](https://code.google.com/archive/p/json-simple/)

If the objective is to compare benchmark results across different platforms, installing MongoDB on a separate machine is recommended.
You can just run these command to install softeware above:
```
 sudo apt-get install mongodb
 sudo apt-get install openjdk-7-jdk
 curl -# -O http://mirror.nus.edu.sg/apache//jmeter/binaries/apache-jmeter-3.0.tgz
 tar -xf apache-jmeter-3.0.tgz
 curl -# -O https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/json-simple/json-simple-1.1.1.jar
```
Remember to add jmeter directory to $PATH. 
```
		export JMETER_DIR={your jmeter directory}
		export PATH=$PATH:$JMETER_DIR/bin
```
This only works for current shell.

### Setting up JMeter
To set up JMeter:
* Make sure you have the Java JDK installed
* Build `acmeair-driver` by calling `./gradlew build` or `gradlew.bat build` in its directory
* After successful build, copy `acmeair-jmeter/builds/libs/*.jar` to `(jmeter installation directory)/lib/ext`
* Also put [json-simple](https://code.google.com/archive/p/json-simple/) in `(jmeter installation directory)/lib/ext`

### Running
Under Windows redirect all output to `nul` or some file. Console output on Windows can introduce significant overhead. 
* Start MongoDB (on Windows: `mongod -dbpath c:\some_db_path`), redirect ouptut to `nul` ,For Ubuntu MongoDB will start automately after installation. If MongoDB doesn't start, you can run `sudo service mongod start`
* Goto AcmeAir directory and resolve module dependencies `npm install`
* Start AcmeAir (`node app.js` in the AcmeAir directory), redirect output to `nul`, otherwise performance will be 2x worse.
* Populate the AcmeAir database by opening http://127.0.0.1:9080/rest/api/loader/load?numCustomers=10000, you can just use `wget` to open the website on bash.
* Run JMeter in benchmarking/experimental/benchmarks/acmeair/jmeter_scripts/: `jmeter -Jduration=240 -Jdrivers=25 -Jhost=127.0.0.1 -Jport=9080 -DusePureIDs=true -n -t AcmeAir.jmx -p acmeair.properties -l log_filename`
* `-Jduration=240` regulates for how long the benchmark will be running. While processing, JMeter will print current throughtput on the screen. After about 4 minutes, the log file will contain details.

### Processing Results
#### Option 1
A node.js application will summarize results and output JSON objects with summary data. It also has options to skip the first n seconds, and an option to aggregate results into time-based groups. 
You'll need to install [node-chakra core](https://github.com/nodejs/node-chakracore/releases) or [node.js](https://nodejs.org/en/download/), and then `cd node-benchmarking\LogAnalysis && npm install`.

Examples:
 - `node LogAnalysis\analyze.js --help` to see help options
 - `node LogAnalysis\analyze.js <results-directory> <results-file-prefix>` to see all results summarized as a single group.
 - `node LogAnalysis\analyze.js --skip 60 <results-directory> <results-file-prefix>` to skip the first 60 seconds of results (e.g., to ignore warm-up times)
 - `node LogAnalysis\analyze.js -s 60 <results-directory> <results-file-prefix>` equivalent to --skip 60
 - `node LogAnalysis\analyze.js --bucketLength 10 <results-directory> <results-file-prefix>` to summarize results in 10-second windows.
 - `node LogAnalysis\analyze.js -b 10 <results-directory> <results-file-prefix>` equivalent to --bucketLength 10

**Note**:  For compatibility with the old c++ tool (option 2 below), you'll want to run `node LogAnalysis\analyze.js --skip 120`

#### Option 2
Compile `LogProcess/JmeterProcess/jmeter.cpp` (Visual Studio solution is provided). Usage is the same as with processing core benchmarks:

`JmeterProcess.exe <directory_with_bechmark_results> <first_group_name> [nth_group_name...]`

`group_name_1` will be used as filename base. This tool will search for `first_group_name1`, `first_group_name2`, etc. files in `directory_with_bechmark_results`. The average results for all those files will be calculated. You can provide more than one group. For each successive group, the percentage change in performance will also be computed.
