Back to [All Sims](https://github.com/CompCogNeuro/sims) (also for general info and executable downloads)

# Introduction

This network learns to encode both syntax and semantics of sentences in an integrated "gestalt" hidden layer. The sentences have simple agent-verb-patient structure with optional prepositional or adverb modifier phrase at the end, and can be either in the active or passive form (80% active, 20% passive). There are ambiguous terms that need to be resolved via context, showing a key interaction between syntax and semantics.

This version of the model uses the more recent *DeepLeabra* framework [(O'Reilly et al, 2017)](#references) that incorporates the functions of the deep layers of the neocortex and its interconnectivity with the thalamus.  We hypothesize that these circuits support a powerful form of predictive error-driven learning, where the top-down cortical projections drive a prediction on the pulvinar nucleus of the thalamus, which serves as the minus phase, and then phasic bursting inputs drive a periodic plus phase representing what actually happened.  This bursting happens at the 10 Hz *alpha* frequency (every 100 msec), and forms the basis for the use of this time scale as the primary organizing principle in our models.

The current model uses these circuits to learn by attempting to predict which word will appear next in the sentences -- this prediction is represented in the `InputP` pulvinar layer.  To support predictive learning, the deep layers also have the ability to maintain "context" information from the previous time step, and this is also critical for enabling the model to remember information from prior words in the sentence.  This contextual information is similar to the simple recurrent network (SRN) model [Elman, 1990](#references), which was used in the prior version of this simulation.

# Training

In addition to the predictive learning, the network is trained by asking questions about current and previous information presented to the network.  During the minus phase of every trial, the current word is on the `Input`, and a question is posed in the `Role` input layer about what a particular semantic role is for the current sentence. For example, for the first word in an active sentence, which is always the "agent" of the sentence, the `Role` input will be `Agent`, and the network just needs to produce on the `Filler` output layer the same thing that is present in the `Input`. However, on the next trial, a new word (the verb or `Action`) is presented, but the network must still remember the `Agent` from the prior trial. This is what the `GestaltD` Deep context layer provides. You can see all of this in the testing we'll perform next. If you want to see the training process in more detail, you can click on `Init` and `Step Trial` through a few trials.

* Next, press `Open Weights` in the toolbar to load pre-trained weights (the network takes many minutes to hours to train). Then, you can start by poking around the network and exploring the connectivity using the `r.Wt` view, and then return to viewing `Act`.

# Testing

Now, let's evaluate the trained network's performance, by exploring its performance on a set of specially selected test sentences listed here:

* Role assignment:
    + Active semantic: The schoolgirl stirred the Kool-Aid with a spoon.
    + Active syntactic: The busdriver gave the rose to the teacher.
    + Passive semantic: The jelly was spread by the busdriver with the knife.
    + Passive syntactic: The teacher was kissed by the busdriver.
    + Active control: The busdriver kissed the teacher.
*  Word ambiguity:
    + The busdriver threw the ball in the park.
    + The teacher threw the ball in the living room.
* Concept instantiation:
    + The teacher kissed someone (male).
* Role elaboration
    + The schoolgirl ate crackers (with finger).
    + The schoolgirl ate (soup).
* Online update
    + The child ate soup with daintiness.
    + control: The pitcher ate soup with daintiness.
* Conflict
    + The adult drank iced tea in the kitchen (living room).

The test sentences are designed to illustrate different aspects of the sentence comprehension task, as noted in the table. First, a set of role assignment tasks provide either semantic or purely syntactic cues to assign the main roles in the sentence. The semantic cues depend on the fact that only animate nouns can be agents, whereas inanimate nouns can only be patients. Although animacy is not explicitly provided in the input, the training environment enforces this constraint. Thus, when a sentence starts off with "The jelly...," we can tell that because jelly is inanimate, it must be the patient of a passive sentence, and not the agent of an active one. However, if the sentence begins with "The busdriver...," we do not know if the busdriver is an agent or a patient, and we thus have to wait to see if the syntactic cue of the word *was* appears next.

* Do `Test trial` to see the first word in the first test sentence.

The first word of the active semantic role assignment sentence (schoolgirl) is presented, and the network correctly answers that schoolgirl is the agent of the sentence. Note that there is no plus-phase and no training during this testing, so everything depends on the integration of the input words.

* Continue to do `Test Trial` through to the final word in this Active semantic sentence (spoon). You can click on the `TstTrlPlot` tab to see a bar-plot of each trial as it goes through, with the network's `Output` response. You should observe that the network is able to identify correctly the roles of all of the words presented. Because in this sentence the roles of the words are constrained by their semantics, this success demonstrates that the network is sensitive to these semantic constraints and can use them in parsing.

* Now Step through the next sentence (Active syntactic) -- you can do `Test Seq` to zip through the entire sequence of one sentence and just look at the bar plot.

This sentence has two animate nouns (busdriver and teacher), so the network must use the syntactic word order cues to infer that the busdriver is the agent, while using the "gave to" syntactic construction to recognize that the teacher is the recipient. Observe that at the final word in the sentence, the network has correctly identified all the words.

In the next sentence, the passive construction is used, but this should be obvious from the semantic cue that jelly cannot be an agent.

* Step through Passive semantic and observe that the network correctly parses this sentence.

In the final role assignment case, the sentence is passive and there are only syntactic constraints available to identify whether the teacher is the agent or the patient. This is the most difficult construction that the network faces, and it does not appear to get it right -- it gets confused about the busdriver toward the end of the sentence, replacing him with pitcherpers and schooolgirl. 

* Step through this Passive syntactic sentence.

Further testing has shown that the network sometimes gets this sentence right, but often makes errors. This can apparently be attributed to the lower frequency of passive sentences, as you can see from the next sentence, which is a "control condition" of the higher frequency active form of the previous sentence, with which the network has no difficulties.

* Step through this Active control sentence.

The next two sentences test the network's ability to resolve ambiguous words, in this case *throw* and *balll* based on the surrounding semantic context. During training, the network learns that busdrivers throw baseballs, whereas teachers throw parties. Thus, the network should produce the appropriate interpretation of these ambiguous sentences.

* Step through these next two sentences (Ambiguity1 and 2) to verify that this is the case.

Note that the network makes a mistake here by replacing *teacher* with the other agent that also throws parties, the *schoolgirl*. Thus, the network's context memory is not perfect, but it tends to make semantically appropriate errors, just as people do. 

The next test sentence probes the ability of the network to instantiate an ambiguous term (e.g., *someone*) with a more concrete concept. Because the teacher only kisses males (the pitcher or the busdriver), the network should be able to instantiate the ambiguous *someone* with either of these two males.

* As you Step through this sentence, observe that `someone` is instantiated with `pitcherpers`. 

A similar phenomenon can be found in the role elaboration test questions. Here, the network is able to answer questions about aspects of an event that were not actually stated in the input. For example, the network can infer that the schoolgirl would eat crackers with her fingers.

* Step through the next sentence (Role elaboration1).

You should see that the very last question regarding the instrument role is answered correctly with fingers, even though fingers was never presented in the input. The next sentence takes this one step further and has the network infer what the schoolgirl tends to eat (crackers). 

* Go ahead and Step through this one (Role elaboration2).

The next test sentence is intended to evaluate the online updating of information in a case where subsequent information further constrains an initially vague word. In this case, the sentence starts with the word *child*, and the original weights for and older version of the network vacillated back and forth about which child it answered the agent question with. When the network received the adverb *daintiness*, this uniquely identified the schoolgirl, which it then reported as the agent of the sentence (even though it did not appear to fully encode the daintiness input, producing *pleasure* instead). In the trained weights for this model, the network strongly encoded pitcher as the child, and even decided that he should be eating steak (perhaps as a carry-over from the fact that the other male, the busdriver, has a strong preference from steak). This reinforcement of the male agent representation made the model impervious to the daintiness input.

* Step through the Online Update sentence.

To verify that *daintiness* is having an effect on this result, we can run the next control condition where the pitcher is specified as the agent of the sentence -- the older version clearly switched from saying *pitcherpers* to saying *schoolgirl* after receiving the *daintiness* input. Again, this model reinforces the male representation and outputs steak and is unaffected by the daintiness input.

* Step through the Online control sentence.

The final test sentence illustrates how the network deals with conflicting information. In this case, the training environment always specifies that iced tea is drunk in the living room, but the input sentence says it was drunk in the kitchen. 

* Step through this Conflict sentence.

Notice that the network responds with pitcher (container) for the location, despite getting kitchen in the input. This also causes it to think that the agent is the teacher for some reason.. The main point is that when *kitchen* is input, the network responds with something more consistent with its prior knowledge (Iced-tea is stirred in the container). This may provide a useful demonstration of how prior knowledge biases sentence comprehension, as has been shown in the classic "war of the ghosts" experiment (Bartlett, 1932) and many others.

# Nature of Representations

Having seen that the network behaves reasonably (if not perfectly), we can explore the nature of its internal representations to get a sense of how it works.

* Press ProbeClust on the overall control panel.

After a short delay, the cluster plot for the unambiguous nouns will show up, followed by the cluster plot for a set of probe sentences (see data/InputData/ProbeSentences data table for these). These sentences systematically vary the agents, patients, and verbs to reveal how these are represented.

* Click the NounEncodeCluster first.

This cluster plot shows very sensible similarity relationships among the encoding representation of the inputs.

* Click the ProbeSentCluster tab to see the sentence probe cluster plot.

This cluster plot shows that the sentences are first clustered together according to verb (not perfectly, but most of the ate (at) cases are in one big cluster, and drink (dr) and stir (st) are in separate clusters, with just a few "outlier" cases at the top), and then by patient, and then by agent within that. Furthermore, across the different patients, there appears to be the same similarity structure for the agents. Thus, we can see that the gestalt representation encodes information in a systematic fashion, as we would expect from the network's behavior. 

> **Question 9.12:** Does this cluster structure reflect purely syntactic information, purely semantic information, or a combination of both types of information? Try to articulate in your own words why this kind of representation would be useful for processing language.

# References

* Elman, J. L. (1990). Finding Structure In Time. Cognitive Science, 14(2), 179–211.

* O’Reilly, R. C., Wyatte, D. R., & Rohrlich, J. (2017). Deep predictive learning: A comprehensive model of three visual streams. ArXiv:1709.04654 [q-Bio]. Retrieved from http://arxiv.org/abs/1709.04654