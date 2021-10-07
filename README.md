# nist-synthetic-data-2021
Source code for the second place submission in the third round of the 2021 NIST differential privacy temporal map challenge.

The contest-submission folder contains the code submitted during the contest, and only works on the contest dataset.  A writeup about the solution can be found in this folder: [AdaptiveGrid.pdf](https://github.com/ryan112358/nist-synthetic-data-2021/blob/main/contest-submission/AdaptiveGrid.pdf).  The extensions folder contains a new mechanism, inspired by the solution to the competition, that works on arbitrary discrete datasets.  Several benchmark datasets can be found in the extensions/datasets folder.

## Setting up

The following setup instructions apply to Linux and OSX.  This code has not been tested on Windows, although it should run with a modified setup procedure.  
First, make sure you have Python>=3.6 installed, and create a virtual environment as follows:

```
$ mkdir $HOME/venvs
$ python3 -m venv pgm
$ source ~/venvs/pgm/bin/activate
$ pip install -r requirements.txt
```

This code depends on [Private-PGM](https://github.com/ryan112358/private-pgm).  Private-PGM can be set up using the following commands:
```
$ cd $HOME
$ git clone git@github.com:ryan112358/private-pgm.git
$ echo 'export PYTHONPATH="PYTHONPATH:$HOME/private-pgm/src/"' >> ~/.bashrc
$ source ~/.bashrc
$ cd private-pgm/test
$ nosetests
........................................
----------------------------------------------------------------------
Ran 40 tests in 5.009s

OK
```

## Quick start

After setting up Private-PGM, we can generate synthetic data using the following command

```
$ cd extensions/
$ python adaptive_grid.py --dataset datasets/adult.zip --domain datasets/adult-domain.json --save adult-synthetic.csv

Measuring ('native-country',), L2 sensitivity 1.000000
Measuring ('fnlwgt',), L2 sensitivity 1.000000
Measuring ('relationship',), L2 sensitivity 1.000000
Measuring ('capital-gain',), L2 sensitivity 1.000000
Measuring ('hours-per-week',), L2 sensitivity 1.000000
Measuring ('income>50K',), L2 sensitivity 1.000000
Measuring ('workclass',), L2 sensitivity 1.000000
Measuring ('sex',), L2 sensitivity 1.000000
Measuring ('marital-status',), L2 sensitivity 1.000000
Measuring ('capital-loss',), L2 sensitivity 1.000000
Measuring ('occupation',), L2 sensitivity 1.000000
Measuring ('age',), L2 sensitivity 1.000000
Measuring ('race',), L2 sensitivity 1.000000
Measuring ('education-num',), L2 sensitivity 1.000000

Measuring ('age', 'marital-status'), L2 sensitivity 1.000000
Measuring ('age', 'hours-per-week'), L2 sensitivity 1.000000
Measuring ('age', 'fnlwgt'), L2 sensitivity 1.000000
Measuring ('age', 'capital-gain'), L2 sensitivity 1.000000
Measuring ('age', 'capital-loss'), L2 sensitivity 1.000000
Measuring ('workclass', 'occupation'), L2 sensitivity 1.000000
Measuring ('fnlwgt', 'native-country'), L2 sensitivity 1.000000
Measuring ('fnlwgt', 'race'), L2 sensitivity 1.000000
Measuring ('education-num', 'occupation'), L2 sensitivity 1.000000
Measuring ('marital-status', 'relationship'), L2 sensitivity 1.000000
Measuring ('occupation', 'hours-per-week'), L2 sensitivity 1.000000
Measuring ('relationship', 'sex'), L2 sensitivity 1.000000
Measuring ('relationship', 'income>50K'), L2 sensitivity 1.000000

Post-processing with Private-PGM, will take some time...
```

As we can see, the mechanism measured queries about all 1-way marginal, and a subset of 13 2-way marginals.  This produces an output adult-synthetic.csv that we can radily view

```
$ head adult-synthetic.csv
age,workclass,fnlwgt,education-num,marital-status,occupation,relationship,race,sex,capital-gain,capital-loss,hours-per-week,native-country,income>50K
14,0,13,3,3,9,3,0,0,0,0,46,0,0
10,0,11,12,2,4,5,0,0,0,0,39,0,0
15,0,9,9,0,3,2,0,1,0,45,19,0,0
8,1,16,8,2,10,1,0,0,0,0,19,20,0
32,0,4,13,0,8,2,0,1,0,0,59,0,1
4,0,21,9,2,9,1,0,1,0,0,39,0,0
21,0,10,8,1,7,3,0,0,0,0,39,0,0
31,8,12,6,0,14,2,0,1,0,0,34,0,1
21,0,14,13,1,4,3,3,0,0,0,44,0,0
```

## The target option

By default, this mechanism will try to preserve all 2-way marginals.  If one column has increased importance, we can specify that with the **targets** column.  With this option specified, we will instead try to preserve higher order marginals involving the targets.   If we specify ```--targets="income>50K"``` then the mechanism will try to preserve 3-way marginals involving the income column.  We can pass in multiple targets if desired, although scalability will suffer if the list is longer than a few columns. 

```
$ python adaptive_grid.py --dataset datasets/adult.zip --domain datasets/adult-domain.json --targets="income>50K" --save adult-synthetic-target.csv

Measuring ('income>50K',), L2 sensitivity 1.000000
Measuring ('marital-status',), L2 sensitivity 1.000000
Measuring ('age',), L2 sensitivity 1.000000
Measuring ('race',), L2 sensitivity 1.000000
Measuring ('capital-gain',), L2 sensitivity 1.000000
Measuring ('workclass',), L2 sensitivity 1.000000
Measuring ('relationship',), L2 sensitivity 1.000000
Measuring ('education-num',), L2 sensitivity 1.000000
Measuring ('hours-per-week',), L2 sensitivity 1.000000
Measuring ('capital-loss',), L2 sensitivity 1.000000
Measuring ('fnlwgt',), L2 sensitivity 1.000000
Measuring ('occupation',), L2 sensitivity 1.000000
Measuring ('native-country',), L2 sensitivity 1.000000
Measuring ('sex',), L2 sensitivity 1.000000

Measuring ('marital-status', 'income>50K'), L2 sensitivity 1.000000
Measuring ('race', 'income>50K'), L2 sensitivity 1.000000
Measuring ('relationship', 'income>50K'), L2 sensitivity 1.000000
Measuring ('capital-loss', 'income>50K'), L2 sensitivity 1.000000
Measuring ('fnlwgt', 'income>50K'), L2 sensitivity 1.000000
Measuring ('native-country', 'income>50K'), L2 sensitivity 1.000000
Measuring ('workclass', 'income>50K'), L2 sensitivity 1.000000
Measuring ('occupation', 'income>50K'), L2 sensitivity 1.000000
Measuring ('hours-per-week', 'income>50K'), L2 sensitivity 1.000000
Measuring ('age', 'income>50K'), L2 sensitivity 1.000000
Measuring ('education-num', 'income>50K'), L2 sensitivity 1.000000
Measuring ('capital-gain', 'income>50K'), L2 sensitivity 1.000000
Measuring ('sex', 'income>50K'), L2 sensitivity 1.000000

Measuring ('age', 'marital-status', 'income>50K'), L2 sensitivity 1.000000
Measuring ('age', 'hours-per-week', 'income>50K'), L2 sensitivity 1.000000
Measuring ('age', 'fnlwgt', 'income>50K'), L2 sensitivity 1.000000
Measuring ('age', 'native-country', 'income>50K'), L2 sensitivity 1.000000
Measuring ('age', 'capital-gain', 'income>50K'), L2 sensitivity 1.000000
Measuring ('age', 'capital-loss', 'income>50K'), L2 sensitivity 1.000000
Measuring ('age', 'race', 'income>50K'), L2 sensitivity 1.000000
Measuring ('workclass', 'occupation', 'income>50K'), L2 sensitivity 1.000000
Measuring ('education-num', 'occupation', 'income>50K'), L2 sensitivity 1.000000
Measuring ('marital-status', 'relationship', 'income>50K'), L2 sensitivity 1.000000
Measuring ('occupation', 'hours-per-week', 'income>50K'), L2 sensitivity 1.000000
Measuring ('relationship', 'sex', 'income>50K'), L2 sensitivity 1.000000

Post-processing with Private-PGM, will take some time...
```

As we can see, the mechanism now measured a lot more things about marginals involving the target column.  In particular, it measured all 2-way marginals involving income, and 12 3-way marginals involving income.

## Evaluating the synthetic data

We can score our synthetic data using the score.py function, as follows:

```
$ python score.py --synthetic adult-synthetic.csv
relationship    sex               0.003655
                income>50K        0.003675
marital-status  relationship      0.007279
sex             income>50K        0.008364
marital-status  income>50K        0.011373
                                    ...   
age             education-num     0.139112
occupation      relationship      0.151089
age             hours-per-week    0.157170
occupation      sex               0.159074
age             fnlwgt            0.207127
Length: 91, dtype: float64
Average Error:  0.05595402713661589
```

The error is calculated as an total variation distance between true and synthetic marginals, averaged over all 2-way marginals.  We can see both the breakdown (which marginals are estimated well and which are not), and the overall error.  We can also specify a list of targets, which modifies the evaluation criteria to include the target columns in all evalaution marginals.

```
$ python score.py --synthetic adult-synthetic-target.csv --targets "income>50K"
relationship    sex             income>50K    0.005139
marital-status  relationship    income>50K    0.011445
workclass       race            income>50K    0.024426
sex             capital-loss    income>50K    0.027927
marital-status  sex             income>50K    0.028029
                                                ...   
occupation      relationship    income>50K    0.140740
age             education-num   income>50K    0.151325
occupation      sex             income>50K    0.161418
age             hours-per-week  income>50K    0.163691
                fnlwgt          income>50K    0.174420
Length: 78, dtype: float64
Average Error:  0.06613134555274515
```

**NOTE**: By specifying targets in adaptive_grid.py, we can expect the synthetic data to score better when passing --targets to score.py.  If we score adult-synthetic.csv with the target option enabled, the score is 0.1038, almost 2X worse than the 0.0661 we achieved.

We can compare the score we obtain from our differentially private mechanism with that of a simple non-private baseline which samples n records with replacement from the original dataset.  We can run this baseline using resample.py and score it using the same score function.

```
$ python resample.py --dataset datasets/adult.csv --save resample.csv
$ python score.py --synthetic resample.csv
sex           income>50K        0.001679
relationship  sex               0.002191
race          income>50K        0.002211
relationship  income>50K        0.002764
capital-loss  income>50K        0.003030
                                  ...   
age           education-num     0.042545
              occupation        0.046599
fnlwgt        hours-per-week    0.048462
age           hours-per-week    0.063142
              fnlwgt            0.069919
Length: 91, dtype: float64
Average Error:  0.014676838660295519
```


## Full configuration options

The default configuration options are shown below.  In general, the dataset, domain, epsilon, delta, targets, and save options should be specified.  For other options, the defaults settings should work fine in most cases.  Interested users can try modifying them if desired.

```
$ cd extensions/
$ python adaptive_grid.py --help
usage: adaptive_grid.py [-h] [--dataset DATASET] [--domain DOMAIN] [--epsilon EPSILON]
                        [--delta DELTA] [--targets TARGETS [TARGETS ...]] [--pgm_iters PGM_ITERS]
                        [--warm_start WARM_START] [--metric {L1,L2}] [--threshold THRESHOLD]
                        [--split_strategy SPLIT_STRATEGY [SPLIT_STRATEGY ...]] [--save SAVE]

A generalization of the Adaptive Grid Mechanism that won 2nd place in the 2020 NIST temporal map
challenge

optional arguments:
  -h, --help            show this help message and exit
  --dataset DATASET     dataset to use (default: datasets/adult.zip)
  --domain DOMAIN       dataset to use (default: datasets/adult-domain.json)
  --epsilon EPSILON     privacy parameter (default: 1.0)
  --delta DELTA         privacy parameter (default: 1e-10)
  --targets TARGETS [TARGETS ...]
                        target columns to preserve (default: [])
  --pgm_iters PGM_ITERS
                        number of iterations (default: 2500)
  --warm_start WARM_START
                        warm start PGM (default: True)
  --metric {L1,L2}      loss function metric to use (default: L2)
  --threshold THRESHOLD
                        adagrid treshold parameter (default: 5.0)
  --split_strategy SPLIT_STRATEGY [SPLIT_STRATEGY ...]
                        budget split for 3 steps (default: [0.1, 0.1, 0.8])
  --save SAVE           path to save synthetic data (default: out.csv)

$ python score.py --help
usage: score.py [-h] [--dataset DATASET] [--domain DOMAIN]
                [--synthetic SYNTHETIC] [--targets TARGETS [TARGETS ...]]
                [--save SAVE]

A script to score the quality of synthetic data

optional arguments:
  -h, --help            show this help message and exit
  --dataset DATASET     dataset to use (default: datasets/adult.zip)
  --domain DOMAIN       domain of dataset (default: datasets/adult-
                        domain.json)
  --synthetic SYNTHETIC
                        synthetic dataset to use (default: out.csv)
  --targets TARGETS [TARGETS ...]
                        target columns to define evaluation criteria (default:
                        [])
  --save SAVE           path to save error report (default: error.csv)
```

Notes about other options:
* The metric options corresponds to the loss function used to resolve inconsistencies in noisy marginals.  The L2 loss function is more natural with Gaussian noise, which is what we use, and also has better smoothness properties, making optimization simpler.  We don't recommend changing this, but feel free to try to see if it makes a difference in your use case.
* The pgm_iters specifies how many iterations to run the proximal gradient algorithm underlying Private-PGM.  Increasing this value may improve error slightly, at the cost of increased runtime.  Decreasing this value too aggressively could destroy performance.
* The warm_start option is activated during the second invocation of Private-PGM after step 3.  If it is turned on, then the parameters from in the previous invocation will be used to initialize the proximal algorithm.  
* The threshold option is used to determine whether to measure a cell in a marginal at finer granularity.  If a cell in a marginal had noisy count below threshold\*sigma, then no future measurements will be made at finer granularity.  The mechanism seems fairly robust to the choice of threshold, and even setting it to -inf should be fine (no threshold).  Feel free to modify it and see how it impact performance on your problem, but expect the default to provide reasonable behavior.
* the split_strategy option specifies how much of the privacy budget to devote to the three steps of the algorithm.  By default 10% is used on step 1 (measuring 1 way marginals), 10% is used on step 2 (selecting higher order marginals), and 80% is used on step 3 (measuring higher order marginals).  Since the first two steps are coarse-grained aggregations, they are more robust to noise than step 3.  Again, feel free to change this, but expect the default to work reasonably well.
