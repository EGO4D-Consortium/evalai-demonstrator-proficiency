Glossary:
- POC: Person of contact. The person setting up and hosting a specific challenge.
- Admin: Currently Tushar Nagarajan or Rohit Girdhar. We manage the main evalai-base repo which will be forked for all challenges. This repo would contain the "common" information like terms, logos, start/end dates etc.
- EvalAI Admin: Currently [Ram Ramrakhya](https://ram81.github.io/) and [Rishabh Jain](https://rishabhjain.xyz/)

# Challenge setup

A repo and associated EvalAI challenge page will be created (and pre-configured) for each benchmark. Each POC needs to modify and push a series of challenge specific files.

```bash
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

`Challenge_config.yaml`: Information about what metrics to display will be shown here. The main section here is the leaderboard which needs to be modified. Only change the keys specified in the comments. The other config has already been done (e.g. start/end date of the challenge, data splits, leaderboard visibility…)

`Evaluation_script/`: all evaluation code goes here. `__init__.py` allows you to install any packages that you require for evaluation. `main.py` must have an `evaluate()` method that produces a dict with metrics. See LTA example.

`EvalAI website/`: A series of files that describe the challenge, prizes, evaluation details, T&C etc. These are html files so there’s plenty of flexibility about what goes in here, but please try and keep the overall structure uniform across challenges following the template (e.g., Ego4d logo placement, challenge logo, Goal -> Task -> Prizes structure etc.)


## Steps to be taken by challenge POCs

### Step 1: Create challenge repo

1. Create a new **empty** repo for your challenge in the `https://github.com/EGO4D-Consortium` organization. The name of the repo should be `evalai-<your challenge short name>-challenge`. For example, the "Short-Term Anticipation (STA)" challenge will name their repo `evalai-sta-challenge`. Lets call the name of this repo as `$CHALLENGE_REPO_NAME`.

2. Next, clone the `evalai-base` repo into your repo. [This repo](https://github.com/EGO4D-Consortium/evalai-base) is an Ego4D specific template (of an existing [EvalAI template](https://github.com/Cloud-CV/EvalAI-Starters)) and contains common configuration details and challenge html templates that every challenge repo should have for consistency. Each challenge repo should ensure that they are synced with this base repo before pushing changes. All the centralized changes to the templates will be pushed to the base repo, hence your challenge repo will have to stay in-sync with the upstream repo. The name of the branch **MUST** be `challenge`. This is how you should set it up:

```bash
$ export CHALLENGE_REPO_NAME="evalai-sta-challenge"
$ git clone git@github.com:EGO4D-Consortium/evalai-base.git $CHALLENGE_REPO_NAME
$ cd $CHALLENGE_REPO_NAME
$ # now change the remote "origin" to your new repo, and keep the base repo as "upstream"
$ git remote rm origin
$ git remote add origin git@github.com:EGO4D-Consortium/${CHALLENGE_REPO_NAME}.git
$ git remote add upstream git@github.com:EGO4D-Consortium/evalai-base.git
$ git push --set-upstream origin challenge
```

And you are all set! This repo will have all the good stuff from the base repo, and has base set as upstream, so whenever things change upstream, you will be able to fetch and merge in changes (see Step N below).


### Step 2: Setup EvalAI <--> github access
Then follow steps 2, 3, 4 from the [EvalAI setup guide](https://evalai.readthedocs.io/en/latest/host_challenge.html) to allow EvalAI to read from this repo. I recap them here:

0. Create an EvalAI account, and ask admins to add you to the "Ego4D" team. You might not be able to create challenges until you are added to this team.
1. Create a [github personal acccess token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) and copy it in clipboard. These are the permissions you need to grant:
   ![image](https://user-images.githubusercontent.com/1893429/156271507-1ad220a5-d7d1-4445-85f9-93a930dfd353.png)

3. For the your repo, go to Settings, select "Secrets" from the sidebar, click on "Actions", create a "New Repository Secret". Make a token with name "AUTH_TOKEN", and copy your github personal access token into the text box.
4. Now go to `github/host_config.json` in your repo, and update the 3 values in there:
   - `evalai_user_auth_token` - Go to [profile page](https://eval.ai/web/profile) after logging in and click on `Get your Auth Token` to copy your auth token.
   - `host_team_pk` - This should be "1744" -- which is the team ID for "Ego4D" team. You should be able to see it on [host team page](https://eval.ai/web/challenge-host-teams) if you have been added. If not, contact admins to add you. 
   - `evalai_host_url` - Set to https://eval.ai


### Step 3: Make changes for your benchmark
Edit the list of files above, tailoring it to your benchmark. Please stick to the structure laid out in the template html files. Push all changes to the challenge branch (not main). Specifically, you need to update the `challenge_config.yaml` and associated template files:

- Update the `title`, `short_description`, `description.html`. Please use Ego4D "branding". The name should be "Ego4D: <title of your challenge>"
- Update the `evaluation_details.html`. It should contain the metrics that will appear on the leaderboard, and which metric will be used for ranking the submissions.
- Please do not change the terms and conditions or the logo.
- Update `submission_guidelines.html` and `leaderboard_description`
- Update the metric names, descriptions, which will be used to sort
- Please update `templates/challenge_test_phase_description.html`

After pushing, check the actions tab on github. If everything went well, the build would have succeeded. A successful build = changes show up on EvalAI challenge website.

### Step 4: Writing the evaluation script
Modify the `evaluate()` function in `evaluation_script/main.py` according to the benchmark definition. The evaluate function looks like this:
```
def evaluate(test_annotation_file, user_annotation_file, phase_codename, **kwargs):
    ...
```

`test_annotation_file` is the path to the ground truth test annotations, while `user_annotation_file` is the path to the file uploaded by the user for evaluation. Since we have only one test phase, `phase_codename` will always be `test` and can be ignored. The function must read these files, calculate relevant metrics and then return a dictionary of metrics as follows:
```python
output = {}
output['result'] = [
   {
       'test_split': {
           'Metric1': 123,
           'Metric2': 123,
           'Metric3': 123,
           'Total': 123,
       }
   }
]
```

For example, a simple version of the `main.py` script could look like this.
```python
import numpy as np
import json

def calculate_top1(scores, labels):
   # calculate top1 accuracy
   ...
   return acc_top1

def calculate_top5(scores, labels):
   # calculate top5 accuracy
   ...
   return acc_top5

def evaluate(test_annotation_file, user_annotation_file, phase_codename, **kwargs):
   gt_data = json.load(open(test_annotation_file, 'r'))
   pred_data = json.load(open(user_annotation_file, 'r'))

   output = {}
   output['result'] = [
      {
          'test_split': {
              'accuracy_top1': calculate_top1(pred_data['scores'], gt_data['labels']),
              'accuracy_top5': calculate_top5(pred_data['scores'], gt_data['labels']),,
          }
      }
   ]
   return output
```

Note: If your evaluation pipeline requires extra packages to be installed via pip, these can be specified in `evaluation_script/__init__.py` before importing `main`.
More info: https://evalai.readthedocs.io/en/latest/evaluation_scripts.html

### Step 4* (Skip this step if you do not need external non-Python packages!)
Please follow this step only if you need an external package that **cannot** be installed by `pip`! Currently, eval.ai does not support installation of non-pip packages from `__init__.py`, especially the ones that require `make`. Suppose that you need a package that is written in bash, perl, C++, etc. and assume that the external code is in a Git repository (e.g. NIST sclite tool for automatic speech recognition evaluation), then you should ask the eval.ai team to install this library on a worker. Please note that if you manually restart or stop the worker in the Manage tab of your challenge page, or if you get a new worker, the installation has to be repeated on the newly assigned worker which requires contacting an eval.ai team member every time (Note: this is also needed when you update your evaluation scripts `evaluation_script/__init__.py` and `evaluation_script/main.py` because you need a restart so that the recent changes can be reflected to the eval.ai page). 

Once your worker has the required package, you may need to add the path of the library or the binary to your path. Therefore, please also ask for the installation directory when you contact the eval.ai member. Then in your `evaluation_script/__init__.py` you can add the necessary paths to your environment in Python (e.g. `sys.path.insert(0, <your_bin_directory>)`). If you need to run the binary in your `evaluation_script/main.py` script, you can create a string variable e.g., `cmd_str` which corresponds to the command that you would run in your shell. In order to run this command, you can use `subprocess.run(cmd_str, env={**os.environ, 'PATH': ':'.join(sys.path)}, shell=True)`. Please note that we need to add the `PATH` variable and to the environment and set `shell=True`. You may also add other arguments to the `run()` function depending on your use case. It might be useful to have your final result written to a file so that once the `run()` process finishes, you can parse this output to get your score. If you generate such scoring files, please do not forget to remove them at the end to prevent memory problems in the worker. 

Past challenges that have used this: [AV Trans](https://github.com/EGO4D-Consortium/evalai-avtrans-challenge) (POC: Leda Sari)

### Step 5: Test evaluation script locally:
To test locally, place the ground truth annotations into the annotations directory `annotations/test_annotations_testsplit.json`. Do **NOT** push this file into the github repo. You will upload it via a CLI tool later. Also place a `submission.json` in the root directory. This is to represent the user generated submission. 

Link files in your evaluation directory into the local challenge dir
```bash
ln -s $PWD/evaluation_script/* challenge_data/challenge_1/
```
This is exactly the same job that will be run on EvalAI worker nodes so if it succeeds here, it should run there as well.
```bash
python -m worker.run
```

### Step 6: Upload GT annotations using the EvalAI CLI.
Install and set up the CLI. The `auth_token` is the same as the one in `github/host_config.json`.
```bash
$ pip install evalai
$ evalai set_token <auth_token>
```

Find your challenge ID and test phase ID.
```bash
$ evalai challenges --host

+------+------------------------------------------------+--------------------------------------------------+---------------------+----------------------+----------------------+
|  ID  |                     Title                      |                Short Description                 |       Creator       |      Start Date      |       End Date       |
+------+------------------------------------------------+--------------------------------------------------+---------------------+----------------------+----------------------+
| 1598 | Ego4D: Long term action anticipation challenge | Ego4D challenge on Long term action anticipation |        Ego4D        | 03/01/22 04:00:00 PM | 05/31/22 04:59:59 PM |
+------+------------------------------------------------+--------------------------------------------------+---------------------+----------------------+----------------------+

$ evalai challenge 1598 phases

+----------+------------+--------------+------------------------------------+
| Phase ID | Phase Name | Challenge ID |            Description             |
+----------+------------+--------------+------------------------------------+
|   3161   | Test Phase |     1598     | Test phase for the LTA challenge   |
+----------+------------+--------------+------------------------------------+
```

Upload test annotations for this phase. This is the file that will be passed to your `evaluate()` function in Step 4.
```bash
$ evalai challenge 1598 phase 3161 submit --file test_annotations_testsplit.json --annotation --large
```

NOTE: Please no not upload the test annotations directly to github. Use the CLI tool to ensure that they only exist on the EvalAI servers.
NOTE 2: If you update your challenge, you might have to upload these annotations again through the CLI.
NOTE 3: It takes ~5-10 minutes for the workers to pick up the latest annotation files, so you might have to wait a bit after uploading the test annoations before the evaluation acn be done correctly. Before that it might still pick up the dummy file from the repo.

### Step 7: Make a baseline submission
To test your setup and report your baseline results, we need to make a submission using your baseline code to generate a submission JSON and upload it through the EvalAI system. Once submitted, make it public on the leaderboard, and mark it as a "baseline" by going to your submissions. It will show up on the leadboard with a "B" to denote that this is the official baseline ([example here](https://eval.ai/web/challenges/challenge-page/802/leaderboard/2195)). You might have to create a participant team before you see the submit option appears. Please make sure to submit a baseline, as without it, no participant will be eligible for prizes (as per our rules, they must outperform the baseline on the "primary" metric to be eligible). For debugging, you might want to increase the `max_submissions_per_day` field in the `challenge_config.yaml` so you can make multiple submissions until things work. Do remember to set it back to the default after.
    
    
### Step N: Sync-ing your repo to the upstream

This will be required to be done whenever the admins change something like challenge start/end dates, high-level templates, terms and conditions etc. Those fields are supposed to be only controlled by the base repo and you should not edit them yourself (or risk running into merge conflicts!!) Here is how you can update your repo to the base repo when asked. The older rebase solution (below) was cumbersome, so I'd recommend doing a merge instead:

```bash
$ git remote -v  # Check your remotes are setup correctly; it should look something like this
origin	git@github.com:EGO4D-Consortium/${CHALLENGE_REPO_NAME}.git (fetch)
origin	git@github.com:EGO4D-Consortium/${CHALLENGE_REPO_NAME}.git (push)
upstream	git@github.com:EGO4D-Consortium/evalai-base.git (fetch)
upstream	git@github.com:EGO4D-Consortium/evalai-base.git (push)
$ git pull upstream challenge
$ # Ideally there shouldn't be any merge conflicts. If there are, fix them (you likely changed something that should only be changed in base)
$ git push origin challenge
```
    
<del>
```bash
$ git remote -v  # Check your remotes are setup correctly; it should look something like this
origin	git@github.com:EGO4D-Consortium/${CHALLENGE_REPO_NAME}.git (fetch)
origin	git@github.com:EGO4D-Consortium/${CHALLENGE_REPO_NAME}.git (push)
upstream	git@github.com:EGO4D-Consortium/evalai-base.git (fetch)
upstream	git@github.com:EGO4D-Consortium/evalai-base.git (push)
$ git fetch upstream
$ git rebase upstream/challenge
$ # Ideally there shouldn't be any merge conflicts. If there are, fix them (you likely changed something that should only be changed in base)
$ git push origin challenge
```
</del>

You challenge will be rebuilt and updated with the latest updates on the base repo.
    
    
### Step N+1: Sign-up for notifications on the discuss channel for your challenge
Once the challenges are made public, the EvalAI admins will set up a discussion forum for your challenge. As the challenge POC, please subscribe to notifications on that forum so you are notified whenever somebody posts a question. This is important to ensure the community has a way to communicate with the challenge hosts, resolve issues etc. The steps are quite straightforward:
    
1. Go to the "discuss" page on your challenge. It will lead you to a link somethine like this: https://evalai-forum.cloudcv.org/c/ego4d-object-state-change-detection/61
2. Create an account on that discuss page. Unfortunately EvalAI accounts don't work there. I personally just linked it to my github account (it allows you to login via github), but feel free to use whatever you like.
3. Once you are logged in, open your challenge's discussion page again, and click the "notification bell icon" and select "watching" so you get notified for all the posts in your challenge.
    <img width="1165" alt="image" src="https://user-images.githubusercontent.com/1893429/158888901-e3ab9341-92aa-4fb3-a601-9e5664335689.png">
4. After that please do try to respond to all the questions in a timely manner so we can ensure our challenge participants have a positive experience :)


### More info
EvalAI docs: https://evalai.readthedocs.io/en/latest/

