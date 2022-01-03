

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



------------------

Start: noon
End: 1240

# Reading Materials

## Machine Learning in Production

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
* Data Drift
  - aka: feature drift, population shift, covariate shift
  - primary gist: the input data has changed (resulting in a meaningfully different distribution of variable)
  - primary problem: the model is not trained for this new situation!
    * still works on the old training, val, and test sets (obviously)
    * possibly even worked for a while in production
    * however, no longer working well in production
  - Example (they use advertising, but I'll use human activity recognition (HAR))
    * HAR model is trained on healthy males aged 15-35
    * HAR model works well early when company's demographic primarily young adult males
    * company generalizes product and extends offers to older males 
    * HAR model's performance seen to drop since the older males move and behave in meaningfully
      different ways
    * HAR model is retrained and found to work well again on company's user base
    * company wants a successful Q3, so begins targeting sales at their untapped demographic - females
    * HAR model's performance seen to drop again
    * etc!
  - Solutions
    * Retrain the model using a deployment-faithful population
    * Rebuild the model for the new population segments
* Training-Serving Skew
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
* Concept Drift      
    






