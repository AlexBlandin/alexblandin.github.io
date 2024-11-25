
# Index for [AISI Proposal](https://www.aisi.gov.uk/grants)

- [2024-10-17 Meeting (Alex and Francis)](./Quick%20meeting%20summary.html)
- [2024-10-21 Meeting (Andy, Alex, Francis)](./meeting-2024-10-21.html)
- [2024-10-22 A few possible names](./names.html)
- [2024-10-22 Tentative pricing for drafting a budget](./pricing.html)

<!--
we can use keras 3, jax, numba, mandela, and so on for the proposal, with sklearn random forests and such as a starting point (probably some Jax implementation available, or a keras one), keras 3 has bert pretrained and available, so that and such can be a starting point into a vector db / index (faiss/usearch), potentially one we can hook into postgres with the pgvector style extensions, as that way we can start to make structured queries and updates to the db
things like fnet and so on are better than transformers too (and fourier transform here is used to do the same kernel smoothing, I think), so if we want to go fast and use a big ensemble for the more interesting uses of 100+ models (not simply % agree, look at distribution, bimodal, outliers, etc)
right tools for the job, like cicero did, some stuff looks at entailment, some stuff is contrapositive diversity (range of models in ensemble, etc)
since the backend is effectively going to be a python / lisp / erlang / prolog repl loop, making queries and responding to inputs / changes (two tier? it'll do it's own loop in the background, you can do your repl stuff in the foreground, etc), the fact is we can hook in all sorts of stuff later if it makes sense as a tool to help solve the job, such as a recommender system to pull out sources or documents you may want to also see, etc, meanwhile that loop can extract a piece of text (maybe using bert or something to find where to snip/copy), parse it as a formula, run that / equate it, make use of it being a regular repl so exact results, visible so human intervention, traceable and explainable later, etc, and then insert it in our modify the text with the correct value (or provide alongside as a lens...), means we can extract and consider structured data and even graphs, etc, as well as dynamic or time series info, (Bayesian) probabilities, etc
to track a "concept" (any semantic marker, unit, etc) we could use kalman filtering (particle) to get a probabilistic path for the concept, to make sure we're on topic, or that certain concepts can be simultaneously tracked and expressed (with checks for if they don't show up where we'd like)
-->
