# Challenge setup

A repo and associated EvalAI challenge page will be created (and pre-configured) for each benchmark. Each POC needs to modify and push a series of challenge specific files.

```
├── annotations                                 # Contains the annotations for Dataset splits
│   └── test_annotations_testsplit.json         # Annotations for test split
├── challenge_config.yaml                       # Configuration file to define challenge setup
├── evaluation_script                           # Contains the evaluation script
│   ├── __init__.py                             # Imports the modules that involve annotations loading etc
│   └── main.py                                 # Contains the main `evaluate()` method
├── logo.jpg                                    # Logo image of the challenge
EvalAI website
└── templates                                   # Contains challenge related HTML templates
    ├── challenge_test_phase_description.html   # Challenge Test Phase description template
    ├── description.html                        # Challenge description template
    ├── evaluation_details.html                 # Contains description about how submissions will be evaluated for each challenge phase
    ├── submission_guidelines.html              # Contains information about how to make submissions to the challenge
    └── terms_and_conditions.html               # Contains terms and conditions related to the challenge
```

`Annotations/`: This folder should contain the ground truth annotations for the test set. `evaluation_script/main.py` will read this json and compare it to the user submitted json to generate scores

`Challenge_config.yaml`: Information about what metrics to display will be shown here. The main section here is the leaderboard which needs to be modified. Only change the keys specified in the comments. The other config has already been done (e.g. start/end date of the challenge, data splits, leaderboard visibility…)

`Evaluation_script/`: all evaluation code goes here. `__init__.py` allows you to install any packages that you require for evaluation. `main.py` must have an `evaluate()` method that produces a dict with metrics. See LTA example.

`EvalAI website/`: A series of files that describe the challenge, prizes, evaluation details, T&C etc. These are html files so there’s plenty of flexibility about what goes in here, but please try and keep the overall structure uniform across challenges following the template (e.g., Ego4d logo placement, challenge logo, Goal -> Task -> Prizes structure etc.)


## Steps to be taken by challenge POCs

### Create challenge repo:


1. Create a new **empty** repo for your challenge in the `https://github.com/EGO4D-Consortium` organization. The name of the repo should be `evalai-<your challenge short name>-challenge`. For example, the "Song-Term Anticipation (STA)" challenge will name their repo `evalai-sta-challenge`. Lets call the name of this repo as `$CHALLENGE_REPO_NAME`.

2. Next, clone the `evalai-base` repo into your repo. [This repo](https://github.com/EGO4D-Consortium/evalai-base) is an Ego4D specific template (of an existing [EvalAI template](https://github.com/Cloud-CV/EvalAI-Starters)) and contains common configuration details and challenge html templates that every challenge repo should have for consistency. Each challenge repo should ensure that they are synced with this base repo before pushing changes. All the centralized changes to the templates will be pushed to the base repo, hence your challenge repo will have to stay in-sync with the upstream repo. This is how you should set it up:

```
$ export CHALLENGE_REPO_NAME="evalai-sta-challenge"
$ git clone git@github.com:EGO4D-Consortium/evalai-base.git $CHALLENGE_REPO_NAME
$ cd $CHALLENGE_REPO_NAME
$ # now change the remote "origin" to your new repo, and keep the base repo as "upstream"
$ git remote rm origin
$ git remote add origin git@github.com:EGO4D-Consortium/${CHALLENGE_REPO_NAME}.git
$ git remote add upstream git@github.com:EGO4D-Consortium/evalai-base.git
$ git push --set-upstream origin master
```

And you are all set! This repo will have all the good stuff from the base repo, and has base set as upstream, so whenver things change upstream, you will be able to fetch and merge in changes, as follows:

#### Sync-ing your repo to the upstream

This will be required to be done whenver @rohitgirdhar (or anyone controlling the base repo) changes something like challenge start/end dates, high-level templates, terms and conditions etc. Those fields are supposed to be only controlled by the base repo and you should not edit them yourself (or risk running into merge conflicts!!) Here is how you can update your repo to the base repo when asked:

```
```

### Setup EvalAI <--> github access:
Then follow steps 2, 3, 4 from the [EvalAI setup guide](https://evalai.readthedocs.io/en/latest/host_challenge.html) to allow EvalAI to read from this repo.

#### Make changes for your benchmark:
Edit the list of files above, tailoring it to your benchmark. Please stick to the structure laid out in the template html files. Push all changes to the challenge branch (not main).
After pushing, check the actions tab on github. If everything went well, the build would have succeeded. A successful build = changes show up on EvalAI challenge website.

## Testing and Development
EvalAI docs: https://evalai.readthedocs.io/en/latest/

#### Test evaluation script locally:
Link files in your evaluation directory into the local challenge dir
```
ln -s $PWD/evaluation_script/* challenge_data/challenge_1/
```
Run worker. The user submission is submission.json in the root dir. This is exactly the same job that will be run on EvalAI worker nodes so if it succeeds here, it should run there as well.
```
python -m worker.run
```

#### Sync with base repo:
You can pull from the external repo using this command.
```
git pull git@github.com:EGO4D-Consortium/evalai-base.git master --no-rebase --allow-unrelated-histories
```
This will result in several merge conflicts that need to be resolved.
```
git checkout HEAD <file> # Retain the challenge repo's file
git add <file> # accept the external repo's change
git merge --abort # to cancel the entire merging process (maybe it's easier to to this manually)
```
Then commit and push as usual.

Note: There must be a better way. Fork?


### Steps to be taken by overall challenge hosts (Rohit, etc)


