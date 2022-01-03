

# The ML Project Lifecycle
## V1: Welcome

A deployment scenario...
* take photo on camera
* send pic to prediction server via api
  - either cloud or edge
* make prediction (or issue error)
* send prediction (or error) back to phone

Deployment set might differ from training set
* concept drift
  - thoughtful data augmentation
  - deployment surveillance (i.e., track various input stats to monitor for drifts)

ML in production
* takes a lot more than just ML code
* Project code
  - Small "PoC" part
    * ML Model Code:  `X -> Y`
  - The rest of it: PoC-to-Production Gap; includes pipelines/infrastructure for:
    * configuration
    * data collection (capture, storage, retrieval, etc)
    * data verification 
    * feature extraction
    * machine resource mgmt
    * process mgmt tools
    * serving infrastructure (kubernetes, docker, APIs, user experience, etc)
    * monitoring (traditional)
    * monitoring (concept drift)
*  Check out "Hidden technical debt in machine learning systems" (2015) paper




## V2: ML Project Steps

ML Project Lifecycle
* Scoping 
  - define project
  - what is `X` and `Y`
* Data
  - define what the data is
  - establish baseline
  - label and organize data
* Modeling
  - select and train model(s)
  - perform error analysis
  - go back to "Data" step if necessary
* Deployment (first time)
  - deploy in production (extra software necessary)
  - monitor and maintain system (e.g., track input distribution)
  - go back to "Modeling" step if necessary
  - go back to "Data" step if necessary
* Deployment (ongoing)
  - similar to the first time, but hopefully fewer "lessons learned" over time





## V3: Speech Recognition (case study)

Speech recognition is an area where DL has really been impressive (think Alexa, Siri, etc).

### Scoping Step
* Define project 
  - decision to work on speech rec for voice search
* Decide on key metrics
  - try to estimate/guestimate what these will be for your project
  - for this case study:  
    * accuracy
    * latency - how long does it take system to process/transcribe speech
    * throughput - how many queries/sec can the system handle
* Estimate resources and timeline
  - typical business-related project activities
  - is the estimated cost and time worth it to the business?

### Data Step
* Define data
* Establish data baseline
* Label data
* Organize data
* Challenges/Decisions (data definition questions)
  - is the data labeled consistently?
  - How much silence before/after each audio clip? (100ms? 300ms? 500ms?)
  - How to perform volume normalization?

##### Example
An audio clip is played with the words: "Um" - "today's" - "weather"

How should it be transcribed?  For example, maybe like `Um, today's weather.` or 
like `Um... today's weather.`  Or should we consider `um` to be noise and just transcribe
it as `Today's weather`?  

Ng says consistency is most important and that he wouldn't consider `um` to be noise.  He
would choose the first or second option, then make sure any/all data transcribers know the
rule.

It's important to consider and develop frameworks to ensure consistent,
high-quality data such that the system has a chance of working in production. An important
implication here is that such systems should be carefully considered before even the
training data is collected or labeled so that the training dataset matches the 
deployment settings as much as possible.


### Modeling Step
* Select and train model(s)
  - code: algorithm, model
  - hyperparameters
* Perform error analyses

```
# Often Ng uses the shorthand
ML System = Code + Data

# But more realistically, he said you might consider it
ML System = Code + Hyperparameters + Data
```

##### Model-Centric Optimization
Often in academic research, **the data is held fixed** while the researchers vary
the code (alg, model) and/or hyperparameters to try to improve performance.  

##### Data-Centric Optimization
In contrast,
for a lot of product teams where the goal is to build and deploy a valuable ML system,
Ng finds that production teams often find it better to **hold the code fixed** while
varying the data and perhaps the hyperparameters.

Error analysis can help define exactly what type of pain points exist and
what types of data to collect (instead of a "just collect more data" approach).

Also, consider that a data-centric view also means one might use domain expertise 
to consider data representations that will help exploit useful patterns (e.g., instead
of a NN having to learn to look for certain patterns that are characterized by particular
frequencies, it might help to input the Fourier spectrum instead of or in addition to
the time series data).

### Deployment
* Deploy in production
* Monitor and maintain system

##### Deploy in Production
Example: Deployment Components needed for Mobile Phone (Edge Device) Speech Rec App 
* Local Software
  - Microphone --> VAD module (selects out just voice audio)
* Prediction Server (Edge or CLoud)
  - Input: MobilePhone(VAD module output) --> Speech API --> Prediction Server
  - Outputs: Prediction Server --> (Transcript, Search Results) --> MobilePhone(Frontend Code)

##### Monitor and Maintain
Data Drift - Ng's speech rec sys trained on older voices, then didn't work on the 
uptake population, which was all younger people.  Needed some retraining.




## V4: Course Outline

It's good to start from the deployment perspective, then work backwards when planning out
the ML Project steps (before doing anything).  The course is outlined this way.

1. Deployment
2. Modeling
3. Data
4. Scoping



---------------------------------------------------------------




# Deployment


## Key Challenges

### Concept Drift & Data Drift
What if the data changes after the system has been deployed?

Example: lighting changes (photos), voice types (speech)

#### Example

* Training set
  - purchased data
  - historical user data w/ transcripts
* Test
  - data from a few months ago
* Deployment
  - has the data changed?
    * e.g., new model of smartphone / different microphone
  - are the changes **gradual**?
    * e.g., human language trends
  - are the changes **sudden and disruptive**?
    * e.g., C19 caused many behavioral changes in credit card purchases, which
      shocked various anti-fraud systems

#### Data Drift vs Concept Drift
housing cost: X is size, Y is cost of house

* Data Drift:  Changes in distribution of features.
  - e.g., people start building much bigger houses over time
  - deployment data input distribution differs from training/test distributions
* Concept Drift:  The mapping changes (X -> Y)
  - def of Y given x changes
  - e.g., a disruptive event causes Y to shift (more expensive) for the same X
  - deployment data output distribution differs from taining/test distributions


### Software Engineering Issues

Checklist of questions
* Real-time or Batch predictions
  - Batch example:  business doesn't need results from today's data until time (nightly batch processing)
  - Real-time example:  user inputs speech and expects immediate response
* Cloud vs Edge/Browser
  - Cloud has any type of resources required, but may induce latency
  - Edge has limited resources but may be necessary for quick computation on certain events; 
    is available without internet
* Compute resources available in deployment (GPU, CPU, memory, etc)
  - training and test environment can be on a high-powered GPU, but you also need to
    know how it will work in the deployment setting
* Latency, Throughput (QPS - Queries per Second)
  - deployment setting defines what's necessary
* Logging
  - do you want to save data for later training/retraining use
  - what do you want to log?
* Security and Privacy
  - are there any regulatory requirements?

Note that the practices during the very first deployment will likely include
as much work as initially developing the ML code and initial system.  General
monitoring and system maintenance thereafter should likely be less dramatic.

Next, we will look at common deployment/design patters for ML systems.

-------------------

## Review
* the steps of the ML project lifecycle include:
  - Scoping
  - Data (Definition, Etc)
  - Modeling
  - Deployment
* Edge vs Cloud
  - Edge could provide lower latency, does not have network bandwith requirements, and can
    function when the network is slow or down
  - Cloud can provide more computational power, etc
* Data used should be as consistent as possible
  - Speech recognition example:  "Um, xyz" and "Um... xyz" are both ok, but only one 
    format should be chosen and used for training.
* Once deployed, it is important to monitor for data drift and concept drift, and to
  "maintain" the system's performance by doing what's necessary (e.g., retraining)
* The ML system/project lifecycle is an iterative process where at any later stages we might
  have to return to earlier stages in the lifecycle

-------------------------

## Deployment Patterns


Common Deployment Scenarios
* New product/capability
  - start w/ a small amount of traffic, monitor, then ramp up
* Automate/assist w/ manual task 
  - shadow mode deployment (compare automation w/ ongoing manual tasks)
* Replace previous ML system (w/ better one!)

Key Ideas
* gradual ramp up w/ monitoring
* rollback
  - if monitoring shows degraded performance, rollback to previous system (if available)


#### Visual Inspection Example

* Shadow Mode Deployment
  - have ML system run in parallel with human monitoring/judgment
    * human inspection continues uninterupted on entire incoming dataset
    * ML prediction runs on entire dataset, or a smaller subset
  - ML system output not used for any decisions during this phase
  - point: gather data on how model is performing in deployment before 
    using it in any mission-critical way (i.e., test
    deployment data against the human inspection data as was done in training/test
    set)
* Canary Deployment
  - roll out to small portion of traffic (e.g.,, 5%) to the new prediction model
    * in distinction to SMD, the model/algorithm impacts/influences downstream decision-making on this subset
  - monitor activity/consequences/results wrt those from ongoing human inspection 
  - gradually ramp up traffic to model (away from human inspection)
  - continue monitoring
* Blue-Green Deployment
  - Blue (Old) prediction system vs Green (New) prediction system
  - Inputs go through router
  - Router initially routes inputs to Blue (Old) system
  - When ready, one switches the routers output to the Green (New) system
  - Monitor the Green system
  - Easy rollback to the Blue system if necessary
 
 
Note that these deployment patterns are not necessarily incongruent or orthogonal to
each other. For example, the BGD's system switch can be gradual (it's not necessarily 
"all at once"); thus, the BGD pattern is congruent with the CD deployment 
pattern - not distinct or independent of it. In the CD pattern, the BGD router is used 
to switch from routing "everything to only
the manual/human system" to routing "mostly everything to the manual/human system and
the remaining scraps to the ML system." Here, the Blue (Old) System is the manual 
"human inspection" system, while the Green (New) system is the project's first ML
system. The routing technique might be called "proper routing" to emphasize that
the Green System and Blue System receive nonoverlapping "flow subsets," the total of 
which compose the total router flow.

The BGD pattern, then, is essentially a generalization of the CD pattern that does not
specify the new or old systems exactly. However, this generalization is only a "semi-generalization"
in that the BGD pattern also emphasizes the use of a well-defined, prediction-system-independent 
"router" for switching between prediction systems to allow for easy rollback; i.e.,  the
prediction systems should be considered as modular endpoints of a data processing pipeline,
which the pipe can route its "flow" into.

Finally, the SMD pattern fits into the BGD pattern as well if we allow for "copycat routing"
to be an allowable routing style of the BGD pattern.  In the SMD pattern, the BGD router is 
used to switch from routing "everything to only the manual/human system" to routing 
"everything to the manual/human system as well as everything (or a subset) to the ML system." Here, 
like in the CD setup, the Blue (Old) System is interpreted as the manual 
"human inspection" system, while the Green (New) system is the project's first ML 
system.  The routing technique is different than the "proper routing" technique specified in the
CD design; I would call it "copycat routing" to emphasize that 
the Green System receives a copy (or subset copy) of what is sent to the Blue System (in distinction
of "proper routing" which physically Y-splits the flow out of the router).






Shadow Mode
```
      /======== Human
====<
      \======== ML
```

Canary Mode
```
      /----95%--- Human
====<
      \----5%---- ML
```


#### Degrees of Automation
The above patterns can give a sense that there is an either/or requirement, but
one should also think about the degrees of automation too.  Above, it might have
had this feeling that the systems were "fully automated."  

* Human only
* Shadow mode
* AI assistance
* Partial automation
* Full automation

AI assistance and partial automation designs are known as human-in-the-loop designs.

AI assistance is when a task is assisted by some AI automation.  For example, if looking for
cracks in phone screens, the AI might assign a probability which can be used to prioritize
cases for humans to inspect; or the AI could draw a bounding box where it thinks a defect is
to help a human focus on where to look.

Partial automation is similar to assistance, but generally means the AI has more decision-making
power.  For example, it may include a probability threshold considered to be "super low probabability,"
and route only probabilities higher than this to humans for further inspection; or in the bounding box
example, the AI might only send the bounding-box-cropped image to the human (deciding that nothing else
in the image need be inspected).






------------------------------------

## Monitoring

Use a monitoring dashboard.
* Server load
* Non-null outputs
  - is system no longer working right?
* Fraction of missing input values
  - has something changed about the data?

Brainstorm metrics/statistics that should be tracked.
* start out with a lot
* gradually removes ones you find over time to not be too useful

Metrics
* Software metrics (many mlops tools have these out-of-the-box)
  - memory
  - compute
  - latency
  - throughput
  - server load
* Input Metrics (has input distro X changed)
  - avg input length (e.g., duration for speech or EEG signal)
  - avg input amplitude (e.g., volume for speech, or image brightness for images)
  - number of missing values (e.g., for structured data)
* Output Metrics
  - how often is a null result returned (e.g., the empty string for speech rec, or class indecision, etc)
  - how often does the user do very quick searches in a row with substantially same input 
  - number of times user tries to use ML system then switches to a more manual option
  - output distributions

Most input/output metrics must be custom-made.

Just as ML modeling is iterative, so is deployment. 
```
# ML training/test model
   ----->  ML model/data  ----->
  ^                             \ 
 /                               V
 <-- Error Analysis <--  Experiment

# ML deployment model/monitoring
   -->  Deployment/Monitoring  -->
  ^                               \ 
 /                                 V
 <-- Performance Analysis <--  Traffic
```

The iterative process in deployment can help inform what metrics aren't
that useful as well as metrics that need to be monitored, which you didn't
first consider.

Each metric being monitored should typically have a trigger, such as an alarm (or
automated process, such as a retraining session, etc).
* Set threshold for metrics
* Adapt metrics and thresholds over time

Manual retraining is far more common that automatic retraining. Developers are
typically reluctant to allow this process to be fully automated, but it is 
done in some circustances/industries.


## Pipeline Monitoring

We often depict a ML system too simply:
```
Input -> Model -> Output
```

For example, in speech recognition:
```
Audio -> Speech Recognition Model -> Transcript
```

However, in practice, this picture is too simple to be very useful.  For example,
in speech recognition on a mobile device, the diagram is more accurately described like:
```
Audio -> VAD (Voice Activity Detection Module) -> Speech Recognition Model -> Transcript
```

Here, the VAD first assess the probability that a given audio clip even has any voice/speech 
going on.  This is important for many reasons, e.g., if the speech model is cloud-based, the VAD
limits how many audio clips must be transmitted back and forth over the network.  This also limits
the deployment inputs to those that "should look like the training/test data," like Roozbeh and 
I were doing with the "walk-like detection" module in our PD detection model.

Note that this ML pipeline depicts a system made up of two learning algorithms:
* the system of interest
* a preliminary filtering system

Changes to either system can affect performance.

For example, different cellphones might have different
onboard VADs that are better or worse, etc.

For example, one might develop "User Profiles" that are built up for input into a recommender
system.  These user profiles might depend on their own feature predictions based on observed
data (e.g., does this user likely own a car?).  

Want to monitor the same types of metrics for the filtering models as for the primary ML system
* Software metrics
* Input metrics
* Output metrics


How quicly do input distributions change?
* User data often has a slow drift overall 
* External events can cause sudden changes (e.g., C19)




====================================================================================



# Suggested Reading Materials

## Machine Learning in Production (Evidently.AI)

[Machine Learning in Production: Why You Should Care About Data and Concept Drift](https://towardsdatascience.com/machine-learning-in-production-why-you-should-care-about-data-and-concept-drift-d96d0bc907fb)

* Model Decay
  - aka: model drift, model staleness
  - primary gist: "Past performance is no guarantee of future results."
  - secondary gist: "No model lives forever, but the speed of decay varies."
  - Changes may be detected in 
    * model quality/performance metrics 
      - e.g., accuracy, mean error rate
    * downstream business KPIs
      - e.g., click-through rate
  - Rate of decay is situation/scenario dependent
    * Years, e.g., a computer vision or language model 
    * Days, e.g., maybe a stock market model
    * Domains, e.g., general HAR model on a unique subpopulation (or on an individual basis)
  - Rule out (differential diagnosis): data quality, system update errors, etc

### Data Drift (Changes in Feature/Population Distributions)
* **Data Drift**
  - aka: feature drift, population shift, covariate shift
  - primary gist: the input data has changed (resulting in a meaningfully different distribution of variable)
  - primary problem: the model is not trained for this new situation!
    * still works on the old training, val, and test sets (obviously)
    * possibly even worked for a while in production
    * however, no longer working well in production
  - Example: Advertising Cohorts
    * company attracts new customers through advertising
    * company initially attracts customers through "paid search" (I think, e.g., Google search ads)
    * company builds a model on a cohort of "paid search" records
    * company starts new ad campaign on Facebook
    * at first, model still fares pretty well since most clients still coming from paid search
    * however, as the FB ad campaign goes on, more and more clients start coming from FB
    * the model's performance dramatically reduces
    * it is found that the gradual decay in performance is highly correlated with class distribution
      changes in the feature `source_channel`, as well as related vars such as `device` 
    * going forward: the company finds similar performance drops when targeting new geographical
      areas and demographics
    * **All of this could have been detected early if the company was closely monitoring the model!**
  - My Own Example: human activity recognition (HAR)
    * HAR model is trained on healthy males aged 15-35
    * HAR model works well early when company's demographic primarily young adult males
    * company generalizes product and extends offers to older males 
    * HAR model's performance seen to drop since the older males move and behave in meaningfully
      different ways
    * HAR model is retrained and found to work well again on company's user base
    * company wants a successful Q3, so begins targeting sales at their untapped demographic - females
    * HAR model's performance seen to drop again
    * etc!
  - Note on Data Quality
    * sometimes a "data drift" can occur if the incoming data quality changes
    * example in HAR: sensor degradation
    * "data quality" issues are most often associated with "training/deployment skew" (see below)
  - Solutions
    * Retrain the model using a deployment-faithful population
    * Rebuild the model for the new population segments
* **Training-Serving Skew**
  - aka: Training-Deployment Skew 
  - note: 
    * sometimes mixed in with or even called "data drift"
    * however "skew" refers to an actual mismatch between the training data and the data seen in deployment 
    * thus, "skew" is not the same phenomenon as "drift" (like that described above)
    * skew and drift arise from fundamentally different root causes
  - primary gist: the training/validation population data used to build the model is significantly different 
    than the population that the model is "served" to (i.e., the deployment population)
  - common causes: overly simplified training data
    * e.g., in the HAR work I do, this is the reason we harp on the important distinctions
      between in-lab data (typically a guided/supervised collection of a small/finite 
      set of simple, somewhat-artificially "orthogonal" activities that are cleanly/clearly recorded
      in a setting with little environmental variation, challenge, etc; "closed world") 
      vs out-in-the-wild data (a nigh-infinite set of
      typically unguided, composite/co-occurring activities ranging over an array of environmental
      conditions and settings, recording qualities, etc; "open world")
  - other examples given in article
    * invoice classification model
      - trained on a limited set of crowdsourced invoice images, where
        the handwriting is very clear/clean, image quality is great, and 
        people fill out the form blanks in the right spots, etc
      - deployed on real world data where the handwriting can often be sloppy,
        the image quality varies greatly, and people do not always fill out
        the forms so perfectly
    * Google Health retinopathy classification model
      - trained on high-quality, well-lit images
      - deployed on dataset of varying image quality and lighting conditions

#### A Few More Thoughts on Data Drift
The author doesn't dig into "gradual" vs "sudden" data drifts, though they do
so for concept drifts (below).

In their example, the gradual change in the `source_channel` classes from mostly
"paid search" to mostly "Facebook" is an example of a gradual data drift.  

This "gradual data drift" could
have just as easily been portrayed as a "sudden data drift." For example,
consider a company that builds a model for LinkedIn, which works well in
production until it begins getting run for ad campaigns on TikTok as well several months
later.  Considering the change in demographics here, I'd expect the data drift to be almost 
immediately noticeable.  This might be considered a sudden data drift!

Note that one might actually consider both of these examples a training/deployment
skew problem, which the author tries to distinguish from "drift".  I now see why
the author says that many people also refer to "training/deployment skew" as a 
type of "data drift" -- seems they even do it in their own article!  And, actually,
I think that's ok: it's better to have a term that generally refers to changes in 
P(X) in general.  I'm fine with simply considering training/deployment skew as a 
subtype of data drift instead of in a category of its own.  The above example
shows that the distinction between general "drift" and the specific case of 
"skew" isn't actually as clear as their section on skew makes it out to be.  

In the above example, P(X) changed because the source of X was deliberately
altered (from just `source_channel={"paid search"}` to
`source_channel={"paid search"|"Facebook"}`); in turn, this change 
then gradually affected the rest of the feature space, X, over time.

One can also imagine data drift occurring even if they maintained a 
static `source_channel` feature.  They give some examples of this: targeting
new geographic regions or demographics.  These specific examples, though,
are still examples of "intentional" or "deliberate" sources of data drift.  One
can imagine the opposite too: "unintentional" sources of data drift stemming
from some world event.  Such an unintentional source of data drift can also be
further classified as "sudden" or "gradual".  


The author does invoke unintentional sources of drift like this, but only in
the section on concept drift.  

Take their gradual concept example where a competitor launches a new product.
* If the model is a sales forecast, then the customers now have another choice,
  thus `P(sale|X)` might indeed change and this is a real concept drift that has
  changed `X->Y`; their population might change in size but may remain statistically
  similar.
* However, assume the competitor product is targeted at a specific subpopulation (e.g., 
  women); here again, the population size might noticeably change, but so too will
  the feature distributions (e.g., sex, weight, height, preferences, etc). In this 
  case, there is definitely concept drift (and ultimately data drfit) for the subpopulation 
  of women, but no likely concept or data drift in the subpopulation
  of men.  Here, the sales forecasts would suffer from the concept drift in women, but also
  the data drift in the population as a whole.  An edge case can help clarify: assume all
  women use the new competitor product, such that the company no longer attracts any 
  new women customers; the update to this model to correct for concept drift is as 
  simple as hard-coding P(sale|woman)=0; however, the model's performance would still
  likely change (for better or worse) assuming that P(sale|man) has always been
  associated with a different level of uncertainty than P(sale|woman), e.g., if the
  uncertainty in P(sale|man) is 10xP(sale|woman), then the new forecasts will now
  be 10x more uncertain in general.  This degradation of performance is from the 
  remaining data drift that occurred.  (Right?!?! :-p) 

I think what I'm pointing out here is that concept drift (i.e., changes in P(Y|X)) is 
often, but now always, associated with changes P(X) as well (i.e., data drift).  The 
distinction is only slight and, to my mind, seems to be dependent on the specifics of the data
and the model being discussed.  The author even says at the end of the article:
"In practice, the semantic distinction makes little difference. More often than not, 
the drift will be combined and subtle."

(This is really similar to the types of issues/lingo used in the transfer learning 
stuff I did a while back...and I guess for good reason - it is essentially in the realm
of domain adaptation.)

But, still, there are certainly instances of "unintentional data drift" (changes in P(X)) without 
concept drift (changes in P(Y|X)).

For example (I wrote this in a note on "Data Quality" above), a HAR model's performance
might change in step with sensor degradation on one's wearable (e.g., Fitbit or Apple
Watch).  At the individual level this is certainly true, but one can also imagine this
at the population level as well.  For example, say that 90% of all Apple Watch "N" sales
occurred within the first 3 months of release and that Apple Watch "N+1" is not released
for another 2 years. The demographic data here might not show any significant drifts
over these couple of years, but it's easy to imagine (true or not) that the sensors can
degrade quite a bit -- and with that degradation, HAR models might degrade as well.

Another example simply comes from aging.  The Rolling Stones have had a solid, dedicated
fanbase for decades.  Let's assume that the fanbase itself remained static over all those
decades and, thus, that some of the feature space is static, but not all: any part of the 
feature space directly or indirectly dependent on age will vary with age.  Thus, if the 
Rolling Stones had trained models in the 70s that predicted things like "concert preferences" 
(seating, standing room, etc) and optimal ticket prices, they would surely be losing a lot
of money in 2020.  This is a demographic change, but not exactly an intentional one.

Anyway...

To recap some of these distinctions:
* Data Drift
  - Deliberate/Intentional
    * Sudden
    * Gradual
  - Natural/Unintentional
    * Sudden
    * Gradual 

This naming scheme can be a bit misleading though, e.g., the training/deployment
skew problem is a a type of "sudden drift" (despite the author of the article wanting to
distinguish it as a "skew" instead of "drift"), but it could be confusing to call
it "deliberate" or "intentional" since the people responsible might not have known
anything about training/deployment skew and consider their mistake to be 
"whoopsie-daisy unintentional!"

But anyway, what naming scheme is perfert...?




### Concept Drift (Changes in the X->Y Relationships)
* Concept Drift     
  - primary gist: "Concept drift occurs when the patterns the model learned no longer hold."
    * i.e.: "In contrast to the data drift, the distributions (such as user demographics, 
      frequency of words, etc.) might even remain the same. Instead, the relationships between 
      the model inputs and outputs change."
    * i.e.: "the very meaning of what we are trying to predict evolves."
* Gradual Concept Drift
  - primary gist: "the world slowly changes and the model does not"   
  - given examples:
    * competitor launches a new product
      - consumers now have more choices
      - more choices lead to behavioral changes 
      - behavior changes lead to changes in sales, S(Y|X)
      - the sales forecast model, P(Y|X), must be updated/retrained to match the 
        evolution in S(Y|X)
    * macroeconomic conditions evolve
    * mechanical wear of equipment
      - "Under the same process parameters, the patterns are now slightly different. It 
        affects quality prediction models in manufacturing."
      - Interestingly, this sounds a lot like my "sensor degradation / HAR" example given above
        for gradual data drift (as a wearable sensor degrades, the quality of its data X degrades
        (which can be interpreted as a shift in the data distribution), which results in a degradation 
        of M(Y|X)==P(Y|X).....)
  - Solutions
    * For supervised models with a decent amount of historical data: 
      - Initial Estimates: "If we are building a supervised prediction model, we know the past ground 
        truth. We can train the model on older data and apply it to later periods. Then we can imitate 
        model retraining with different frequencies and measure how it impacts model quality."
      - Updated Estimates:  Make sure to monitor performance in deployment and to reassess retraining
        intervals, etc.
* Sudden Concept Drift         
  - primary gist: "the world suddenly changes and the model does not"
  - given examples:
    * covid 19
      - e.g., impacts on mobility, shopping patterns, demand forecasting, hospitalization forecasts, etc
      - even less obvious models were affected: e.g., computer vision models for predicting pneumonia
        from x-ray images
    * changes in the interest rate by the central bank
    * technical revamp of the product line
      - "Predictive maintenance becomes obsolete since modified equipment has new failure modes (or lack of those)."
    * Major update in the app interface
* Recurring Concept Drift
  - aka: Seasonality
    * one might also see it called: Seasonal Concept Drift, Deterministic Concept Drift
  - note: this is your more classic "seasonality" issue from time series modeling; the 
    author does not consider this a true example of "drift" or "model decay," but instead
    an example of a shitty model since "seasonality is a known modeling concept," should be
    anticipated by the modeler (if applicable), and that a "sound model" should be 
    able to "react to these patterns" by design
    * *"Weekends happen every week, and we don’t need an alert. Unless we see a new pattern, of course."*
  - Solutions
    - build seasonality into the model
    - "If needed, domain experts can help to add manual post-processing rules or corrective 
      coefficients on top of the model output."


### Dealing with Drift

* For a model that isn't extremely mission-critical or difficult to manually override:
  - can wait for more data to be collected
  - use expert rules or heuristics as a fallback strategy
* For models in general, try one or more data-oriented retraining strategies
  - Everything Unweighted:  "Retrain the model using all available data, both before and after the change."
  - Everything, but Weighted for Recency: "Use everything, but assign higher weights to the new data so that 
    model gives priority to the recent patterns."
  - Recent Only: "If enough new data is collected, we can simply drop the past."
* If a simple data-oriented retraining does not work
  - a more rigorous hyperparameter/architecture search might work, including (but not limited to):
    * domain adaptation strategies
    * model stacking/composition using both old and new models
    * new data sources
    * new architectures
  - a change of model scope or business process might be best
    * shorten the prediction horizon (e.g., from a week to a day)
    * increase frequency of model runs (e.g., from weekly to daily)
  



# [Monitoring Machine Learning Models in Production (Comprehensive Guide)](https://christophergs.com/machine%20learning/2020/03/14/how-to-monitor-machine-learning-models/)

* [ ] ToDo: Get back to this!



====================================================================================
  
  
Start: noon
End: 1240
Start: 1257
End: 306
Start: 440
End: 


  
====================================================================================




# Additional Reading Materials

I liked the suggested article from Evidently.AI, so I opened a few related articles and looked a little more into 
their open-source data drift tools.
* Article: [Evidently 0.0.1 Release: Open-Source Tool To Analyze Data Drift](https://evidentlyai.com/blog/evidently-001-open-source-tool-to-analyze-data-drift)
* GitHub: [https://github.com/evidentlyai/evidently](https://github.com/evidentlyai/evidently)


## ML Monitoring: What it is (Evidently.AI)
[ML Monitoring: What it is and How it Differs](https://evidentlyai.com/blog/machine-learning-monitoring-what-it-is-and-how-it-differs)


Most people only know ML up to deployment
```
                ________________________________
               |               |     _______    |
               V               V    |       V   |
| Data |__| Feature     |__| Model    |__| Model      |__| Model      |__| ???? |
| Prep |  | Engineering |  | Training |  | Evaluation |  | Deployment |  | ???? |
```


Why monitoring matters: "An ounce of prevention is worth a pound of cure."
```             ________________________________
               |               |     _______    |
               V               V    |       V   |
| Data |__| Feature     |__| Model    |__| Model      |__| Model      |__| Model   |
| Prep |  | Engineering |  | Training |  | Evaluation |  | Deployment |  | Serving |
                              ^              ^                              |
                              |______________|______________________________|
```

**Machine learning monitoring**: a practice of tracking and analyzing production model performance 
to ensure acceptable quality as defined by the use case. It provides early warnings on performance 
issues and helps diagnose their root cause to debug and resolve.

* Software Deployment Classics
  - "A deployed model is a software service, and we need to track the usual health metrics such 
    as latency, memory utilization, and uptime."
* ML Deployment Novelties
  - Data adds an extra layer of complexity: 
    * "It is not just the code we should worry about, 
      but also data quality and its dependencies."
    * "Is the world changing too fast? In machine learning monitoring, this abstract question 
      becomes applied. We watch out for data shifts and casually quantify the degree of change."
  - Models often fail silently:
    * "There are no 'bad gateways' or '404's. Despite the input data being odd, the system will likely return 
      the response. The individual prediction might seemingly make sense⁠—while being harmful, biased, or wrong."
  - The distinction between "good" and "bad" is not clear-cut:
    * "One accidental outlier does not mean the model went rogue and needs an urgent update. At the same 
      time, stable accuracy can also be misleading. Hiding behind an aggregate number, a model can quietly 
      fail on some critical data region."
  - Metrics are useless without context:
    * "Acceptable performance, model risks, and costs of errors vary across use cases." 
      - "In lending models, we care about fair outcomes." 
      - "In fraud detection, we barely tolerate false negatives." 
      - "With stock replenishment, ordering more might be better than less."
      -  "In marketing models, we would want to keep tabs on the premium segment performance."




## To retrain, or not to retrain? (Evidently.AI)
[To retrain, or not to retrain? Let's get analytical about ML model updates](https://evidentlyai.com/blog/retrain-or-not-retrain)

> * Is it time to retrain?
>   - First, how often should we usually retrain a given model? 
>     * We can get a ballpark of our retraining needs in advance by looking at the past speed of drift. 
>   - Second, should we retrain the model now? 
>     * How is our model doing in the present, and has anything meaningfully changed?
>   - Third, a bit more nuanced. Should we retrain, or should we update the model? 
>     * We can simply feed the new data in the old training pipeline. Or review everything, from 
>       feature engineering to the entire architecture. 




### Part 1: Define the retraining strategy in advance
**"Model Retraining"**: "Adding new data to the old training pipelines and running them once again."

- Check #1: How much data does the model need?
- Check #2: How quickly will the quality go down in production?
- Check #3: How often do you receive the new data?
  * what comes first: the new data or the model decay?
    - Say the model decays quicker than new data is reported/collected; solutions:
      * A cascade of models. 
        - "Is some data available earlier? E.g. some points of sales might deliver the data before the 
          end of the month. We can then create several models with different update schedules. "
      * A hybrid model ensemble. 
        - "We can combine models of different types to better combat the decay. For example, use 
          sales statistics and business rules as a baseline and then add machine learning on top 
          to correct the prediction. Rough rules might perform better towards the end of the period, 
          which would help maintain the overall performance."
      * Rebuild the model to make it more stable. 
        - "Back to training! We might be able to trade some digits of the test performance for slower degradation."
      * Adjust the expectations. 
        - "How critical is the model? We might accept reality and just go with it. Just don't forget to communicate 
          the now-expected performance metrics! The model will not live up to its all-star performance in the first 
          test week."
    - If data is reported much faster than model decay, choose your own schedule!
- Check #4: How often should you retrain?
  * What is potentially wrong with frequent retraining?
    - adds complexity
    - adds costs
    - can often be error-prone
- Check #5: Should you drop the old data?
  * Dropping the old data makes the model worse. 
    - "Okay, let's keep it all for now!"
  * Dropping the old data does not affect the quality. 
    - "Something to consider: we can make the model updates more lightweight. For example, we 
      can exclude a bucket of older data every time we add a newer one."
  * Dropping the old data improves the quality! 
    - "These things happen. Our model forgets the outdated patterns and becomes more relevant. That 
      is a good thing to know: we operate in a fast-changing world, and the model should not be 
      too 'conservative'!"
  * Keep the data for less-represented populations and minor classes. 
    - "Dropping past data might disproportionately affect the performance of less popular classes. This 
      is an important thing to control for. You might decide to drop the old data selectively: remove what's 
      frequent, keep what's rare."
  * Assign higher weights to the newer data. 
    - "If the old data makes the model worse, you might decide to downgrade its importance but not exclude it 
      entirely. If you are feeling creative, you can do that at a different speed for different classes!"


### Part 2: Monitor the model performance in action

* Check #1. Monitor the performance changes
* Check #2. Monitor the shifts in data
* Part 3. Retraining vs Updates
