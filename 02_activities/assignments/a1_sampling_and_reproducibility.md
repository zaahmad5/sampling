# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times

# Author: Zaryab Ahmad

```
Question 1: 
Sampling is occurring in 4 parts of this model, as follows:

1. Sample of people who get infected - this uses simple random sampling, with each individual in the population having an equal probability of getting randmoly selected. 
- The Function used was: 
    infected_indices = np.random.choice(ppl.index, size=int(len(ppl) * ATTACK_RATE), replace=False)
    ppl.loc[infected_indices, 'infected'] = True
- The sample size was: attack_rate = 0.10 (i.e., 10% of the total poulation of 1000 individuals). 100 individuals were expected to be infected.
- The sampling frame: the full list of 1000 event attendees (200 at weddings, 800 at brunches)
- Underlying distribution: Binomial distribution, since each individual is independently infected with probability 0.10.
- How this relates to the procedure outlined in the blog post: the blog post assumes that infection happens randomly across events, meaning that all individuals at both types of events (weddings and brunches) have the same baseline risk  of infection. This step does not favour any event type at this stage.

2. Sample of infected people who get traced - this uses probability sampling, where each infected person has a 20% fixed probability of being traced.
- The Function used was: 
    ppl.loc[ppl['infected'], 'traced'] = np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS
- The sample size was: trace_success = 0.20 (i.e., 20% of infected individuals are expected to be traced). 20 individuals of the 100 were expected to be traced.
- The sampling frame: only the infected subset (100 out of 1000 attendees)
- Underlying distribution: Bernouli distribution, with each infected person individually having a 20% probability of being traced.
- How this relates to the procedure outlined in the blog post: In the blog post, contact tracing is imperfect, with only a fraction if infections successfully being traced. This indicates that most infections go untraced. Bias is introduced here because the untraced infections disappear from the dataset, leading to missing data in the final traced cases.

3. Sample of additional people traced due to event-based secondary tracing - this uses cluster sampling, where an entire event becomes filly traced if at least two infections have already been traced at that event.
- The Function used was: 
    event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()
    events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index
    ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True
- The sample size was: variable. As mentioned in the blog post, more weddings can be fully traced because they have larger guest lists and a higher chance of meeting the threshold.
- The sampling frame:  all infected individuals at events where at least 2 cases were already traced.
- Underlying distribution: conditional probability as it depends on the number of other traced cases at the same event.
- How this relates to the procedure outlined in the blog post: as mentioned above, the blog describes how larger, more structured events like weddings are easier to trace while smaller, more informal ones are harder to trace. This sampling step directly introduces and reinforces bias as it skews the percieved sources of infection.

4. Sampling from multiple simulation runs - this uses repeated random sampling to analyze how biased tracing affects the final perceived infection sources over many runs.
- The Function used was: 
    results = [simulate_event(m) for m in range(1000)]
- The sample size was: 1000 independent simulations using a new random sample each time
- The sampling frame: a fresh sample of 1000 individuals.
- Underlying distribution: with the large number of simulations involved in this step, the results stabilize due to the Law of Large Numbers.
- How this relates to the procedure outlined in the blog post: the blog post indicates that contact tracing data gives a biased picture of infection sources. This step quantifies this bias by showing that across 1000 runs, the true proportion of infections from weddings remains at 20% and that the proportion of traced cases attributed to weddings is much higher due to event-based secondary tracing bias. The graph generated by the final step demonstrates this systematic bias.

Question 2:
No, the python script doesn't fully reproduce the graphs used in the blog post. While the python script captures the general trend of weddings being overrepresented, it doesn't match the extreme bias shown in the blog post's original visualization.

Question 3:
I actually accidentally ran the updated script 3 times and saw overall the same pattern, but with a lot more randomness affecting the results. Each graph looked slightly different with where it both ranges were shown along the x-axis and the level of overlap with the two variables.

Question 4:
To increase reproducibility, we would need to increase the number of runs back to 1000 or higher. I chose to do 10,000 runs. 

To add to this, I also tried setting a random seed using np.random.seed(42) to ensure that the same sequence of random numbers was generated every time the script ran. However, this is probably not good for simulating real-world variability. 

```


## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `23:59 - 16/02/2025`
* The branch name for your repo should be: `assignment-1`
* What to submit for this assignment:
    * This markdown file (a1_sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-1`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via the help channel in Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
