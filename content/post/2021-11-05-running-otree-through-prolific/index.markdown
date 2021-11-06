---
title: Running oTree on Prolific for Beginners
author: ''
date: '2021-11-05'
slug: running-otree-on-prolific-for-beginners
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-11-05T13:12:50-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---





[oTree](https://www.otree.org/) is an open-source, Python-based framework that lets you build Multiplayer strategy games, behavioral experiments, and surveys/quizzes.

Despite its extreme versatility, picking up a new software platform for conducting studies can have a steep learning curve. Combined with a need to learn Python (and javascript/html/Django) makes it feel like a steep curve straight into [hell]{style="color:red"}.

One of the many hurdles I have faced in my oTree journey is getting my project properly onto a server and getting it implemented in Prolific.

<br>

To my horror, most guides and forums assume some degree of knowledge and competence in programming.

<br>

This guide is designed for people like me: those just getting into programming, with small tasks often ballooning into week-long slogs of work, frustration, and probably depression. What follows is a step-by-step instruction to get you from only running your project in localhost to having it be accessible to participants on Prolific. It is broken into five sections:

1.  Assumptions
2.  Otree side
3.  Heroku side
4.  Prolific side
5.  Final implementation

This guide is meant to be followed sequentially. For example, you'll have a rougher time skipping straight to the Heroku section, because some of the referenced objects are built for my example environment

<br>

[**I also use color coding:**]{.ul}

[**Green** is for mandatory steps]{style="color:green"}

[**Orange** is for steps for extra testing -- not mandatory, but highly recommended - can help troubleshoot and build understanding -- FAIL EARLY AND OFTEN]{style="color:orange"}

[**Red** is for cautionary notes]{style="color:red"}

And **Black** is for general description and contextualizing <br> <br>

I am one of the least qualified people to give advice on topics of software. Better resources and smarter people can be found at the oTree docs <https://otree.readthedocs.io/en/latest/index.html>, the oTree Google groups forum <https://groups.google.com/g/otree>, or the Otree
discord channel <https://discord.com/channels/857869226554818592/857869227066130444>.

Special thanks to Chris Wickens for being active not just in the development of oTree, but in responding to the many questions posed by the community.

I'm also indebted to those that have posted video tutorials on using oTree:

-   oTree tutorials

    -   <https://www.youtube.com/watch?v=VrPdBEghYEM>

-   Accounting experiments

    -   <https://www.accountingexperiments.com/post/otree-tutorial-part-4/>

# Assumptions

I don't know whether these assumptions are a necessary part of being able to follow this workflow, but I do know I got it working in this environment

1.  Have your project set up in PyCharm (free license available for
    students and academics)

2.  Have a functioning, multi-app otree project

    -   I'm a novice, so this guide might be limited to one-sided (i.e.,
        non-partnered) studies, check accordingly

3.  Have otree up-to-date to your preferences

    -   `pip install otree`

        -   to install otree lite (aka. Otree 5 -- not dependent on
            Django)

    -   If you would like to guarantee that you stay on version 3.x, use
        this command next time you upgrade:

        -   `pip3 install -U "otree\<5"`
            (<https://groups.google.com/g/otree/c/D5TZseoELN0/m/eLLw1gTFBAAJ>)

4.  You wish to connect heroku through github (instead of using otreehub
    or a command line method)

# Otree Side

**How to integrate Prolific IDs in study**

Identifying and storing Prolific IDs are useful for making sure that participants are paid properly (e.g., making sure an ID isn't seen twice in the study, or being able to confirm whether an ID provided valid responses)

Working with Prolific IDs in oTree is accomplished through configuring a **room** and the **study url**

1.  [Go to your settings.py and set ROOMS with your desired name (used
    in url) and display_name]{style="color:green"}

2.  [set up a room with a participant label file]{style="color:orange"}

    -   [this is just for testing, on implementation, you will NOT have
        a participant label file]{style="color:red"}

    -   [try a simple .txt document, with two rows, first with
        test_prolific_id, second with
        id_prolific_test]{style="color:orange"}

For example:


```r
ROOMS = [
    dict(
        name='your_study',
        display_name='your_study',
        # participant_label_file='_rooms/your_study.txt',
        # use_secure_urls=True,
    ),
    dict(
        name='your_prolific_study',
        display_name='your_prolific_study3',
        participant_label_file='_rooms/your_study.txt',
        # use_secure_urls=True,
    ),
]
```

3.  [on one of your early apps, assign the location (player), and method (before_next_page) for extracting participant.label]{style="color:green"}

    -   [(try one after your consent page so you don't extract
        unnecessary info)]{style="color:green"}


```r
class Player(BasePlayer):
    prolific_id = models.StringField(default=str(" "))


class Demographics(Page):
    form_model = 'player'
    form_fields = ['age', 'gender', 'education', "income"]

    @staticmethod
    def before_next_page(self, timeout_happened):
        self.prolific_id = self.participant.label
pass
```

4.  [click the 'Rooms' tab through
    <http://localhost:8000/>]{style="color:orange"}

    -   [select "close this room" if a session is active, then
        restart]{style="color:orange"}

5.  [When creating a new session, select the Session Config that you wish to test]{style="color:orange"}

6.  [input at least one participant (try two at first)]{style="color:orange"}

    -   If you scroll down to Participant-specific URLs on this page and
        click "show", you'll see URL-participant_label combos that were
        generated from the participant label file

7.  [select "create"]{style="color:orange"}

    -   If you scroll down to Participant-specific URLs on this page,
        you'll see individual links, but they won't invoke a participant
        label

    -   [Instead, try clicking the Room-wide URL, you'll then be
        prompted to enter a participant label]{style="color:orange"}

    -   [In this case, you can enter test_prolific_id or
        id_prolific_test]{style="color:orange"}

        -   This is not what you want participants to see -- instead you
            want these participant labels extracted from the URL

        -   [For URL extraction, directly copy/paste in browser (after
            changing study name)]{style="color:orange"}

            -   [[http://localhost:8000/room/your_prolific_study?participant_label=id_prolific_test]{style="color:orange"}](http://localhost:8000/room/your_prolific_study?participant_label=id_prolific_test){.uri}

            -   [[http://localhost:8000/room/your_prolific_study?participant_label=test_prolific_id]{style="color:orange"}](http://localhost:8000/room/your_prolific_study?participant_label=test_prolific_id){.uri}

8.  [Back on the menu, click on the monitor icon]{style="color:orange"}

    -   Here, you can see the participants, and the label they have

9.  [Progress through the demo, until you're done the page with the before_next_page function (in this case, demographics)]{style="color:orange"}

10. [Click on the data icon]{style="color:orange"}

    -   At this stage, you should see the filled prolific_id field --
        this confirms that the participant label is being passed from
        your .txt file, through the participant.label object, and stored
        in a variable that can be downloaded!

> ![](otree_data.png){width="75%"}

The above verifies that you can work with participant label information,
[but doesn't work yet with a Prolific ID.]{style="color:red"}

## After testing participant labels

1.  [Remove/comment out the participant label file (if you haven't
    already)]{style="color:green"}

The next thing is to change the input for the url's participant_label
parameter to {{%/*PROLIFIC_PID*/%}}, which gets substituted with the
participant's actual prolific ID. This ID needs to be configured and
tested on Prolific's end; oTree is not involved in that (this will be
explained in section 4 -- Prolific Side).

Before this step, you'll need to change the link from local host to
wherever you're hosting the link (e.g., Heroku -- explained below)

**Linking to completion page**

Recommended practice in prolific seems to be  re-directing participants back to the prolific website for the completion code. This can be specified in the session configs like so:

2.  [Update completionlink field in session_configs with a link to a
    prolific completed submission site with your custom completion
    code]{style="color:green"}


```r
SESSION_CONFIGS = [
dict(name='your_prolific_study', app_sequence=['consent', 'intro','your_prolific_study', 'questionnaires', 'payment_info'], num_demo_participants=1, completionlink='https://app.prolific.co/submissions/complete?cc=11111111',
),
]
```

For better generality, I'd recommend assigning your completionlink variable to a javascript variable.

3. [In init for your last app, configure a completion link for your very last page]{style="color:green"}


```r
class PaymentInfo(Page):
    form_model = 'player'

    def is_displayed(player):
        participant = player.participant
        return participant.consent == True

    @staticmethod              
    def js_vars(player):
        return dict(
            completionlink=player.subsession.session.config['completionlink']
        )
    pass
```


4.  [Pass submission link to participants in your targeted html template
    payment page using]{style="color:green"}


```r
<script>
    let completionlink = js_vars.completionlink;

    window.onload = function(){
        window.location.href=completionlink;
    }
</script>
```

The is_display statement in 3 is useful so that only participants who consent will end up seeing your payment page. This is especially useful if you configure your consent app like below:

### Bonus: consent configuration 


```r
    @staticmethod       # populates a participant variable with the respondent's consent status (for use across apps)
    def before_next_page(player: Player, timeout_happened):
        participant = player.participant
        participant.consent = player.consent

    @staticmethod       # sends nonconsenting participants to the last app
    def app_after_this_page(player, upcoming_apps):
        if not player.consent:
            return upcoming_apps[-1]
```

note: for this consent piping to work, you'll also need to go to settings.py and set 

```r
PARTICIPANT_FIELDS =['consent']
```


# Heroku Side

https://github.com/oTree-org/otree-docs/blob/143a6ab7b61d54ec2be1a8bc09515d78e0b07c71/source/server/heroku.rst#heroku-setup-option-2

There are a few ways to connect to heroku -- otreehub, github (through
Heroku), and the command shell (uses git) - otreehub is currently the
recommended method, but requires a subscription fee for \>2 projects. In
fact, oTree hub doesn't recommend that you ever delete an app:

["After you finish a study, don't delete your site from Heroku, because
oTree Hub's registration keys are not reusable. Instead, you should keep
your site so that you can run your next study on it. You can deploy new
code and even change the URL name. There is no need to create a new
site".]{style="color:red"}

During learning/testing, I find that I end up going through a few
different versions, so I describe the command shell and GitHub methods.

<!-- If you've made many changes to your survey, I'd recommend deleting your -->

<!-- app and re-starting, to ensure that the database is fresh, and you're -->

<!-- running the latest version of otree. -->

## Command shell method

<details>

<summary> **Click to expand** </summary>

1.  [Downloading software]{style="color:green"}

    -   [Download heroku CLI]{style="color:green"}

        -   [[https://devcenter.heroku.com/articles/heroku-cli]{style="color:green"}](https://devcenter.heroku.com/articles/heroku-cli){.uri}
        -   connects computer to Heroku

    -   [Git]{style="color:green"}

        -   [[https://git-scm.com/downloads]{style="color:green"}](https://git-scm.com/downloads){.uri}
        -   [Download default settings]{style="color:green"}

2.  [Create Heroku account]{style="color:green"}

    -   [Click create new app]{style="color:green"}

    -   [Create app name (unique)]{style="color:green"}

    -   [Under resources tab]{style="color:green"}

        -   NOTE: these are paid options, so you should consider sticking to hobby options when first testing

            -   [Add postgres]{style="color:green"}

                -   [Select hobby basic]{style="color:green"}

                    -   [Hobby basic is free, but limited to \~20 participants at a time, and can store plenty of data. RESULT: limited ability to run a study with many participants]{style="color:red"}

                        -   Proper postgres/resource configuration will be discussed in implementation section

3.  [Open powershell on main otree folder]{style="color:green"}

    -   i.e., where you could run your "otree devserver" command

4.  [commands]{style="color:green"}


```r
Heroku login
# Press any key to open browser

Git init
#makes folder
Git add.
#add files into folder

Git commit
#Put in email and name if prompted

Heroku <git:remote> -a name_of_your_heroku_app

Git push Heroku main

# uploads to Heroku

Heroku run "otree resetdb"

#prepares database to work in otree *needs to be done everytime changes are made to models.py*)
```

[**After any otree changes**]{style="color:green"}


```r
Git commit -am "your commit message"

Git push Heroku main

Heroku run "otree resetdb"
```

<br>

5.  [Test on Heroku]{style="color:green"}

-   [Navigate to Heroku]{style="color:green"}

    -   [Click open app]{style="color:green"}

    -   [Click your_app]{style="color:green"}

        -   [Need security features (e.g. so that participants cannot
            download data)]{style="color:red"}

6.  [Navigate to Powershell to configure security]{style="color:green"}


```r
Heroku config:set OTREE_AUTH_LEVEL=DEMO
#Controls what a user without password can do, In this case, demo

Heroku config:set OTREE_ADMIN_PASSWORD=mypassword
#When accessing admin interface -- require mypassword

OTREE_PRODUCTION=0
#Takes value 1 or 0; If an error, toggles whether participants will see a brief (1) or detailed error message
# left 0 because a full-page error is thrown otherwise
```

6.  Create a session

-   [Click the 'Rooms' tab in your link]{style="color:green"}

-   [select "close this room" if a session is active, then restart]{style="color:green"}
    
-   [when creating a new session, select the Session Config that you wish to test]{style="color:green"}

-   [input the targeted number of participants for that session]{style="color:green"}

-   [select "create"]{style="color:green"}

</details>

<br>

## GitHub method

<details open>

<summary> **Click to expand** </summary>

1.  [Go to your Heroku account, personal link]{style="color:green"}

2.  [Click "new", then "create new app"]{style="color:green"}

3.  [Create a (unique) app name]{style="color:green"}

4.  [Click deploy]{style="color:green"}

5.  [Select GitHub as your deployment method]{style="color:green"}

6.  [For "app connected to GitHub" slect your account and input the name
    of your app]{style="color:green"}

7.  [Enable automatic deploys]{style="color:green"}

8.  [Select the branch you'd like to use for your app, and click
    'deploy']{style="color:green"}

9.  [Click resources tab]{style="color:green"}

    -   [NOTE: these are paid options, so you should consider sticking
        to hobby options when first testing]{style="color:orange"}
    -   [Heroku charges by the second, so monthly fees can be smaller
        than they appear]{style="color:red"}

10. [Add postgres]{style="color:green"}

    -   [For your purposes: look at connection limit]{style="color:orange"}

    -   [Standard 0 is often fine, but look at other options (e.g., Standard 2 or premium 0, premium 2 based on your needs -- better safe than sorry)]{style="color:green"}

11. [Add redis]{style="color:green"}

12. [Define dyno]{style="color:green"}

    -   [Set as professional, and maybe add 1 dyno for web, Standard should be enough for most experiments]{style="color:green"}

    -   Impacts how long it takes to create sessions

13. [Click settings tab, click  'reveal configs' button]{style="color:green"} (https://otree.readthedocs.io/en/latest/admin.html?highlight=OTREE_AUTH_LEVEL#password-protection)

    -   [in 'key' field, type 'OTREE_AUTH_LEVEL'; in 'value' field, type 'STUDY'; click 'add']{style="color:green"}
    
        - Controls what a user without password can do
        
        -If you are launching an experiment and want visitors to only be able to play your app if you provided them with a start link, set the environment variable OTREE_AUTH_LEVEL to STUDY.
        
        - To put your site online in public demo mode where anybody can play a demo version of your game (but not access the full admin interface), set OTREE_AUTH_LEVEL to DEMO.
        
    -   [OTREE_ADMIN_PASSWORD=mypassword]{style="color:green"}
    
        - When accessing admin interface -- require mypassword; prevents participants from accessing sensitive info
        
    -   [OTREE_PRODUCTION=1]{style="color:green"}
    
        - Disables debug mode
    


</details>

<br>

Your resulting link will look like:
<https://your_heroku_app.herokuapp.com/room/your_prolific_study>



## Further testing of Heroku app

<br> 1. [Navigate to Heroku]{style="color:orange"}

-   [Click open app]{style="color:orange"}

2.  [Enter rooms tab again, create session with target config]{style="color:orange"}

-   [Double check that you've removed the participant label file -- otherwise you hit a label input page]{style="color:orange"}

-   [If you want to test how the participant label is handled, keep a participant label file, and add some participant labels]{style="color:orange"}

    -   [Then, you can use a url that specifies a valid participant label using the `?participant_label=`argument]{style="color:orange"}

        -   E.g. <https://your_heroku_app.herokuapp.com/room/your_prolific_study?participant_label=id_prolific_test>

        -   Or anything, so long as it's in your participant label file

# Prolific Side

<br> 

After the past two sections, you should have the means to *track Prolific URLs*, *administer completion links*, and *host your study outside of your local server*. The next major step is to make this hosted URL available and compatible for administration on Prolific.

If you [open a new study in Prolific]{style="color:green"}, their
interface is very user-friendly in this respect

1.  [In the section "How to record Prolific IDs," paste in your Heroku room url, and]{style="color:green"}

2.  [select "I'll use URL parameters]{style="color:green"}

<br> <br>

> ![](record_prolific_id.png)

<br> <br> 

[Do note, that the default url parameter in prolific is PROLIFIC_PID, and these instructions use participant_label, so you would have to:]{style="color:red"}

3.  [change the url parameters in prolific accordingly if following this guide verbatim]{style="color:green"}

-   (I kept participant_label because it passes in otree by default to participant.label, and I can't be bothered to work out the translation)

link provided through Prolific by default (do not use):
<https://your_heroku_app.herokuapp.com/room/your_prolific_study?PROLIFIC_PID=>{{%/*PROLIFIC_PID*/%}}

link used in this outline:
<https://your_heroku_app.herokuapp.com/room/your_prolific_study?participant_label=>{{%/*PROLIFIC_PID*/%}}

If you are using a URL redirect for study completion, prolific will generate a new one for you.

4.  [Copy the generated URL and replace the input for completionlink in session_config]{style="color:green"}

-   E.g. '<https://app.prolific.co/submissions/complete?cc=1A1A1111>'

[If you are using multiple sessions/dyadic games, maybe keep the SESSION_ID parameter (I'm not sure)]{style="color:red"} <br> <br>

## [Testing prolific URL]{style="color:orange"}

[After configuring your URL (assuming functioning and open Heroku
app),]{style="color:orange"}

1.  [save the draft study,]{style="color:orange"}

2.  [preview it]{style="color:orange"}

<br> 

> ![](prolific_preview.png){width="75%"}

<br>

3.  [Click on open study link]{style="color:orange"}

<br>

> ![](open_study.png){width="50%"}

<br> 

4. [Click through until the page that records participant_label]{style="color:orange"}

5.  [In the admin page, check back on the room, and you should see in the recorded data (in this case, variable prolific_id, is filled with a string, passed PROLIFIC_PID in prolific to participant_label]{style="color:orange"} 

<br> <br>

> ![](prolific_id.jpg)

<br> 

# Final Implementation

<br>

After you first deploy an otree app on Heroku, it will remain on the original oTree version upon deployment, even after redeploying, or restarting dynos, redis, and postgres.

<!-- If you've made many changes to your survey, I'd recommend deleting your -->

<!-- app and re-starting, to ensure that the database is fresh, and you're -->

<!-- running the latest version of otree. -->

[Double checks]{style="color:orange"}

1.  [password protect the admin. (prevents participants from accessing sensitive data)]{style="color:green"}    <https://otree.readthedocs.io/en/latest/server/heroku.html#server-performance>
    
2. [Disable debug (set OTREE_PRODUCTION server-side).]{style="color:green"}

[If you add any variables after the first deploy to Heroku (e.g., as a result of testing), trying to open your app will yield a "column not found error" -- to resolve, detach and re-attach redis and postgres -- deletes data -- JUST MAKE SURE YOU DOWNLOAD ANY DATA YOU NEED]{style="color:red"}

-   My guess for the issue, the SQL call isn't setup for new variables after the first deployment After the app is created (even after, the redis/postgres refresh), Heroku will run the otree version that was used on the first installation/deployment

<br> <br> **Voila! you should be good to go, please use your new-found computer powers for good**

<br> <br>
![](C:/Users/dalla/Google Drive/Class media/hackit.jpg){width="50%"}


<br> <br>

# Other Info

<br>

## On participant labels

Usually, if you are using participant labels, you need a participant_label_file which is a relative (or absolute) path to a text file with the participant labels. It should contain one participant label per line.

## On oTree Hub

-   Otreehub

    -   Easy, quick
    -   Requires publicizing code
    -   Only two slots available

-   Github

    -   Github desktop  add local depository  add file destination
    -   Heroku  connect to github
    -   Add repository name, search, and connect
    -   Go to Heroku website, should see it connects
    -   easy

-   Command prompt/terminal

    -   Full control through advanced coding

## On number of participants

"in praxis oTree's session size is limited by the fact that oTree does not utilise multiple CPU cores" If participants play at different times then that will be better. Think of it like road traffic. Congestion happens when everyone plays at once. Using multiple rooms or multiple sessions doesn't make a difference.

oTree Hub also has a performance analysis feature which can be very useful in troubleshooting slowdowns.

## Session overload

The most demanding sessions are the ones with a combination of (1) many rounds, (2) players spending just a few seconds on each page, and (3) many players playing concurrently, because these sessions have a high number of page requests per second, which can overload the server. Consider adapting these games to use Live pages, which will result in much faster performance. Live pages use the web dyno.

• With the higher dyno tiers, Heroku provides a "Metrics" tab. Look at "Dyno load". If users are experiencing slow page load times and your your dyno load stays above 1, then you should get a faster dyno. (But don't run more than 1 web dyno.)

• If your dyno load stays under 1 but page load times are still slow, the bottleneck might be something else like your Postgres database.

## Differences from Mturk

1.  From oTree's admin interface, you publish your session to MTurk.

2.  Workers on Mechanical Turk participate in your session.

3.  From oTree's admin interface, you send each participant their
    participation fee and bonus (payoff).

## Browser bots

<https://otree.readthedocs.io/en/latest/bots.html> Browser bots are
useful for stress-testing an app

## Rooms and sessions

<https://otree.readthedocs.io/ja/latest/rooms.html#rooms>)

Rooms are necessary for permanent links that are used in real studies

Rooms vs. sessions - If you want to prevent the same person from playing
twice, use rooms

Rooms provide:

1.  Links that you can assign to participants or lab computers, which
    stay constant across sessions

2.  A "waiting room" that lets you see which participants are currently
    waiting to start a session.

3.  Short links that are easy for participants to type, good for quick
    live demos.

-   Room URLs are designed to be reused across sessions

sessions: Participants can join after the session has been created, as
long as there are spots remaining.

## oTree studio

-   Otree Studio is recommended in the docs, but it's a quasi-text-based
    format.

    -   If I'm going to be dealing with a bunch of text commands anyway,
        I'd prefer to be more hands on (warning: greater need to
        understand python and Django languages)

-   Easier to do straight imports from forums and external sources

-   Otree studio has limited project slots \~2

-   Otree studio is public

o Projects cannot be imported into oTree Studio. 
<https://groups.google.com/g/otree/c/OSzij9_NdUM/m/ZfIPFkzZBgAJ>

## Number of database connections

<https://groups.google.com/g/otree/c/1aYV-JY5l2Y/m/_BiyB5WwBAAJ> - The
number of database connections is not related to the number of
participants. Rather, it is related to the number of server processes.
In the simplest setup, each process has 1 database connection. Now that
oTree runs in a single main process, it uses fewer connections than
before, when there were 3 processes that used connection pooling which
further complicates things.

-   In reality, there may be a few more connections active. For example
    there is actually a second process that runs a task queue. I haven't
    confirmed this but I would expect that the new version should only
    use like 5 or fewer connections even when there are many
    participants.

## Recording time spent on page

Knowing how much time participants spend on each page is very useful for many reasons:

1. Calibrating payment
2. Detecting problematic observations
3. Projecting how long a module of a study will be

My first attempt was to run Javascript on all of my pages, and kick the values back to player objects. This worked fine until I realized that page errors reset the timer. Clearly produces underestimates.

The folks at oTree have [written a script for this already](https://otree.readthedocs.io/en/latest/admin.html#export-data), and you can download it [directly here](https://otree.readthedocs.io/en/latest/_downloads/d4f9f0315d093284dedcf1c2e66e7c68/pagetimes.py):

https://groups.google.com/g/otree/c/NHBISjrnlxo/m/xEEDMKsQDQAJ



For the command python pagetimes.py infile.csv outfile.cs,

You run in the terminal.

All you need to do is:

1) Download the pagetimes.py file to your project directory

2) Download the "infile" .csv file from oTree's "data' tab (at the bottom of the screen) to an appropriate location

3) run the command in your terminal

As best practice, I wouldn't download the file to your project directory, in case it's a public repository. For instance, I might do the following:
python pagetimes.py C:\Users\dalla\Desktop\infile.csv C:\Users\dalla\Desktop\outfile.csv

## Other resources

<https://learnxinyminutes.com/docs/python/>
<https://github.com/TiesdeKok/LearnPythonforResearch>
<https://djangobook.com/course/introduction/>
