# DQMScripts

This package allows E/gamma (HLT so far) to manipulate the histograms stored in the DQM for easier validation. There are seperate scripts for the type of monitoring, so far MC RelVals and Prompt Data 

## Prompt Data Monitoring workflow:

The prompt data monitoring is used to monitor the quality of the data taken over the course of the year. 

The script:

    pyScripts/makePromptDQMPlots.py  egammaDQMHists.root -r hltData2018_runInfo.json  -o eghltweb
 
generates a webpage with all the monitoring information for easy viewing. It requires two input files, "egammaDQMHists.root" a file containing the histograms from the DQM and "hltData2018_runInfo.json" a file containing information about the runs. It outputs a webpage with all the validation information in the directory "eghltweb"

### Downloading the histograms from the DQM gui:

To download the histograms from the gui, simply use this command
    
    pyScripts/egHLTDQMDownloader.py --output egammaDQMHists.root --dataset /EGamma/Run2018\*-PromptReco-v\*/DQMIO

This will dump all the required histograms for all runs in any of the datasets matching the patter specified after "--dataset" into the root file "egammaDQMHists.root". It is enforced that a run only appears once per primary dataset, it resolves this by taking the run from the newest dataset. This can take some time, say around 20-30mins to complete. 

### Generating the runInfo file:

The run info file is used to group the runs into fills. It is generated by a different package https://github.com/Sam-Harper/wbmtools, which involves installing some dependences. 


### Full setup command and workflow

First to setup a working directory. 

    mkdir egammaDQM #call it whatever
    cd egammaDQM
    #first setup CMSSW, we only do this to ensure we all have
    #a consistent python + root environment, we dont depend on
    #CMSSW otherwise, so you can skip if you already setup 
    #CMSSW_10_1 or higher somewhere in your login
    cmsrel CMSSW_10_1_0
    cd CMSSW_10_1_0/src
    cmsenv
    cd -
    #now download and setup the two packages
    git clone git@github.com:cms-egamma/DQMScripts.git 
    git clone git@github.com:Sam-Harper/wbmtools.git
    #install the packages for wbmtools
    cd wbmtools
    python -m virtualenv virenv
    source virenv/bin/activate #assuming your using bash
    pip install -r requirements.txt
    deactivate #exits the virtual python environment
    #one last thing, we need the cern CA bundle to verify the WBM cert
    scp lxplus.cern.ch:/etc/ssl/certs/ca-bundle.crt ./
    cd -
    
    
Now once we have an area setup, the workflow is as follows
    
    #first generate the runInfo file
    source wbmtools/virenv/bin/activate #we now go into our special python env
    export PYTHON27PATH=$PYTHON27PATH:wbmtools 
    export REQUESTS_CA_BUNDLE=$PWD/wbmtools/ca-bundle.crt 
    ./wbmtools/bin/getRunData.py -o DQMScripts/hltData2018 #get a coffee it'll take a while
    deactivate #puts us back to the normal python env
    
    #now download histograms from the DQM gui
    cd DQMScripts
    ./pyScripts/egHLTDQMDownloader.py --output egammaDQMHists.root --dataset /EGamma/Run2018*-PromptReco-v\*/DQMIO
    
    #now run over the histograms and generate the webpage
    ./pyScripts/makePromptDQMPlots.py  egammaDQMHists.root -r hltData2018_runInfo.json  -o eghltweb
    
### Output

The script with generate a simple webpage with the e/gamma DQM histograms displayed. It automatically groups runs into fills and sums them together. For each specified path, it generates webpage comparing all fills to a reference set of runs. Additionally it will update with new runs each week and generate a webpage corresponding to which runs are new compared to the previous time it was run. It is enough for a single run in a fill to be new to consider the fill new so the same fill may appear in multiple weeks. In the weekly summary it will also plot the efficiency for all new runs summed together. A week can be updated multiple times during that week, if its still the "current" week, it will just regenerate that page for the week so you can monitor it during the week. A new week begins on wednesday (aka TSG meeting). 
