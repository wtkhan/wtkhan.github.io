---
layout: post
title: "Data analyses as pipelines"
summary: "Wrapping code in a pipeline brings reusability, consistency, and clarity to ad-hoc analyses. These qualities are useful for sharing best practices across projects and teams"
description: "Wrapping code in a pipeline brings reusability, consistency, and clarity to ad-hoc analyses. These qualities are useful for sharing best practices across projects and teams"
---

There are two types of analyses in product data science. One is where you do a series of ad-hoc queries to the company's data warehouse (and dimensional models, etc.), external data sources, then visualize the results to make a series of recommendations. Typical examples of these analyses are investigating root causes (why is CTR down this week?), evaluating new ML feature candidates and developing logic for marketing campaigns. The notebook, where this is almost always done, is shared with non-technical stakeholders who decide how to best proceed with the recommendations and offer their own takes on the visualizations. This is important, impactful work that cannot be routinized. A data colleague of the team will review the logic and validate the methodologies and findings. Each analysis is as different as the question posed. If the same question is to be posed again, simply rerun the notebook with newer data.

The second type of analysis is "ML insights". Here the analysis requires building the best model to either understand predictive relationships or validate building a new model for production purposes. Examples:
- Given some data, what features are most predictive of user conversions?
- Given some data, develop a model to predict new conversions
- How would adding evaluation metric X change quality of predictions? Would a Y-optimized model beat it?

These analyses will assume that we have a good understanding of the data that will be used to train the model. If that isn't the case, absolutely do that exploration work first before jumping into modeling. I think we all agree in principle that's the right order of operations, but I will confess to having underestimated time for either exploration or model development and tried to jump into doing both things in one ticket.

I believe that these analyses should be done rigorously in the code development sense. I have worked on these questions in notebooks and didn't like the developer experience. If we can keep

- Data exploration: It takes a good bit of time to just marinate in the data. If this was a work ticket, I'd at least take a workday to really understand the data, including answering some really basic questions about it before I can advise a stakeholder on how we could use that data. I think the business impact of any data wrangler's work is to handle each messy dataset/database as its own special problem and get the best out of it. Most takehomes are not designed to do this. In fact, poorly designed ones invite suspicion that companies just want you to work for them for free.
- Data modeling: I fight every temptation to jump into modeling immediately after receiving the assignment and having a baseline to iterate on. After all, this is the concrete deliverable on which I will be judged. Spaghetti queries are of no use to the company's graders, so having some ML work ready is paramount.
- Communication: I spend a lot of time at work figuring out what the clear actions are

So what can I do? I want to automate/standardize as much of the analysis so I can focus on where I think any data professional can deliver value: actions informed by fluency with the data. this is why I view takehomes as ETL tasks in a sense. I want to focus on extracting and tidying the data, and let the loading components of the pipeline build a minimum viable model out of it. and within the extraction and tidying components, I want to further abstract away the routine checks for each input schema so I can focus on the abstract business question. in an idealized setting, I would do the following steps very quickly, within the first 2 hours of starting on the assignment:

- write an initial series of ad hoc queries for a basic understanding of the data. store the ones I want to refer back to in a dictionary. in these queries, I'd have at least defined the key analysis outcome/metric, business segments/facets and split the dataset to avoid information leakage. these would form the basis of the actions I can focus on and validate in my analysis. for example: outcome y is lagging with segment x1, so we need to dig deeper into that. y is great with segment x2, so we need to do more of whatever we're doing there
- profile the raw training data for a report card of all its issues
- take action on those issues-- delete rows, drop columns, transform existing ones, etc.-- by refining my queries. document those actions
- do steps 1 and 2 again
- draw some initial graphs to display the distribution of the outcome, faceted by some key segments
- drill down on patterns and anomalies in the graphs, again by refining queries and segments
- at the end of the initial investigation, I should have quality numerical features and a set of impactful graphs. if I have done this well, I should go into modeling with good intuition about the most salient features. and the modeling should tell me if the patterns I see in the graphs are up to snuff
- non-goal: have the most automated data solution possible. I cannot build against every possible data problem without being in the loop

Here, the modeling aspect is routinized because I invested time up front understanding the data and its nuances. I will undoubtedly miss some stuff because this whole assignment is my first pass or I have limited knowledge of the domain. but those are things that can be addressed. also, having some of the modeling work as routine helps me bring value where I can bring