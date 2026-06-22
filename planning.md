# Film Comparison 

**film comparison: reaction vs critique**


### Community: 
Chose to target the film industry because its distinction between reviews, questions, and hot_takes is very straightforward. This community is a good fit for a classification task because there is a difference between each reddit post that the user reads. 


### Labels: 

**1.** review: a personal review of a movie they have recently reviewed. 

**2.** hot_take: a unique opinion of parts of a movie or movie in general.

**3** questions: posing a question about aspects of a movie. 




### Hard edge cases: 

Hot_Takes and Reactions. 

### Data Colletcion Plan: 
Will collect samples off the reddit channel, TrueFilm. Will also supply 50 per label. 
If the label is underrpresented then I will find a new label. 

### Verification of Accuracy: 
If based on the labeling, I am happy with the results it outputs and agree with it, then it will be considered verified. 


## AI Tool Planning: 

**1.** Label Stress-Testing: 

reactions: a personal review of a movie they might've recently reviewed. 

hot_take: a unique opinion of parts of a movie or movie in general.

questions: posing a question about aspects of a movie. 

Generate 5-10 posts that sit at the boundary between hot takes and reactions. 

**2.** Annotation Assistance: 
Will use Groq to pre-label a batch of examples. 
