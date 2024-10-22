
# Bullet points from Thursday (2024-10-17) meeting between Alex and Francis

Just a few notes following ~1hr of discussion, apologies if I (Alex) forgot something. I've formatted as nested bullet-points simply as a function of expediency.

We did not systematically cover all the points in the [Examples of systemic AI safety projects](https://cdn.prod.website-files.com/663bd486c5e4c81588db7a1d/670d69c66574c9a112179373_Examples%20of%20systemic%20AI%20safety%20projects%20and%20focus%20areas.pdf) document, so more discussion to be had.

In particular, the 10 listed sector-specific interventions in that document seem like a good starting point. Of which, we mostly discussed points 4, 8, and 10 with some detail, and briefly covered 2 (early mention, simply not revisited after that), 5 (as "high-stakes" but not as high-stakes as healthcare), 7 (in connection to 8 via disaster response), and 9 (climate and sustainability direction would be nice, not sure where to start/intervene).

- Felt should be a strong focus / part
  - Privacy Preserving and Explainability
    - An underlying current, the "how"
    - Feeding into the "why" (as a "but safely/correctly/robustly")
    - Established (Federated Learning) but hasn't reached widespread adoption
      - Ensembles of independently federated models, so even if federated learning could learn data by extraction, data is present in only 1/N models, effectively providing a blurring/smoothing/censoring effect over individual models, with disparate models providing noise that covers in the other direction, with the overall ensemble providing effective aggregation of output (weighted summation, mixture-of-experts, learned spline/kernel combinations, hierarchical composition, etc)
  - Citizen Science
    - Federated Learning
    - Individualisation (in a privacy preserving way? based on the main AI and/or de novo? ensemble of both?)
- Discussed and potentially contributes to focuses
  - Neurosymbolic or more structured AI and/or ensembles
    - More AlphaGo/AlphaFold than GPT, even RETRO improves on GPT without major difference
      - Though RETRO and such "embedding" approaches to this suffer the continuous-training / stale-data problem (embedding of Wikipedia in-network to retrieve exact/relevant "facts")
        - Search engine / vector similarity, etc, indicates that embedding + retrieval, perhaps with AI/DL speed-ups/heuristics or learnt kernels, can be surprisingly effective
    - ProtoPNet as an example a computer vision model that explains by providing examples where "this looks like that" from the training data (bounded salient regions)
  - Adversarial attacks and data extraction
    - Potentially even on federated learning and other privacy preserving approaches
      - (Note to self: Must ask Xianghua Xie, presented a "leak" theorem(?) in a talk earlier this year based on observation by Alan Dix — from what I recall, learned models encode data in a way that can be extracted at a rate proportional to the accuracy)
      - Potentially undermines concepts such as holomorphic encryption, etc, as the learned model may still be extracted from even if trained in a "perfectly" privacy-preserving way
- Areas that we may want to focus on (or mentioned with interest)
  - Emerging domains / applications of AI
  - (Climate) Disaster Response
  - Surveillance for emergency location/presence detection and/or crowd safety
    - Where are people in case of fires, where safe, where to direct to, etc
    - Issue with crowd safety is clear connection to non-privacy preserving uses, so the AI is privacy-preserving, but the system after that is not
- Areas that don't seem to clearly benefit from (proper) AI
  - Autonomous threat detection and response
    - State-of-the-art seems sufficiently capable, statistical learning for outlier detection, etc
      - Potentially there are "proactive" preventative measures, code scanning, etc, but unclear how we could use AI on this appropriately at current (LLMs Considered Harmful)
- Areas that may be appropriate if done in the right way, but that may be challenging to do so
  - Autonomous drones/swarms in disaster responses
    - Downside is, well, autonomous drones and certain belligerent nation states
    - A goal for situations like this is harm reduction and risk mitigation, so where we must alleviate and avoid aggravating, so some interventions should be avoided for opening one beneficial door here that simultaneously opens a dangerous one elsewhere, naturally there's a rounding-off point, however many alternative solutions will be viable which do not have the same properties, so proper systemic responses should be motivated to explore those first
  - Direct healthcare areas (clinician tooling, patient outcomes, simulation/modelling, etc)
    - Difficult to get data for (promptly)
      - COVID data, while accessible, is thoroughly researched, so fewer emerging ideas to levy
    - High-stakes!
      - Delays in ethics approval, etc, may lead to downsizing
      - Domain experts likely required for access to data/facilities
      - Domain experts, or access to them, likely required for successful bids
- Areas to avoid
  - Autonomous cars
    - Simply not viable at present, and they are a transitionary technology anyway
      - Collaborative and railway style scheduling and linking are inevitable, since needs of efficiency and practicality will win out should they become the predominant mode of automobile
      - General purpose cognition, recognition, judgement, and reaction, potentially at human-levels, may be necessary to deal with unstructured real-world scenarios bound only by the laws of physics, not probability
  - Pure LLM, "LLM + Fact Checking", etc
    - Somehow even LLM + Fact Checking is unbelievably nascent, and often uses an LLM to do so, completely defeating the point of it (though RETRO indicates some architectural potential)
      - discussion with some presenting LLM-based work at CHI 2023 ended up taking a turn as they admitted to not considering using an external fact-checking system to ensure consistent/correct statements are made, little has been readily apparent in development/adoption since
- Mentioned Papers / Research
  - Cynthia Rudin's work on explainability
    - Chen, Chaofan, et al. "This Looks Like That: Deep Learning for Interpretable Image Recognition." Advances in Neural Information Processing Systems 32 (2019).
      - ProtoPNet introduced here
    - Rudin, Cynthia. "Stop explaining black box machine learning models for high stakes decisions and use interpretable models instead." Nature Machine Intelligence 1.5 (2019).
      - Exemplar case of Rudin arguing that understanding of the problem is essential, our discussion followed on how we use AI when we do not understand the problem well enough to provide a (close-form) solution we can write in normal code, or when we recognise that some portion of the problem (variable tuning) well-enough to see that ML training on data is effective (but not so well that we don't need said approach), and that by focusing on the problem rather than the solution we can invoke and use more appropriate techniques from the broader range of statistical learning, AI, and programming in general, generally through a better understanding of the problem (which may be improved via ML, such as distillation)
  - Xianghua Xie's work on federated learning and computer vision
    - [His group's publications](http://csvision.swan.ac.uk/index.php?n=Site.Publication)
  - Work on Adversarial Attacks, including defensive uses of them
    - Shan, Shawn, et al. "Glaze: Protecting Artists from Style Mimicry by Text-to-Image Models." 32nd USENIX Security Symposium (USENIX Security 23). 2023.
      - Indicative of how adversarial attacks can be used defensively
    - Fig 1A from [this article](https://spectra.mathpix.com/article/2021.09.00001/adversarial-attacks)
      - Indicative of adversarial attacks being a "feature not a bug" with current numerical approach, particularly with "invisible" changes or with non-human failure modes (esp. on "frontier/foundation" models aka computer vision models with LLM "reading" to classify)
  - Neurosymbolic/hybrid AI, or at least AI with more structured inductive biases that may have "side-channels" (e.g. ProtoPNet or RETRO)
    - Borgeaud, Sebastian, et al. "Improving language models by retrieving from trillions of tokens." International conference on machine learning. PMLR, 2022.
      - RETRO introduced here
